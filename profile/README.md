# The Vertex Color Organisation
Using Vertex Colors can be a fast and painless way to starting with a graphics API like OpenGL.

Choosing vertex colours is either a technical choice or an artistic choice, for programmers not having to deal with images and image compression is a good thing, with PLY files you can use [ZLIB](https://www.zlib.net/) or [ZSTD](https://github.com/facebook/zstd) to compress the vertex/index data if desired.

From an artistic perspective it's like picking your brush and paint type, in 3D rendering these days people tend to pile onto the latest and greatest tech, usually involving some form of deferred shading, lots of lights, and a lot of impressive pixel shaders. Using Vertex Colours helped me to discover my own artistic preferences with it comes to 3D rendering.

## Three main uses of Vertex Colors
* Subdivide mesh and then project a pre-existing UV Texture as Vertex Colors.
* Split mesh into seperate parts that represent its primary colors.
* Blending between two colors on a face can be used to advantage, in the image below the cube pillars all use the automatic color interpolation between vertices to it's advantage to create an intended colour blend, pink to purple.

![Image of a Pink BMW E34 in a Cyberpunk setting](https://camo.githubusercontent.com/6b0807eced228ca80a35a1427aae346c66eb2c0dce203671df39322b5d152612/68747470733a2f2f64617368626f6172642e736e617063726166742e696f2f736974655f6d656469612f6170706d656469612f323032332f31312f53637265656e73686f745f323032332d31312d30315f32312d35372d32372e706e67)

In the image below the token coins and figurines posed as prizes in the coin pusher are ML/AI generated 3D models by [LUMA GENIE](https://lumalabs.ai/genie) that have had their UV Texture maps projected to Vertex Colors in Blender using the [Cycles rendering engine](https://docs.blender.org/manual/en/latest/render/cycles/introduction.html). Each model is ~50,000 triangles uniformly spaced, once "vertex projected" I like to call it for the transformation of the UV Texture to Vertex Colors, this 50,000 uniform spacing of triangles around an object becomes their own pixels. You can tell a distinct difference between the "vertex projected" models and the penguin coins due to their textural differences which are more traditionally vertex shaded per part as a single color.

It tends to look great after a particular distance, and gets worse than a texture close up, and this form of Vertex Color use is expensive on triangle usage ofc, there is the adage that a good texture is all you need, the amount of polygons is more of a luxuary, that is a true statement that casts more of a gluttonous bias upon the expense of textural detail when working with Vertex Colors.

But vertex colors are a bigger saving in the long run, because you don't need to keep high resolution textures around, you basically settle on some vertex density that you are happy with and vertex project it, adding a color to each vertex only needs to be an additional 3 bytes for r,g,b which is already less than UV Mappings at 8 bytes for 2 floats.

You could keep it simple like in the car game above, artistic use of vertex colors on low poly models, which is a great benefit over textures, I personally really like the art style to it and it can be done realtively cheaply, particularly if you only had one color per part rendered, that could mean no color buffer per vertex just update the shader [glUniform3f()](https://registry.khronos.org/OpenGL-Refpages/es2.0/) with batches of parts ordered by color.

![TuxPusher game screenshot displaying Texture to Vertex Color projection objects](https://dashboard.snapcraft.io/site_media/appmedia/2024/01/Screenshot_2024-01-11_05-37-55.png)

## Reading and Writing PLY Files
PLY Files are pretty easy to write a custom reader for, in ASCII or BINARY format, so I encourage you to hack together something to convert these buffers into buffers on the GPU. Most software such as [MeshLab](https://www.meshlab.net/) or [Blender](https://www.blender.org/) can export to PLY so really your main concern is reading them into your program somehow.

The [PLY Format](https://paulbourke.net/dataformats/ply/) already closely represents what a GPU requires; typically a vertex buffer and index buffer; and the PLY format specifically supports Vertex Colors.

* Typically a **vertex buffer** is an interleaved array of data [ position, vertex normal, color ] or [ x, y, z, nx, ny, nz, r, g, b ] but we can provide this via OpenGL using [glVertexAttribPointer](https://registry.khronos.org/OpenGL-Refpages/es2.0/) using a stride and offset on a single vertex buffer or just use multiple seperate vertex buffers.
* **To quickly load model data**, the PLY format already supplies the data in Binary format as interleaved rows making it a simple memory copy and access with the right stride and offset in OpenGL, but I do reccomend parsing out each vertex array, position, normal, color, into their own respective arrays if you intend to be doing anything with them.
* **The index buffer** in PLY files is preceded by the amount of vertex used to define a face, usually I work in triangles so that's always 3 for me, but I still need to parse a new index buffer without the preceding face counts for each proceding three vertices that make a triangle face, as OpenGL is generally used with the  [GL_TRIANGLES](https://registry.khronos.org/OpenGL-Refpages/es2.0/) draw mode that expects the supplied index buffer to be of triangle faces only.

## Loading PLY files in C
[The RPLY Project](https://w3.impa.br/~diego/software/rply/) has simple examples, is fast and light-weight to implement into projects.

## Compiling PLY files into C as memory buffers
...

## Defining the VCO format, Vertex Color Object.
...
