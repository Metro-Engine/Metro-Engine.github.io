---
layout: post
title: Lighting in Metro Engine
tags: [math, opengl, spotlight, directional-light, pointlight, engine, lighting]
---

## Introduction

  In every engine we need to have some sort of way to illuminate a scene or lit a specifiv object, that is the fundamental basis of a game engine, being able to lit objects in a scene to make them appear realistic. That is why in an engine we need to have **different** ways to cast light in our scene, so we need to mimic **real life** and think about types of lights that we can find in real life.
  
  The most obvious one is in the sky, the sun itself, we can think of it as a **global** illumination light, an omniponent directional light that is going to lit the whole scene in a specific direction. 
  
  Afterwards we have the spotlight , it is commonly used to shine a specific spot in a concert for example, it is used to capture your attention on a specific place in the scene.
  
  Lastly we have the pointlight is is a light that is omnidirectional 360 degrees, is is pretty much the standardized light that is used in engines, a lamp in itself is the most iconic way to see a spotlight represented in a game scene.
  
  Given that we are utilizing the **OpenGL** graphic backend in our PC version, we will be explaining how to implement all three types of lights in the engine to have various way to lit a scene.
  
### Directional Light

As previously stated this type of light can act as a "global light" where all the light rays are parallely lighting a surface, it is apparent that if the light rays are parallel to each other it creates the effect that they all come from the same direction, we need to consider that the position of the directional light is of no relevance, it is not going to change the intensity with which it is going to be lighting up a surface, even though in our engine the directional light has an actual position parameter, for the lighting we assume that the light rays are infinitely far away, just like the sun in some kind of way.

