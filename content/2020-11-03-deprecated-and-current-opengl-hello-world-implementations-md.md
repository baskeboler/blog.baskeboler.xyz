---
title: Deprecated and Current OpenGL "Hello World" implementations
date: 2020-11-04T00:37:18.639Z
cover: /assets/opengl.jpeg
slug: deprecated-vs-current-opengl
category: programming
tags:
  - opengl
  - c++
  - glsl
---
This is what I was taught in school:

```cpp
#include <GL/glut.h>
void display(void) {
  glClear(GL_COLOR_BUFFER_BIT);
  glBegin(GL_POLYGON);
  glVertex2f(-0.5, -0.5);
  glVertex2f(-0.5, 0.5);
  glVertex2f(0.5, 0.5);
  glVertex2f(0.5, -0.5);
  glEnd();
  glutSwapBuffers();
}
int main(int argc, char **argv) {
  glutInit(&argc, argv);
  glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE);
  glutCreateWindow("HelloWorld");
  glutDisplayFunc(display);
  glutMainLoop();
}
```

This is the new way to do it:

```cpp
#include "Angel.h"
void init(void) {
  vec2points[6] = {vec2(-0.5, -0.5), vec2(0.5, -0.5), vec2(0.5, 0.5),
                   vec2(0.5, 0.5),   vec2(-0.5, 0.5), vec2(-0.5, -0.5)};
  GLuintvao, buffer;
  GLuintglGenVertexArrays(1, &vao);
  glBindVertexArray(vao);
  GLuintglGenBuffers(1, &buffer);
  glBindBuffer(GL_ARRAY_BUFFER, buffer);
  glBufferData(GL_ARRAY_BUFFER, sizeof(points), points, GL_STATIC_DRAW);
  GLuintprogram = InitShader("vsimple.glsl", "fsimple.glsl");
  glUseProgram(program);
  GLuintloc = glGetAttribLocation(program, "vPosition");
  glEnableVertexAttribArray(loc);
  glVertexAttribPointer(loc, 2, GL_FLOAT, GL_FALSE, 0, 0);
  glClearColor(0.0, 0.0, 0.0, 1.0);
}
void display(void) {
  glClear(GL_COLOR_BUFFER_BIT);
  glDrawArrays(GL_TRIANGLES, 0, 6);
  gutSwapBuffers();
}
int main(int argc, char **argv) {
  glutInit(&argc, argv);
  glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE);
  glutCreateWindow("HelloWorld");
  init();
  glutDisplayFunc(display);
  glutMainLoop();
}
```

```glsl
in vec4 vPosition;
void main() {
  gl_Position = vPosition;
}
```

```glsl
out vec4 FragColor;
void main() {
  FragColor = vec4(1.0, 1.0, 1.0, 1.0);
}
```
