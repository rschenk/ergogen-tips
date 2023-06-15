# Cautiously use  YAML anchors and `$extends` ~~Prefer `$extends` to YAML anchors~~

You may find yourself needing to duplicate some of your config to share attributes between multiple sections (see [Mirror refs within the `points` section](using-mirror-coordinates-within-zones.md) for an example). Since our config files are in YAML, we could use YAML's anchor/alias feature, or we could use Ergogen's built-in `$extends` syntax. They both will accomplish the same thing.

Ideally we would always prefer `$extends` to anchors because you can also write Ergogen files in JSON and of course YAML anchors will only work in YAML files, whereas `$extends` will work in both file formats. 

Unfortunately at the current time you cannot nest calls to `$extends`, which is an untintentional oversight in ergogen. My recommendation is to check the status of this [issue](https://github.com/ergogen/ergogen/issues/97) and once it's closed, always prefer `$extend` to yaml anchors... but until that day, you'll want to use anchors too.

```yaml
# Anchors work, but only in YAML files
points:
  zones.matrix:
    mirror: &mirror
      ref: some_key
      distance: 4U
    key.name: some_key
  zones.thumb:
    mirror: *mirror
    anchor.shift: [0, 1.25U]
```

```yaml
# Extends do the same thing
# and will be preferred once they can be nested
points:
  zones.matrix:
    mirror:
      ref: some_key
      distance: 4U
    key.name: some_key
  zones.thumb:
    mirror:
      $extends: points.zones.matrix.mirror
    anchor.shift: [0, 1.25U]

# You could also one-liner it
  zones.another_thumb:
    mirror.$extends: points.zones.matrix.mirror
    anchor.shift: [0, 2.5U]
```
