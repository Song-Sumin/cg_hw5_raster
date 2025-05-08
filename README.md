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


2. Unzip a download file

![image](https://github.com/user-attachments/assets/02ce25e2-37ec-4fe7-823b-380dabb302b4)


4. Open hw2_Q1-master. Double click hw2_Q1-master and opne OpenglViewer.sln
![image](https://github.com/user-attachments/assets/7c0ca433-d940-42fd-a2a3-3893344c0344)


5. click "F5" on your keybord. Then you will get the result.
![image](https://github.com/user-attachments/assets/559d3c38-31c0-4d00-82d8-61836b1e78ec)

## Code explanation
```
vec3 lightPos(-4.0f, 4.0f, -3.0f);
vec3 lightColor(1.0f, 1.0f, 1.0f);
```
Set position of the light and color white.

-----------
```
	vec3 ka, kd, ks;
	float specularPower;
	virtual vec3 getNormal(const vec3& point) const = 0;
```
In class surface, declare vectors representing the material properties of an object:

ka: Ambient coefficient. This defines how much ambient light the material reflects.

kd: Diffuse coefficient. This defines how much diffuse light the material reflects.

ks: Specular coefficient. This defines how much specular light the material reflects.

specularPower define shininess of the specular hightlight.

get normal function return normal vector at that point.

![image](https://github.com/user-attachments/assets/81aa8f94-b64b-40e4-8832-9a341ba54165)

--------------
```
vec3 getNormal(const vec3& point) const override {
	return normalize(point - center);
}
Sphere(const vec3& c, float r, const vec3& ka, const vec3& kd, const vec3& ks, float specularPower)
	: center(c), radius(r) {
	this->ka = ka;
	this->kd = kd;
	this->ks = ks;
	this->specularPower = specularPower;
}
```
In Sphere and Plane class, overridden virtual funtion getNormal.

Read Ka, Kd, Ks and specularPower value

-------------
In Scene.trace funtion, there is 'if' funtion. This code calculates the hit point and then computes the normal vector at that point to obtain the result of Phong shading.
```
if (hit_surface) {
	vec3 hitPoint = ray.origin + closest_t * ray.direction;
	vec3 normal = hit_surface->getNormal(hitPoint);
	return phongShading(ray, hitPoint, normal, *hit_surface);
}
```

-------------
### Important Idea. PhongShading funtion

```
vec3 phongShading(const Ray& ray, const vec3& hitPoint, const vec3& normal, const Surface& surface) const {
	vec3 ambient = surface.ka;
	vec3 lightDir = normalize(lightPos - hitPoint);
	vec3 viewDir = normalize(ray.origin - hitPoint);
	vec3 halfDir = normalize(lightDir + viewDir);

	// Shadow check
	Ray shadowRay;
	shadowRay.origin = hitPoint + normal * 1e-4f;
	shadowRay.direction = lightDir;
	bool inShadow = false;
	for (const auto& s : surfaces) {
		float t;
		if (s->intersect(shadowRay, 0.0f, length(lightPos - hitPoint), t)) {
			inShadow = true;
			break;
		}
	}

	vec3 diffuse(0.0f);
	vec3 specular(0.0f);
	if (!inShadow) {
		diffuse = surface.kd * std::max(dot(normal, lightDir), 0.0f);
		specular = surface.ks * pow(std::max(dot(normal, halfDir), 0.0f), surface.specularPower);
	}

	return ambient + diffuse + specular;
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

---------------------------
The code can be implemented according to the conditions shown in the image as follows.

![image](https://github.com/user-attachments/assets/10b0910a-7215-4ee4-9835-1206343c09c7)

```
scene.surfaces.push_back(new Plane(vec3(0.0f, -2.0f, 0.0f), vec3(0.0f, 1.0f, 0.0f), vec3(0.2f, 0.2f, 0.2f), vec3(1.0f, 1.0f, 1.0f), vec3(0.0f, 0.0f, 0.0f), 0.0f)); // Plane P
scene.surfaces.push_back(new Sphere(vec3(-4.0f, 0.0f, -7.0f), 1.0f, vec3(0.2f, 0.0f, 0.0f), vec3(1.0f, 0.0f, 0.0f), vec3(0.0f, 0.0f, 0.0f), 0.0f)); // Sphere S1
scene.surfaces.push_back(new Sphere(vec3(0.0f, 0.0f, -7.0f), 2.0f, vec3(0.0f, 0.2f, 0.0f), vec3(0.0f, 0.5f, 0.0f), vec3(0.5f, 0.5f, 0.5f), 32.0f)); // Sphere S2
scene.surfaces.push_back(new Sphere(vec3(4.0f, 0.0f, -7.0f), 1.0f, vec3(0.0f, 0.0f, 0.2f), vec3(0.0f, 0.0f, 1.0f), vec3(0.0f, 0.0f, 0.0f), 0.0f)); // Sphere S3
```
--------------
The rest code is same as hw1.
