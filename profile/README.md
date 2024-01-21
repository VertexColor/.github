# The Vertex Color Organisation
Using Vertex Colors can be a fast and painless way to starting with a graphics API like OpenGL.

Choosing vertex colours is either a technical choice or an artistic choice, for programmers not having to deal with images and image compression is a good thing, with PLY files you can use [ZLIB](https://www.zlib.net/) or [ZSTD](https://github.com/facebook/zstd) to compress the vertex/index data if desired.

From an artistic perspective it's like picking your brush and paint type, in 3D rendering these days people tend to pile onto the latest and greatest tech, usually involving some form of deferred shading, lots of lights, and a lot of impressive pixel shaders. Using Vertex Colours helped me to discover my own artistic preferences with it comes to 3D rendering.

## Three main uses of Vertex Colors
* Subdivide mesh and then project a pre-existing UV Texture as Vertex Colors.
* Split mesh into seperate parts that represent its primary colors.
* Blending between two colors on a face can be used to advantage, in the image below the cube pillars all use the automatic color interpolation between vertices to it's advantage to create an intended colour blend, pink to purple.

![Image of a Pink BMW E34 in a Cyberpunk setting](https://camo.githubusercontent.com/6b0807eced228ca80a35a1427aae346c66eb2c0dce203671df39322b5d152612/68747470733a2f2f64617368626f6172642e736e617063726166742e696f2f736974655f6d656469612f6170706d656469612f323032332f31312f53637265656e73686f745f323032332d31312d30315f32312d35372d32372e706e67)

In the image below the token coins and figurines posed as prizes in the coin pusher are ML/AI generated 3D models by [LUMA GENIE](https://lumalabs.ai/genie) that have had their UV Texture maps projected to Vertex Colors in Blender using the [Cycles rendering engine](https://docs.blender.org/manual/en/latest/render/cycles/introduction.html). Each model is ~50,000 triangles uniformly spaced, once "vertex projected" _(I like to call it for the transformation of the UV Texture to Vertex Colors)_, this 50,000 uniform spacing of triangles around an object becomes their own pixels. You can tell a distinct difference between the "vertex projected" models and the penguin coins due to their textural differences which are more traditionally vertex shaded per part as a single color.

It tends to look great after a particular distance, and gets worse than a texture close up, this form of Vertex Color use is expensive on triangle usage ofc, there is the adage that a good texture is all you need, the amount of polygons is more of a luxuary, that is a true statement that casts more of a gluttonous bias upon the expense of textural detail when working with Vertex Colors.

But vertex colors are a bigger saving in the long run, because you don't need to keep high resolution textures around, you basically settle on some vertex density that you are happy with and vertex project it, adding a color to each vertex only needs to be an additional 3 bytes for r,g,b which is already less than UV Mappings at 8 bytes for 2 floats.

You could keep it simple like in the car game above - artistic use of vertex colors on low poly models. I personally really like the art style to it and it can be done realtively cheaply particularly if you only render one color per part as that could mean no color buffer per vertex just update the shader [glUniform3f()](https://registry.khronos.org/OpenGL-Refpages/es2.0/) with batches of parts ordered by color.

![TuxPusher game screenshot displaying Texture to Vertex Color projection objects](https://dashboard.snapcraft.io/site_media/appmedia/2024/01/Screenshot_2024-01-11_05-37-55.png)

## Projecting Textures to Vertex Colors
The easiest method to do this is to use [MeshLab](https://www.meshlab.net/) although these days I use the Cycles rendering engine in [Blender](https://www.blender.org/) 

## Reading and Writing PLY Files
PLY Files are pretty easy to write a custom reader for, in ASCII or BINARY format, so I encourage you to hack together something to convert these buffers into buffers on the GPU. Most software such as [MeshLab](https://www.meshlab.net/) or [Blender](https://www.blender.org/) can export to PLY so really your main concern is reading them into your program somehow.

The [PLY Format](https://paulbourke.net/dataformats/ply/) already closely represents what a GPU requires; typically a vertex buffer and index buffer; and the PLY format specifically supports Vertex Colors.

* Typically a **vertex buffer** is an interleaved array of data [ position, vertex normal, color ] or [ x, y, z, nx, ny, nz, r, g, b ] but we can provide this via OpenGL using [glVertexAttribPointer](https://registry.khronos.org/OpenGL-Refpages/es2.0/) using a stride and offset on a single vertex buffer or just use multiple seperate vertex buffers.
* **To quickly load model data**, the PLY format already supplies the data in Binary format as interleaved rows making it a simple memory copy and access with the right stride and offset in OpenGL, but I do reccomend parsing out each vertex array, position, normal, color, into their own respective arrays if you intend to be doing anything with them.
* **The index buffer** in PLY files is preceded by the amount of vertex used to define a face, usually I work in triangles so that's always 3 for me, but I still need to parse a new index buffer without the preceding face counts for each proceding three vertices that make a triangle face, as OpenGL is generally used with the  [GL_TRIANGLES](https://registry.khronos.org/OpenGL-Refpages/es2.0/) draw mode that expects the supplied index buffer to be of triangle faces only.

## Loading PLY files in C
[The RPLY Project](https://w3.impa.br/~diego/software/rply/) has simple examples, is fast and light-weight to implement into projects.
```
#include <stdio.h>
#include "rply.h"

static int vertex_cb(p_ply_argument argument)
{
    long eol;
    ply_get_argument_user_data(argument, NULL, &eol);
    printf("%g", ply_get_argument_value(argument));
    if (eol) printf("\n");
    else printf(" ");
    return 1;
}

static int face_cb(p_ply_argument argument)
{
    long length, value_index;
    ply_get_argument_property(argument, NULL, &length, &value_index);
    switch (value_index) {
        case 0:
        case 1:
            printf("%g ", ply_get_argument_value(argument));
            break;
        case 2:
            printf("%g\n", ply_get_argument_value(argument));
            break;
        default:
            break;
    }
    return 1;
}

int main(void)
{
    long nvertices, ntriangles;
    p_ply ply = ply_open("input.ply", NULL, 0, NULL);
    if (!ply) return 1;
    if (!ply_read_header(ply)) return 1;
    
    nvertices = ply_set_read_cb(ply, "vertex", "x", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "y",     vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "z",     vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "nx",    vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "ny",    vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "nz",    vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "red",   vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "green", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "blue",  vertex_cb, NULL, 1);
    ntriangles = ply_set_read_cb(ply, "face", "vertex_indices", face_cb, NULL, 0);

    printf("%ld\n%ld\n", nvertices, ntriangles);
    if (!ply_read(ply)) return 1;
    ply_close(ply);
    return 0;
}
```

## OpenGL Vertex & Fragment Shaders for Vertex Colors [per-pixel lighting]
**vertex shader**
```
#version 100
uniform mat4 modelview;
uniform mat4 projection;
uniform float ambient;
uniform float saturate;
uniform float opacity;
uniform vec3 lightpos;
attribute vec4 position;
attribute vec3 normal;
attribute vec3 color;
varying vec3 vertPos;
varying vec3 vertNorm;
varying vec3 vertCol;
varying float vertAmb;
varying float vertSat;
varying float vertOpa;
varying vec3 vLightPos;
void main()
{
  vec4 vertPos4 = modelview * position;
  vertPos = vertPos4.xyz / vertPos4.w;
  vertNorm = vec3(modelview * vec4(normal, 0.0));
  vertCol = color;
  vertAmb = ambient;
  vertSat = saturate;
  vertOpa = opacity;
  vLightPos = lightpos;
  gl_Position = projection * vertPos4;
}
```
**fragment shader**
```
#version 100
precision highp float;
varying vec3 vertPos;
varying vec3 vertNorm;
varying vec3 vertCol;
varying float vertAmb;
varying float vertSat;
varying float vertOpa;
varying vec3 vLightPos;
void main()
{
  vec3 lightDir = normalize(vLightPos - vertPos);
  float lambertian = min(max(dot(lightDir, normalize(vertNorm)), 0.0), vertSat);
  gl_FragColor = vec4((vertCol*vertAmb) + (vertCol*lambertian), vertOpa);
}
```

## OpenGL Vertex & Fragment Shaders for Vertex Colors [per-vertex lighting]
**vertex shader**
```
#version 100
uniform mat4 modelview;
uniform mat4 projection;
uniform float ambient;
uniform float saturation;
uniform float opacity;
uniform vec3 lightpos;
attribute vec4 position;
attribute vec3 normal;
attribute vec3 color;
varying vec4 fragcolor;
void main()
{
  vec4 vertPos4 = modelview * position;
  vec3 vertNorm = normalize(vec3(modelview * vec4(normal, 0.0)));
  vec3 lightDir = normalize(lightpos - (vertPos4.xyz / vertPos4.w));
  fragcolor = vec4((color*ambient) + (color * min(max(dot(lightDir, vertNorm), 0.0), saturation)), opacity);
  gl_Position = projection * vertPos4;
}
```
**fragment shader**
```
#version 100
precision highp float;
varying vec4 fragcolor;
void main()
{
  gl_FragColor = fragcolor;
}
```

## Specular Mapping
The best method to implement specular mapping would be to pass a specular map to the Ambient paramerter, this would allow you to have brighter highlights on metalic parts of a model by boosting it's ambient light value for those vertices.

## Using the OpenGL shaders
I reccomend using [GLFW](https://www.glfw.org/) or [SDL](https://www.libsdl.org/) as a portable method of instantiating a window to render to and obtaining user inputs. With these you can use the [esAux6.h](https://gist.github.com/mrbid/8563c765116f2dce3d4461adea15fdd1) header that I created. It will also require [vec.h](https://gist.github.com/mrbid/77a92019e1ab8b86109bf103166bd04e) and [mat.h](https://gist.github.com/mrbid/cbc69ec9d99b0fda44204975fcbeae7c).

You can use [ptf2.c](https://gist.github.com/mrbid/35b1d359bddd9304c1961c1bf0fcb882) to convert ASCII PLY files into C Header Files that contain the individual buffers required for rendering in OpenGL _(or use [RPLY](https://github.com/VertexColor#loading-ply-files-in-c) to load them from file)_.

The shader uses two main systems for lighting models, first it has a view-space light that you can set the offset position of - if the position is unset (0,0,0) then the light will always be at the postion of the camera, some times you might want to increase the height of the light from the player or extend it out a little. It's preference to use on view-space light than many world-space lights.

There are then two main parameters per object rendered:
* Ambient - This defines how much environmental light the model naturally reflects.
* Saturate - This clamps the max lightness value of the model being rendered, this can prevent a model having overly bright spots.

Opacity can also be set per model but requires [GL_BLEND](https://registry.khronos.org/OpenGL-Refpages/es2.0/) to be enabled with some blending function such as `glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);`.

**Some notes from the esAux6.h source file**
```
No Textures, No Phong, One view-space light with position, ambient and saturation control.
Default: ambient = 0.648, saturate = 0.26 or 1.0

The model header files have their own register function with the new ptf2.c
the idea is simply that the order you call the "<model-name>_register()" functions in
is the index of the model loaded, so the first 0, then, 1 and so on.
Then you use this index to call esBindModel(id) and esRenderModel()
    or just esBindRender(id).

//#define VERTEX_SHADE // uncomment for vertex shaded, default is pixel shaded
//#define MAX_MODELS 32 // uncomment to enable the use of esBindModel(id) and esRenderModel() or just esBindRender(id)
//#define GL_DEBUG // allows you to use esDebug(1); to enable OpenGL errors to the console.
                    // https://gen.glad.sh/ and https://glad.dav1d.de/ might help
//#define MODEL_DATA_STRIDED // uncomment to load vertex data into GPU memory from one strided vertex buffer
```

**An example program using GLFW that loads a `test.ply` file as strided data and then renders it**
```
/*
    James William Fletcher (github.com/mrbid)
        January 2024

    https://github.com/VertexColor

    cc main.c rply.c glad_gl.c -I inc -Ofast -lglfw -lm -o plv
*/

#include <stdio.h>
#include <stdlib.h>
#define uint GLuint
#define sint GLint
#include "gl.h"
#define GLFW_INCLUDE_NONE
#include "glfw3.h"
#define fTime() (float)glfwGetTime()
#define MAX_MODELS 1 // hard limit, be aware and increase if needed
#define MODEL_DATA_STRIDED // load vertex data as strided onto the GPU
#include "esAux6.h"
#include "rply.h"

const char appTitle[]="PLY Viewer";
uint winw=1024, winh=768;
GLFWwindow* wnd;
mat projection, model;
void updateModel(){glUniformMatrix4fv(modelview_id, 1, GL_FALSE, (float*)&model.m[0][0]);}

// load model from file to gpu memory with a permanent 12 MB staging buffer
#define MAX_SIZE 2097152
GLfloat vertex_buffer[MAX_SIZE];
GLushort index_buffer[MAX_SIZE];
uint vbl = 0, ibl = 0; // buffer lens
uint ntris = 0, nverts = 0;
static int vertex_cb(p_ply_argument argument)
{
    if(vbl > MAX_SIZE-1){return 0;}
    static uint vc = 0;
    long eol;
    ply_get_argument_user_data(argument, NULL, &eol);
    vertex_buffer[vbl] = ply_get_argument_value(argument);
    if(vc > 5){vertex_buffer[vbl] *= 0.003921569f;}
    vbl++;
    vc++;
    if(eol){vc = 0;}
    return 1;
}
static int face_cb(p_ply_argument argument)
{
    if(ibl > MAX_SIZE-1){return 0;}
    long length, value_index;
    ply_get_argument_property(argument, NULL, &length, &value_index);
    switch(value_index)
    {
        case 0:
        case 1: 
            index_buffer[ibl] = ply_get_argument_value(argument);
            ibl++;
            break;
        case 2:
            index_buffer[ibl] = ply_get_argument_value(argument);
            ibl++;
            break;
        default: 
            break;
    }
    return 1;
}
void loadModel(const char* fp)
{
    // reset buffers
    vbl = 0, ibl = 0;

    // open file
    p_ply ply = ply_open(fp, NULL, 0, NULL);
    if(!ply){esModelArray_index++; return;}
    if(!ply_read_header(ply))
    {
        ply_close(ply);
        esModelArray_index++;
        return;
    }

    // read file setup
    nverts = ply_set_read_cb(ply, "vertex", "x", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "y", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "z", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "nx", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "ny", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "nz", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "red", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "green", vertex_cb, NULL, 0);
    ply_set_read_cb(ply, "vertex", "blue", vertex_cb, NULL, 1);
    ntris = ply_set_read_cb(ply, "face", "vertex_indices", face_cb, NULL, 0);

    // read file
    if(!ply_read(ply))
    {
        ply_close(ply);
        esModelArray_index++;
        return;
    }

    // close file
    ply_close(ply);

    // bind to gpu
    esBind(GL_ARRAY_BUFFER, &esModelArray[esModelArray_index].vid, vertex_buffer, vbl*sizeof(GLfloat), GL_STATIC_DRAW);
    esBind(GL_ELEMENT_ARRAY_BUFFER, &esModelArray[esModelArray_index].iid, index_buffer, ibl*sizeof(GLushort), GL_STATIC_DRAW);
    esModelArray[esModelArray_index].itp = GL_UNSIGNED_SHORT;
    esModelArray[esModelArray_index].ni = ibl;
    printf("Loaded PLY: %u %u %u\n", esModelArray_index+1, vbl, ibl);
    esModelArray_index++;
}

// process entry point
int main(int argc, char** argv)
{
    // create window with custom MSAA level
    int msaa = 16;
    if(argc >= 2){msaa = atoi(argv[1]);}
    if(!glfwInit()){printf("glfwInit() failed.\n"); exit(EXIT_FAILURE);}
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);
    glfwWindowHint(GLFW_SAMPLES, msaa);
    glfwWindowHint(GLFW_RESIZABLE, 0);
    wnd = glfwCreateWindow(winw, winh, appTitle, NULL, NULL);
    if(!wnd)
    {
        printf("glfwCreateWindow() failed.\n");
        glfwTerminate();
        exit(EXIT_FAILURE);
    }
    const GLFWvidmode* desktop = glfwGetVideoMode(glfwGetPrimaryMonitor());
    glfwSetWindowPos(wnd, (desktop->width/2)-(winw/2), (desktop->height/2)-(winh/2)); // center window on desktop
    glfwMakeContextCurrent(wnd);
    gladLoadGL(glfwGetProcAddress);
    glfwSwapInterval(1); // 0 for immediate updates, 1 for updates synchronized with the vertical retrace, -1 for adaptive vsync

    // load our test model
    loadModel("test.ply");

    // configure render options
    glEnable(GL_CULL_FACE);
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.f, 0.f, 0.f, 0.f);
    makeLambert();
    shadeLambert(&position_id, &projection_id, &modelview_id, &lightpos_id, &normal_id, &color_id, &ambient_id, &saturate_id, &opacity_id);
    glViewport(0, 0, winw, winh);
    mIdent(&projection);
    mPerspective(&projection, 55.0f, (float)winw / (float)winh, 0.01f, 64.f);
    glUniformMatrix4fv(projection_id, 1, GL_FALSE, (float*)&projection.m[0][0]);
    glUniform1f(ambient_id, 0.32f);
    glUniform1f(saturate_id, 1.f);

    // render loop
    while(!glfwWindowShouldClose(wnd))
    {
        // Poll GLFW events so that we know when the window is closed
        glfwPollEvents(); 

        // clear buffer
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        // render model
        mIdent(&model);
        mRotX(&model, 90.f * DEG2RAD);
        mRotY(&model, 90.f * DEG2RAD);
        mRotZ(&model, fTime() * 1.2f);
        mSetPos3(&model, 0.f, 0.f, -2.f);
        updateModel();
        esBindRender(0);

        // display render
        glfwSwapBuffers(wnd);
    }

    // end
    glfwDestroyWindow(wnd);
    glfwTerminate();
    exit(EXIT_SUCCESS);
    return 0;
}
```
**Here is the full source code repository:** https://github.com/VertexColor/PLY-View

## More options for compiling PLY files into C Header Files as memory buffers
We are currently working on a more efficient program to mass convert PLY BINARY files to C Header Files.

