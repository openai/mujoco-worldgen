**Status:** Archive (code is provided as-is, no updates expected)

# Worldgen: Randomized MuJoCo environments

Worldgen allows to generate complex, heavily randomized environments environments
with goals. Examples of such environments can be found in the `examples` folder.

Actions in action_space are all actuators of objects added during world building. Not all objects will have actuators, but some do (e.g. `ObjFromXML('particle')` and `ObjFromXML('particle_hinge')`). You can examine the meaning of a given action by looking at the xml file of the object in `assets/xmls/.../main.xml`

## Installation

This repository requires the MuJoCo physics engine. To install MuJoCo, follow the instructions in the [mujoco-py](https://github.com/openai/mujoco-py) repository.

```
pip install -r requirements.txt
pip install -e .
```

This repository has been used on Mac OS X and Ubuntu 16.04 with Python 3.6

## Walkthrough

### Initial steps

Let’s analyze an example of one such generation:

`world_params = WorldParams(size=(5, 5, 3.5))`

First, we define the global properties of our generated world.
Metric units used in worldgen are meters, kilograms, and angles are given in radians: `[-pi, pi]`. 
None of these values has to be specified as there are default settings.
There are the following paramaters:

-  `randomize_light`: randomize lighting conditions
-  `randomize_material`: randomize materials that are used to cover
   objects
-  `num_substeps`: number of steps performed in every function call to
   step().
-  `size`: size of space available for placing objects.

Then, we create a builder. Builder takes seed which determines
randomness, and WorldParams object:

	builder = WorldBuilder(world_params, seed)

### Placing objects
```
    floor = Floor()
    robot = ObjFromXML("particle")
    floor.append(robot)
    box = Geom('box')
    floor.append(box)
    sphere = ObjFromXML("sphere")
    floor.append(sphere)
    builder.append(floor)
```
`append()` allows to specify location in a given placement.
Placements are spaces we’re able to place objects. Usually “top” or “inside”.
Placements are always world-aligned rectangular prisms, and all objects placed within them are sitting on the bottom (so specified with X,Y coordinates).
If there are multiple placements for a given name (e.g. “inside\_0”, “inside\_1”, …) then we choose one at random.
The default placement name is “top”. 

To customize the placement position:

```obj.append(child_obj, placement_name="top", placement_xy=None)```

`placement_xy` is None if you want the world generation algorithm to randomly place it.
Otherwise it is an X, Y pair where both are in [0.0, 1.0].
The object will be placed within the bound scaled by its size.
*Note:* this is because the size of the placement (e.g. table) might not
be known until generation time. To add some more objects:
```
    obj.append(child_obj, 'top', (0.5, 1))  # placed in the center of the back
    obj.append(child_obj, 'top', (0.5, 0.5))  # placed in the center
    obj.append(child_obj, 'top', (0, 0))  # placed in the lower left corner
```

### Geoms
There are several kinds of geoms: “box”, “sphere”, “cylinder”.
We can specify size of a geom by providing second parameter: `Geom("box", (0.1, 0.2, 0.3))` which would result in box of size 10cm x 20cm x 30cm.
For a cube: `Geom("box", 0.25)` results in “box” of size 25cm x 25cm x 25cm.
Moreover, we can provide a range to sample a random size box: `Geom("box", (0.1, 0.2, 0.3), (1.1, 1.2, 1.3))`.
Size of this box would be random between 10cm x 20cm x 30cm and 1.1m x 1.2m x 1.3m.

### Environments

You can create new environments with Worldgen by subclassing the `Env` class and defining the `_get_sim`, `_get_obs` and `_get_reward` methods (or using a reward wrapper instead of defining `_get_reward`.). You can see two examples in the `examples` folder - the `simple_particle` environment and the `particle_gather` environment.

You can test out environments by using the `/bin/examine` script
