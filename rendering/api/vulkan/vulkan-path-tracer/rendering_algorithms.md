
# 1. pinhole camera

Under OBJ coordinate system : 
	- positive x-axis points to right
	- positive y-axis points to up
	- posivie z-axis poinst to backward

Steps :
- define camera origin : `vec3(-0.001, 1, 6)`
	  1. Find center of pixel 
		  $$(pixel.x + 0.5, pixel.y + 0.5)$$
	  2. translate the coordinate to the top-left direction as much as half of resolution.
		$$(pixel.x + 0.5 - \frac{resolution.x}{2}, pixel.y + 0.5 - \frac{resolution.y}{2})$$
	3. normalize coordinates.
		- intial range : (-resolution.x/2, resolution.x/2) , (-resolution.y/2, resolution.y/2)
		- normalized coordinates : (-1, 1), (-a, a) where a is called aspect ratio$$(\frac{pixel.x + 0.5 - {resolution.x}/{2}}{resolution.y},\frac{ pixel.y + 0.5 - {resolution.y}/{2}}{resolution.y})$$
		Multiplying the coordinates by 2,$$(\frac{2pixel.x + 1 - {resolution.x}}{resolution.y},\frac{ 2pixel.y + 1 - {resolution.y}}{resolution.y})$$
- calculate ray direction based on camera origin (ray origin)
- initialize ray query
- trace ray until closest hit occurs.
- get `t` coefficient of the current ray (determine the distance from origin)
- turn `t` value into grayscale color
- write the grayscale color into a buffer.


# 2. Depth image

- get `t` value of current intersection
- assign `vec3(t/10.0,t/10.0,t/10.0)` into imageData(storage buffer) to represent a grayscale color

```glsl
...
const float t = rayQueryGetIntersectionTEXT(rayQuery, true);
...
imageData[linearIndex] = pixelColor;
```


# 3. Determine if hit or miss

- use `rayQueryGetIntersectionTypeEXT(rayQueryEXT rayQueryobject, bool committed)`
	returns unsigned integer
	- gl_RayQueryCommittedIntersectionNoneEXT  - miss
	- gl_RayQueryCommittedIntersectionTriangleEXT - triangle
	- gl_RayQueryCommittedIntersectionGeneratedEXT - procedural mesh


# 4. Get triangle indices

- use `rayQueryGetIntersectionPrimitiveIndexEXT`
	- primitiveID - the index of a triangle
	- primitiveID is ranged from 0 to `max_vertices` - 1

# 5. Barycentric coordinates

- tells us where the intersection is , relative to the vertices of the triangle.
- equation :
$$p = (1-b_x-b_y)v0+ b_x{v1} + b_y{v2}$$, where $(1-b_x-b_y)$ is weight of vertex 0, $b_x$ is weight of vertex 1 and $b_y$ is weight of vertex 2 


# 6. Triangle normal

- suppose that a triangle is composed of three vertices($v_0, v_1, v_2$)
- we can get two arbitrary edges of the triangle by using vector calculations
	$$\vec{e_1} = \vec{v_1} - \vec{v_0}$$
	$$\vec{e_2} =\vec{v_2} - \vec{v_0}$$

- If we take a cross product operations of two vectors on the same plance, we can get a vector which is orthogonal to the two vectors
- Normalize the orthogonal vector, then we can get a normal vector of the surface (triangle in this case.)


# 7. Steps of Path tracing

- For each pixel in parallel
	- Repeat some number of times :
		- Choose a ray through the camera, then loop :
			- Find where the ray intersects the scene
			- Get information about the intersection
			- Do
				Estimate how much direct light reaches the intersection
				flows back along the ray, break out the loop
			- or :
				Choose a new ray origin and direction and update the ray's information.
	- Average the pixel's sampled colors in some way.