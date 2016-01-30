---
layout: post
title:  "OpenGL ES2 API"
date:   2016-01-27 15:34:55
categories: OpenGL iOS
comments: true
---
##OpenGL ES2 API

----------
###**glClear(GL_COLOR_BUFFER_BIT)**
http://docs.gl/es2/glClear

glClear sets the bitplane area of the window to values previously selected by glClearColor, glClearDepth, and glClearStencil.

使用之前glClearColor, glClearDepth, glClearStencil設定的值, 清除緩衝區

參數:
GL_COLOR_BUFFER_BIT
Indicates the buffers currently enabled for color writing.

GL_DEPTH_BUFFER_BIT
Indicates the depth buffer.

GL_STENCIL_BUFFER_BIT
Indicates the stencil buffer.


----------
###**void glGenBuffers(	GLsizei  	n,GLuint *  	buffers)**
http://docs.gl/es2/glGenBuffers

generate buffer object names, 建立緩衝區

**Parameters**
n
Specifies the number of buffer object names to be generated.

buffers
Specifies an array in which the generated buffer object names are stored.



----------
###**void glBindBuffer(	GLenum target, GLuint buffer);**

bind a named buffer object

**Parameters**
target
Specifies the target to which the buffer object is bound. The symbolic constant must be GL_ARRAY_BUFFER or GL_ELEMENT_ARRAY_BUFFER.

buffer
Specifies the name of a buffer object.



----------
###**void glBufferData(	GLenum target, GLsizeiptr size,const GLvoid * data, GLenum usage);**

create and initialize a buffer object's data store

**Parameters**
target
Specifies the target buffer object. The symbolic constant must be GL_ARRAY_BUFFER or GL_ELEMENT_ARRAY_BUFFER.

size
Specifies the size in bytes of the buffer object's new data store.

data
Specifies a pointer to data that will be copied into the data store for initialization, or NULL if no data is to be copied.

usage
Specifies the expected usage pattern of the data store. The symbolic constant must be GL_STREAM_DRAW, GL_STATIC_DRAW, or GL_DYNAMIC_DRAW.



----------
###**glEnableVertexAttribArray (GLuint index)**
http://docs.gl/es2/glEnableVertexAttribArray

enable a generic vertex attribute array, 給shader存取
使用後需disable

參數:
typedef NS_ENUM(GLint, GLKVertexAttrib)
{
    GLKVertexAttribPosition,
    GLKVertexAttribNormal,
    GLKVertexAttribColor,
    GLKVertexAttribTexCoord0,
    GLKVertexAttribTexCoord1
} NS_ENUM_AVAILABLE(10_8, 5_0);


----------
###**void glVertexAttribPointer( GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid * pointer);**

需要先enable vertex attribute
define an array of generic vertex attribute data

**Parameters**
Parameters
index
Specifies the index of the generic vertex attribute to be modified.
參數:
typedef NS_ENUM(GLint, GLKVertexAttrib)
{
    GLKVertexAttribPosition,
    GLKVertexAttribNormal,
    GLKVertexAttribColor,
    GLKVertexAttribTexCoord0,
    GLKVertexAttribTexCoord1
} NS_ENUM_AVAILABLE(10_8, 5_0);

size vertex中的屬性要多少size (如果x,y,z就是3)
Specifies the number of components per generic vertex attribute. Must be 1, 2, 3, or 4. The initial value is 4.

type 參數類型(x y z 使用的類型)
Specifies the data type of each component in the array. Symbolic constants GL_BYTE, GL_UNSIGNED_BYTE, GL_SHORT, GL_UNSIGNED_SHORT, GL_FIXED, or GL_FLOAT are accepted. The initial value is GL_FLOAT.

normalized 是否要normalize, 一般為false
Specifies whether fixed-point data values should be normalized (GL_TRUE) or converted directly as fixed-point values (GL_FALSE) when they are accessed.

stride 一個vertex的size
Specifies the byte offset between consecutive generic vertex attributes. If stride is 0, the generic vertex attributes are understood to be tightly packed in the array. The initial value is 0.

pointer  偏移量 可使用(const GLvoid *) offsetof(Vertex,XXX)
Specifies a pointer to the first component of the first generic vertex attribute in the array. The initial value is 0.


----------
###**void glDrawElements( GLenum mode, GLsizei count, GLenum type, const GLvoid * indices);**

render primitives from array data

**Parameters**
mode
Specifies what kind of primitives to render. Symbolic constants GL_POINTS, GL_LINE_STRIP, GL_LINE_LOOP, GL_LINES, GL_TRIANGLE_STRIP, GL_TRIANGLE_FAN and GL_TRIANGLES are accepted.

count 有幾個vertex
Specifies the number of elements to be rendered.

type
Specifies the type of the values in indices. Must be one of GL_UNSIGNED_BYTE, GL_UNSIGNED_SHORT, or GL_UNSIGNED_INT.

indices
Specifies a byte offset (cast to a pointer type) into the buffer bound to GL_ELEMENT_ARRAY_BUFFER to start reading indices from. If no buffer is bound, specifies a pointer to the location where the indices are stored.



----------
Example:

{% highlight objective-c %}

typedef struct {
  GLfloat Position[2];
  GLfloat TexCoord[2];
} Vertex;

Vertex* _verticies;

GLuint _index_buffer;
GLuint _vertex_buffer;

GLushort* _indices;
GLushort _vertex_num;

glGenBuffers(1, &_vertex_buffer);
glBindBuffer(GL_ARRAY_BUFFER, _vertex_buffer);
glBufferData(GL_ARRAY_BUFFER, _vertex_num * sizeof(Vertex), _verticies, GL_STATIC_DRAW);

glGenBuffers(1, &_index_buffer);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _index_buffer);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, _vertex_num * sizeof(GLushort), _indices, GL_STATIC_DRAW);

glEnableVertexAttribArray(GLKVertexAttribPosition);
glEnableVertexAttribArray(GLKVertexAttribTexCoord0);

glVertexAttribPointer(GLKVertexAttribPosition, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex), (const GLvoid *) offsetof(Vertex, Position));
glVertexAttribPointer(GLKVertexAttribTexCoord0, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex), (const GLvoid *) offsetof(Vertex, TexCoord));

glClear(GL_COLOR_BUFFER_BIT);

glDrawElements(GL_TRIANGLES, _vertex_num, GL_UNSIGNED_SHORT, 0);

{% endhighlight %}
