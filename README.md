**Status:** Archive (code is provided as-is, no updates expected)

# Worldgen: Randomized MuJoCo environments

Worldgen allows to generate complex, heavily randomized environments
with goals. Examples of such environments can be found in the `examples` folder.

Actions in `action_space` are all actuators of objects added during world building. Not all objects will have actuators, but some do (e.g. `ObjFromXML('particle')` and `ObjFromXML('particle_hinge')`). You can examine the meaning of a given action by looking at the xml file of the object in `assets/xmls/.../main.xml`

## Installation

This repository requires the MuJoCo physics engine (the repository has been tested with MuJoCo 1.50). To install MuJoCo, follow the instructions in the [mujoco-py](https://github.com/openai/mujoco-py/tree/1.50.1.0) repository.

```
    pip install -r requirements.txt
    pip install -e .
```

This repository has been used on Mac OS X and Ubuntu 16.04 with Python 3.6

## Walkthrough

### Initial steps

Let’s analyze an example of one such generation.  First we create the `WorldParams`:

```
    world_params = WorldParams(size=(5, 5, 3.5))
```

This defines the global properties of our generated world.
Metric units used in worldgen are meters, kilograms, and angles are given in radians: `[-pi, pi]`. 
The following are all optional paramaters with defaults:

-  `size`: The size of the space available for placing objects. (Default: `(10., 10., 2.5)`)
-  `num_substeps`: The number of physics substeps to perform within every call to `step()`. (Default: `1`)
-  `randomize_light`: Whether to randomize the lighting conditions. (Default: `False`)
-  `randomize_material`: Whether to randomize the materials that are applied to objects. (Default: `False`)
-  `placement_margin`: The minimum distance between placed objects. (Default: `0.0`)
-  `show_outer_bounds`: Whether to visualize the outer bounds. (Default: `False`)

Then, we create a builder. The `WorldBuilder` constructor takes a `WorldParams` object and `seed`, which is used for randomization:

```
	builder = WorldBuilder(world_params, seed)
```

### Placing objects

Here's an example of adding geometries to our world:

```
    # Create a floor
    floor = Floor()

    # Load geometries from XML, and add to floor
    robot = ObjFromXML("particle")
    floor.append(robot)
    sphere = ObjFromXML("sphere")
    floor.append(sphere)

    # Create a primitive geometry, and add to floor
    box = Geom('box')
    floor.append(box)

    # Add the root floor to the builder
    builder.append(floor)
```

The `append()` function allows to specify a placement indicator (Usually “top” or “inside”).
Placements are spaces we’re able to place objects. 
Placements world-aligned rectangular prisms, which objects can be oriented within.  All objects within a placement align along the bottom (and are positioned with X,Y coordinates).
If there are multiple placements for a given name (e.g. “inside\_0”, “inside\_1”, …) then we choose one at random.
The default placement name is “top”. 

To customize the placement position:

```
    obj.append(child_obj, placement_name="top", placement_xy=None)
```

`placement_xy` is None if you want the world generation algorithm to randomly place it.
Otherwise it is an X, Y pair where both are within `[0.0, 1.0]`.
The object will be placed within the bounds, scaled by its size.
*Note:* this is because the size of the placement (e.g. table) might not
be known until generation time. To add some more objects:

```
    obj.append(child_obj, 'top', (0.5, 1))  # placed in the center of the back
    obj.append(child_obj, 'top', (0.5, 0.5))  # placed in the center
    obj.append(child_obj, 'top', (0, 0))  # placed in the lower left corner
```

### Geoms

There are several kinds of primitive geoms: “box”, “sphere”, “cylinder”.
We can specify the size of a geom by providing a second parameter: `Geom("box", (0.1, 0.2, 0.3))` which would result in box of size 10cm x 20cm x 30cm.
For a cube: `Geom("box", 0.25)` results in “box” of size 25cm x 25cm x 25cm.
Moreover, we can provide a range to sample a random size box: `Geom("box", (0.1, 0.2, 0.3), (1.1, 1.2, 1.3))`.
Size of this box would be random between 10cm x 20cm x 30cm and 1.1m x 1.2m x 1.3m.

### Environments

You can create new environments with Worldgen by subclassing the `Env` class and defining the following methods:

- `_get_sim`
- `_get_obs`
- `_get_reward` : You can also use a reward wrapper instead of defining this.

You can see two examples in the `examples` folder - the `simple_particle` environment and the `particle_gather` environment.
You can test out environments using the `/bin/examine` script, providing either a python script defining the `make_env` or a jsonnet file.

```
    ./bin/examine.py examples/simple_particle.py
```

```
    ./bin/examine.py examples/example_env_examine.jsonnet
```
