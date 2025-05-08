# HW5 readme
## What you need
You need Visual Studio 2022 and window 11 OS.

And C/C++ should be available in VS2022.

## About
This project is about Rasterizer.

To view the result image, open result.png, 

and if you want an explanation of the code, scroll down below.

## How to run

1. Click code and download as zip file.
   
![image](https://github.com/user-attachments/assets/bbae1c64-faea-443f-976d-671e86e3c519)


3. Unzip a download file

![image](https://github.com/user-attachments/assets/02ce25e2-37ec-4fe7-823b-380dabb302b4)


3. Open hw2_Q1-master. Double click hw2_Q1-master and opne OpenglViewer.sln

![image](https://github.com/user-attachments/assets/1bf5961d-c697-4b59-98bf-85451fbdbbcb)



5. click "F5" on your keybord. Then you will get the result.

![image](https://github.com/user-attachments/assets/592f3dd3-35b4-4229-a20d-2478eec89a18)


## Code explanation
```
const int WIDTH = 512;
const int HEIGHT = 512;

std::vector<vec3> gVertices;
std::vector<int> gIndexBuffer;
std::vector<float> DepthBuffer(WIDTH* HEIGHT, 1e9f);
std::vector<float> OutputImage(WIDTH* HEIGHT * 3, 0.0f);

```
make viewport 512 * 512 and declare vector

-----------
```
void create_scene() {
    int width = 32;
    int height = 16;
    float theta, phi;

    gVertices.resize((height - 2) * width + 2);
    gIndexBuffer.resize((height - 2) * (width - 1) * 6 + 6 * (width - 1));

    int t = 0;
    for (int j = 1; j < height - 1; ++j) {
        for (int i = 0; i < width; ++i) {
            theta = (float)j / (height - 1) * M_PI;
            phi = (float)i / (width - 1) * 2 * M_PI;
            float x = sinf(theta) * cosf(phi);
            float y = cosf(theta);
            float z = -sinf(theta) * sinf(phi);
            gVertices[t++] = vec3(x, y, z);
        }
    }
    gVertices[t++] = vec3(0, 1, 0);
    gVertices[t++] = vec3(0, -1, 0);

    t = 0;
    for (int j = 0; j < height - 3; ++j) {
        for (int i = 0; i < width - 1; ++i) {
            gIndexBuffer[t++] = j * width + i;
            gIndexBuffer[t++] = (j + 1) * width + (i + 1);
            gIndexBuffer[t++] = j * width + (i + 1);
            gIndexBuffer[t++] = j * width + i;
            gIndexBuffer[t++] = (j + 1) * width + i;
            gIndexBuffer[t++] = (j + 1) * width + (i + 1);
        }
    }
    for (int i = 0; i < width - 1; ++i) {
        gIndexBuffer[t++] = (height - 2) * width;
        gIndexBuffer[t++] = i;
        gIndexBuffer[t++] = i + 1;
        gIndexBuffer[t++] = (height - 2) * width + 1;
        gIndexBuffer[t++] = (height - 3) * width + (i + 1);
        gIndexBuffer[t++] = (height - 3) * width + i;
    }
}
```
Create sphere scene using sphere_scene.cpp (given code)

--------------
```
void rasterize_triangle(vec4 v0, vec4 v1, vec4 v2) {
    // Homogeneous divide
    vec3 p0 = vec3(v0) / v0.w;
    vec3 p1 = vec3(v1) / v1.w;
    vec3 p2 = vec3(v2) / v2.w;

    // Viewport transform
    auto to_screen = [](vec3 p) -> vec2 {
        return vec2((p.x + 0.5f) * WIDTH, (p.y + 0.5f) * HEIGHT);
        };

    vec2 s0 = to_screen(p0);
    vec2 s1 = to_screen(p1);
    vec2 s2 = to_screen(p2);

    float minX = std::max(0.0f, floor(std::min({ s0.x, s1.x, s2.x })));
    float maxX = std::min((float)WIDTH - 1, ceil(std::max({ s0.x, s1.x, s2.x })));
    float minY = std::max(0.0f, floor(std::min({ s0.y, s1.y, s2.y })));
    float maxY = std::min((float)HEIGHT - 1, ceil(std::max({ s0.y, s1.y, s2.y })));

    for (int y = (int)minY; y <= (int)maxY; ++y) {
        for (int x = (int)minX; x <= (int)maxX; ++x) {
            vec2 p(x + 0.5f, y + 0.5f);
            vec2 v0v1 = s1 - s0, v0p = p - s0;
            vec2 v1v2 = s2 - s1, v1p = p - s1;
            vec2 v2v0 = s0 - s2, v2p = p - s2;

            float a = v0v1.x * v0p.y - v0v1.y * v0p.x;
            float b = v1v2.x * v1p.y - v1v2.y * v1p.x;
            float c = v2v0.x * v2p.y - v2v0.y * v2p.x;

            if ((a >= 0 && b >= 0 && c >= 0) || (a <= 0 && b <= 0 && c <= 0)) {
                float depth = (p0.z + p1.z + p2.z) / 3.0f;
                int idx = y * WIDTH + x;
                if (depth < DepthBuffer[idx]) {
                    DepthBuffer[idx] = depth;
                    OutputImage[3 * idx + 0] = 1.0f;
                    OutputImage[3 * idx + 1] = 1.0f;
                    OutputImage[3 * idx + 2] = 1.0f;
                }
            }
        }
    }
}
}
```
Make rasterize triangle algorithm using below image. mark as white.

![image](https://github.com/user-attachments/assets/ba60885d-ff80-40c8-b4a9-95b72a414655)


-------------
```
void render_scene() {
    create_scene();

    mat4 model = glm::scale(mat4(1.0f), vec3(2.0f));
    model = glm::translate(model, vec3(0, 0, -7));

    mat4 view = lookAt(vec3(0.0f, 0.0f, 0.0f), vec3(0.0f, 0.0f, -1.0f), vec3(0.0f, 1.0f, 0.0f));

    mat4 persp = glm::frustum(-0.1f, 0.1f, -0.1f, 0.1f, -0.1f, -1000.0f);

    mat4 ortho = glm::scale(mat4(1.0f), vec3(1, 1, -1));

    mat4 MVP = ortho * persp * view * model;

    for (int i = 0; i < gIndexBuffer.size(); i += 3) {
        vec4 v0 = MVP * vec4(gVertices[gIndexBuffer[i]], 1.0f);
        vec4 v1 = MVP * vec4(gVertices[gIndexBuffer[i + 1]], 1.0f);
        vec4 v2 = MVP * vec4(gVertices[gIndexBuffer[i + 2]], 1.0f);
        rasterize_triangle(v0, v1, v2);
    }
}
```
render scene 

-------------
### Important Idea. PhongShading funtion

```
int main() {
    if (!glfwInit()) return -1;
    GLFWwindow* window = glfwCreateWindow(WIDTH, HEIGHT, "Software Rasterizer", NULL, NULL);
    if (!window) return -1;
    glfwMakeContextCurrent(window);

    render_scene();

    while (!glfwWindowShouldClose(window)) {
        glClear(GL_COLOR_BUFFER_BIT);
        glDrawPixels(WIDTH, HEIGHT, GL_RGB, GL_FLOAT, &OutputImage[0]);
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwDestroyWindow(window);
    glfwTerminate();
    return 0;
}
}
```

We have to implement code based on the picture below.

![image](https://github.com/user-attachments/assets/2cdfa6ff-92ea-4b94-b4d4-a69b7c7b5a4b)

"return ambient + diffuse + specular;" final code represent this picture.

Each variable explanation is here.

**1. ambient**

ambient get surface.ka value.


**2. diffuse**

Diffuse is calculated using the dot product of the lightDir and normal vectors.

![image](https://github.com/user-attachments/assets/6e8f5e36-fe64-4573-8a34-2919fc81b26c)


**3. specular**

Specular is calculated using the dot product of the halfDir and normal vectors.

Half vector is calculated by averaging the lightDir and viewDir vectors, and then normalizing it.

![image](https://github.com/user-attachments/assets/4487e1c1-0ea7-48a8-979e-1575da8fe3f9)

End of three bariable explanation.

Next we can see shadowRay.

Because of shadow rounding errors, we set shadowRay.origin = hitPoint + normal * 1e-4f

and we can check shadow clearly.