![Directional Light Example](https://user-images.githubusercontent.com/48097484/119268323-b3536980-bbf2-11eb-87ed-392f9d86ba58.png)

 In our engine we are currently using **Deferred Rendering** for the overall rendering pipeline, we recommend you to have a read [here](https://metro-engine.github.io/2021-05-03-Forward_Rendering_And_Deferred_Rendering/) in case you have not read about it to better understand about the different passes that need to be done in this specific pipeline to get the light working.
 
 To be able to have the geometry to be lit we first need to do a geometry pass, this later will get redirected to the **light pass** and in the **fragment shader** is where we will construct all the different types of lightings because we will have all the necessary information for us to create the final fragment information.
 
 So for a directional light we do not need to have a lot of information stored, in this case we created a struct inside the shader and stored the following information:
 
```glsl
struct DirectionalLight {
  vec4 position;
  vec4 direction;
  vec4 color;

};
 
```
 
 And the function that will be calculating the directional light is the following one:
 
 ```glsl
 vec4 CalculateDirectionalLights(DirectionalLight dl, vec4 currFragNormal, vec4 lightSpacePosition, vec4 colorTexValue, int index){

	vec4 lightDirection = dl.direction;
	vec4 lightColor = dl.color;
	float lightIntensity = max(dot(currFragNormal, -lightDirection), 0.0);

	//Ambient
	float ambientForce = 0.1;
	vec4 ambientColor = ambientForce * lightColor;

	//Diffuse
	vec4 diffuseColor = lightIntensity * lightColor;
	vec4 objectColor = colorTexValue;

	return (diffuseColor + ambientColor) * objectColor;
}
 
 ```
 
  In our case for example our directional light has a diffuse coloring, ambient coloring and we blend it through with the object's color. You can notice that we are currently using the `negative` direction of the light to go **from the light source** towards the **fragment position**.
  
## Point Light

  The following type of light is very different to the previous one, it is not a **global** illumination light that is going to illuminate the whole scene, this type of light is more localized its purpose is to light up specific spots in a scene and this kind of light shines best whenever it is spread and scattered around a scene. In this specific case, the position is a very important factor that is taken into consideration whenever calculating the fragment information for this light.
 
 ![Point Light Example](https://user-images.githubusercontent.com/48097484/119269794-c87fc680-bbf9-11eb-8b75-e5865a73baf9.png)

  This means that for this light we need to take into consideration other factors because as it is an **omnidirectional** light we need to define things such as the distance to which they can lit objects, the attenuation over time depending on the distance, because light attenuation happens the more a light ray travels further from its source. Lastly we can also tweak the intensity with which the light source is emitting that light, making this light source a very modular and highly customizable light for the engine.
  
  In fact to be able to do a correct **attenuation** we could just do a linear decrease the further the light ray travels, but the goal here is to achieve **realism** so we need to go tightly with _mathematics_ to achieve a more logarithmic approach **0 distance** being the highest intensity `1` and for example **100 distance** being intensity approximated to `0`. 

  Some mathematicians already have figured out such formula so we we will be explaining it so you guys can understand it without any trouble:
  
  ![Logarithmic Attenuation Point Light](https://user-images.githubusercontent.com/48097484/119270084-4d1f1480-bbfb-11eb-9aee-4f47e82bda77.png)

  We will start with something easy, the letter `d` here represents the distance from the **fragment**(_pixel_) to the **light source**, in this formula we need to calculate 3 values to be able to achieve this **attenuation** effect, in this case we have `Kc` being a constant value, `Kl` being a linear value and lastly `Kq` being a quadratic variable.

- The `Kc` constant variable is there to usually keep in check that the denominator of the formula never gets smaller than `1` since this would modify the intensity of the light source over certain distances which could cause erratic behavior.
- The `Kl` linear value is multiplied with the distance value to reduce the intensity in a linear fashion from `1` to approximately `0` the more the distance grows.
- The `Kq` quadratic variable is used to determine a quadratic decrease of the intensity for the light source. An example of how quadratic equations work [here](https://www.mathsisfun.com/algebra/quadratic-equation.html).

  Due to the quadratic variable the light will start losing intensity in a linear fashion until a certain threshold of distance is passed, then it will start decreasing a lot faster until it almost reaches `0`, implying that thanks to the quadratic constant we will have a more progressive loss of light intensity towards the distance.
  
  ![Light Intensity Loss Example ](https://user-images.githubusercontent.com/48097484/119270629-e6e7c100-bbfd-11eb-85d0-9d14e7cadb21.png)

  The implementation for this point light is going to be the following, as usual the struct will contain variables for `constant`, `linear` and `quadratic` that will be plugged in the formula above to be able to get the **attenuation** effect. Other than that we might have a position, we notice that we **do not** have a **direction** because as you remember, point lights are **omnidirectional**. Aside from that we can have a diffuse and a specular(_optional_) if we need.
  
```glsl

struct PointLight {
	vec4 position;
	vec4 direction;
	vec4 color;

    float radius_;
    float constant_;
    float linear_;
    float quadratic_;

};

vec4 CalculatePointLights(PointLight light, vec4 worldSpacePos, vec4 colorTexValue)
{
	vec4 lightColor = light.color;
    float distanceF    = length(light.position - worldSpacePos);
    float attenuation = 1.0 / (light.constant_ + light.linear_ * distanceF + 
  			     light.quadratic_ * (distanceF * distanceF));   
				 
    //Ambient
	float ambientForce = 0.6;
	vec4 ambientColor = ambientForce * lightColor;

	//Diffuse
	vec4 diffuseColor = lightColor;
	vec4 objectColor = colorTexValue;
	diffuseColor *= objectColor * attenuation;

  return ambientColor + diffuseColor;
} 

```

  As we can see the attenuation is calculated as expected from the formula shown above, of course the information about the attenuation variables are captured per each light and are set by the user through the CPU or through ImGui.
  
  
## Spotlight

And the last type of light that was added to the engine was the **spotlight**, we know that a spotlight in its essence is a light that shoots its rays in a specific direction, it is the limited version of what a point light is, the direction of the rays is not **omnidirectional** anymore but are constrained in a **circular** limit.

As it is a **circular** shape, we of course can define an angle with which the aperture(_radius_) of the spotlight better known as the `cut-off angle` which defines which objects are going to be lit, the lighting direction of the spotlight and then we also have another internal angle called `theta` that defines the angle between the light direction and the spot direction which is the _perpendicular_ vector of the spotlight.

![SpotLight Calculation Example](https://user-images.githubusercontent.com/48097484/119273690-519ff900-bc0c-11eb-9e3d-1ae3821ec8d8.png)


In essence the spotlight calculations are simple enough, we need to calculate the dot product (_which in essence returns the cosine of the angle between two vectors_) in this case the `LightDir` and the `SpotDir` so we can get the angle `Theta` so we can compare it with the given `cut-off` angle to see which fragments are going to be calculated and which not, this implies that we will have to do some kind of if statement in the **Fragment Shader** as following:


```glsl
struct SpotLight {
	vec4 position;
	vec4 direction;
	vec4 color;

    vec4 radius_;
};

vec4 CalculateSpotLights(SpotLight light, vec4 worldPosition, vec4 colorTexValue) {

    float theta     = dot(light.position.xyz - worldPosition.xyz, normalize(-light.direction.xyz));
    float intensity = 0.0;   


    if(theta > light.radius_.x) 
    {       
      intensity = 1.0;
    }

    vec4 ambient = colorTexValue;
    vec4 diffuseColor = light.color;
    return ambient + diffuseColor * intensity;
    
}


```

As we can see, we do not do any kind of _smoothing_ and we just set the intensity to 1.0 if theta is greater than the given radius(_angle cut-off_), other than that the calculus for a spotlight is pretty straightforward.
	
	

## Conclusion

All the references to the different types of lights and mathematics were extracted from [here](https://learnopengl.com/Lighting/Light-casters) in case you want to see a more in depth explanation on it.






