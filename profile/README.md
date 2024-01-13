# The Vertex Color Organisation
Using Vertex Colors can be a fast and painless way to starting with a graphics API like OpenGL.

## Three main uses of Vertex Colors
* Subdivide mesh and then project a pre-existing UV Texture as Vertex Colors
* Split mesh into seperate parts that represent it's primary colors.
* Blending between two colors on a face can be used to advantage, in the image below the cube pillars all use the automatic color interpolation between vertices to it's advantage to create an intended colour blend pink to purple.

![Image of a Pink BMW E34 in a Cyberpunk setting](https://camo.githubusercontent.com/6b0807eced228ca80a35a1427aae346c66eb2c0dce203671df39322b5d152612/68747470733a2f2f64617368626f6172642e736e617063726166742e696f2f736974655f6d656469612f6170706d656469612f323032332f31312f53637265656e73686f745f323032332d31312d30315f32312d35372d32372e706e67)

## Reading and Writing PLY Files
PLY Files are pretty easy to write a custom reader for, in ASCII or BINARY format, so I encourage you to hack together something to convert these buffers into buffers on the GPU. Most software such as [MeshLab](https://www.meshlab.net/) or [Blender](https://www.blender.org/) can export to PLY so really your main concern is reading usually.

The [PLY Format](https://paulbourke.net/dataformats/ply/) already closely represents what a GPU requires; typically a vertex buffer and index buffer ... and it supports Vertex Colors!

* Typically a **vertex buffer** is an interleaved array of data [ position, vertex normal, color ] or [ x, y, z, nx, ny, nz, r, g, b ] but we can provide this via OpenGL using [glVertexAttribPointer](https://registry.khronos.org/OpenGL-Refpages/es2.0/) using a stride and offset on a single vertex buffer or just use multiple seperate vertex buffers.
* **To quickly load model data**, the PLY format already supplies the data in Binary format as interleaved rows making it a simple memory copy and access with the right stride and offset in OpenGL, but I do reccomend parsing out each vertex array, position, normal, color, into their own respective arrays if you intend to be doing anything with them.
* **The index buffer** in PLY files is preceded by the amount of vertex used to define a face, usually I work in triangles so that's always 3 for me, but I still need to parse a new index buffer without the preceding face counts for each proceding three vertices that make a triangle face, as OpenGL is generally used with the  [GL_TRIANGLES](https://registry.khronos.org/OpenGL-Refpages/es2.0/) draw mode that expects the supplied index buffer is to be of triangle faces only.

## Loading PLY files in C
[The RPLY Project](https://w3.impa.br/~diego/software/rply/) has simple examples, is fast and light-weight to implement into projects.


