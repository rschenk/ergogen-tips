# Prefer `$extends` to YAML anchors

You may find yourself needing to duplicate some of your config to share attributes between multiple sections (see [Mirror refs within the `points` section](using-mirror-coordinates-within-zones.md) for an example). Since our config files are in YAML, we could use YAML's anchor/alias feature, or we could use Ergogen's built-in `$extends` syntax. They both will accomplish the same thing, but latter `$extends` is preferred because you can also write Ergogen files in JSON. YAML anchors will only work in YAML files, whereas `$extends` will work in both file formats.

```yaml
# Anchors work but are NOT preferred
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
# Extends is preferred
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
