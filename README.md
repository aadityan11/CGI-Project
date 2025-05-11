# CGI-Project
Final Code/Explanation Submission for my CGI project

ADAPTIVE PROCEDURAL TERRAIN GENERATION.

Geometry Node Setup

Terrain
Starting with a square Plane (4 vertices). I first scale and subdivide the mesh and set a material to it. The starting mesh does not really matter, because I am anyway deleting it and replacing it with a plane. I could have even used a sphere or cube for the terrain.

Water
I've taken a NURBS Path, and wrapped an array of planes to it, such that the planes follow the curve and set a material to it. To interact with the terrain, I have converted the curve to mesh. 


Displacement Generation

This can be done using any noise generation node/function. (Perlin/Voronoi etc.). We start the displacement by choosing a noise pattern. This is a 2D noise pattern that I have used to generate 3D height maps using the Combine XYZ node, which allows us to separate the 3 axes and choose which one we want to affect. To generate displacement, we need to use the Z value of the plane. We control the displacement using a curve node. This is a nonlinear adjustment to the height values from the noise texture. The noise is a gradient of black and white. I've used it such that the white regions mean elevation and black means depressions - but this can be inverted at any time by multiplying by -1.
One the curve, 
Lower distances are lifted slightly.
Midrange is more steeply boosted.
Higher values are closer to linear.
What I've done next is to take the original position of that point and then vector add it to the new value from the curve. This pushes point on the terrain up and down and generates displacement.


Land and River Intersection - Geometry Proximity.

Geometry Proximity and distance node calculate points on the terrain from the river (NURBS Path). This is connected to a XYZ node, which allows us to affect each of the axes separately. For this, we want the Z value to be affected, So that points on terrain that are close to the river, their Z values (displacement values) are lowered, below river level. 

How much to lower the Z value is done by the multiply add node - which decides from where to start lowering the Z values. This basically controls how wide the river should be and the falloff of the land.
If Distance = 0.0  → Output = 0.04 (closer to river edge - these points will be have the lowest Z value)
If Distance = 1.0  → Output = 0.77 (0.73 * 1 + 0.04) (farther away from river - no change in Z value.
But Clamping is on, so any values outside the 0–1 range will be clipped.
This gives a falloff from the river outward, between 0.04 and 0.77 ( for this setting of the Multiply node.)


Scattering Systems

I have used 2 scattering systems. 
One is a global scattering, that scatters the chosen model over the entire landscape - but is gradually removed closer to the river.
The points(grass models) that are scattered are controlled by the geometry proximity setup above. 
Scattering using geo nodes is done by marking points all over the geometry - this is what the density value of the node does(how many points do you want over that geometry). And each point is instanced to a model/object or a collection of objects that we can choose. So now I am rending the points as plant meshes. 
Output = (Distance Value × Multiplier) + Addend. [Distance value is dist of point from river.] 
Fade out or softly control the density of Grass 1 and Grass 2 near the river.
The way they are scattered is through a Perlin noise and a Voronoi noise texture. I've done this to make the distribution a more random. I've applied random rotation according to the normal of the surface. Scale is also randomized between a set range.

The second is a system that scatters rocks and leaves close to the water. The scattering method is the same, but the border control is inverted. That's why I have multiplied the the distance value by -1. This inverts the scattering and now, experimenting with the multiplier and add nodes, I can get more rocks and leaves close to/in the river. 
Random Value Node
Outputs a random float per instance between 0.3 and 4.0.
Multiply Node
Multiplies R by -1.0.

So now:
M=R × −1.0
This just inverts the random value, flipping the range from (0.3 → 4.0) to (-4.0 → -0.3).
The add node randomly subtracts a value between the range from some other scalar input, but clamps it to prevent the value from going below 0.

Why use this entire geo node system?

Normally, to make a terrain like this, first you would have to make the landscape and add all the displacement. And to add the waterbody, you would have to line up the water manually to get the correct height and width. Then add different scatter systems for different vegetation types. And if you would need to make a change to the river for example, you would have to edit the mesh of the landscape again and vice versa.

This geo node approach is procedural & non-destructive. -You can adjust rivers, paths, vegetation zones, etc., in real-time without redoing the terrain.
In a few clicks you have a full terrain with you - one can keep adding scatter systems, more waterbodies. This is very helpful for creating biomes(forest, mountains, arctic).
All the user has to do is have their models/obj files downloaded so that they can drop it into the tree.
You can make custom drivers and keep adding to the node tree and testing new variations in real time.
This geo node setup can be used across multiple projects.

