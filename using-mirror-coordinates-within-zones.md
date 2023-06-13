# Using mirror refs within the points section

If you are making a monoblock, then `mirror` is the way to make the two halves identical. Let's make a little four-key keyboard as an example.

```yaml
meta:
  engine: "4.0.0"

points:
  mirror:
    ref: matrix_inner_home
    distance: 3U

  zones.matrix:
    columns:
      outer:
      inner:
    rows.home:

outlines:
  example:
    - what: rectangle
      where: true
      size: [$default_width, $default_height]
```

Now what if we wanted to add a circle exactly in the middle? As long as we are in the `outlines` or `pcbs` section, then the handy `ref.aggregate.parts` is perfect for this when used in conjunction with a `mirror_*` reference:

```yaml
meta:
  # ...
points:
  # ...
outlines:
  example:
    - what: rectangle
      where: true
      size: [$default_width, $default_height]

    # Add a circle in the middle
    - what: circle
      where:
        # Averaging matrix_inner_home and mirror_matrix_inner_home
        # gives us a point exactly between them
        ref.aggregate.parts: [matrix_inner_home, mirror_matrix_inner_home]
      radius: .25U
```

But what if you need to center something while you're within the `points` section? For example, what happens if you try this?

```yaml
meta:
  engine: "4.0.0"

points:
  mirror:
    ref: matrix_inner_home
    distance: 3U

  zones.matrix:
    columns:
      outer:
      inner:
    rows.home:

  # This will cause an error
  # Error: Unknown point reference "mirror_matrix_inner_home" in anchor
  # "points.zones.center_button.anchor.ref.aggregate.parts[2]"!
  zones.center_button:
    anchor:
      ref.aggregate.parts: [matrix_inner_home, mirror_matrix_inner_home]

outlines:
  example:
    - what: rectangle
      where: true
      size: [$default_width, $default_height]

```

As noted in the comment, this approach will not work. The culprit is that the `mirror` directive has been set globally on the entire `points` section, and therefore the `mirror_matrix_inner_home`  reference has not been computed yet. 

There is a solution though! And that is to put the mirror directive into the matrix zone itself. Then the mirror point will be computed by the time we reach the center_button zone.

```yaml
meta:
  engine: "4.0.0"

points:
  zones.matrix:
    # Move `mirror` into this zone
    mirror:
      ref: matrix_inner_home
      distance: 3U
    columns:
      outer:
      inner:
    rows.home:

  zones.center_button:
    anchor:
      # This works now because the mirror point is computed as soon as the the
      # matrix zone has been fully declared
      ref.aggregate.parts: [matrix_inner_home, mirror_matrix_inner_home]

outlines:
  example:
    - what: rectangle
      where: true
      size: [$default_width, $default_height]

```

There is a caveat to using this trick, which is if multiple zones need to be mirrored, you must mirror each one individually. You can use a YAML anchor and alias to keep that as DRY as possible. Let's add a thumb cluster to show how this would work.

```yaml
meta:
  engine: "4.0.0"

points:
  zones.matrix:
    # Add a yaml anchor so we can point to this mirror directive from other zones
    mirror: &mirror
      ref: matrix_inner_home
      distance: 3U
    columns:
      outer:
      inner:
    rows.home:

  # Add a thumb cluster
  zones.thumbs:
    # Grab the mirror directive from the matrix zone so we can mirror this zone identically
    mirror: *mirror
    anchor:
      ref: matrix_inner_home
      shift: [0, -1.25U]

  zones.center_button:
    anchor:
      ref.aggregate.parts: [matrix_inner_home, mirror_matrix_inner_home]

outlines:
  example:
    - what: rectangle
      where: true
      size: [$default_width, $default_height]
```

