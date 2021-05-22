---
layout: post
title: Shadow Mapping
subtitle: How shadows work in Metro Engine
tags: [shadow, texture, lighting, engine]
---

## Shadow Mapping

  We all know that this technique revolves around **shadows** just because of the name, and shadows happen whenever there is absence of light due to some kind of **occlusion**, this means that whenever a light source does not hit an object because it is occluded behind a wall or some other object, the object is in shadow.
  
  The idea is rather simple, we basically **render the scene from the light's perspective**, because we know that everything that is **seen** by the light is **lit**, this means that everything that **does** not appear from the light's perspective needs to be in **shadow**.
  
  ![Light Shadow Mapping Example #1](https://user-images.githubusercontent.com/48097484/119214127-a173a880-bac4-11eb-81fa-b2d16c27d296.png)

  From this basic principle and the image shown above we can see that the light is illuminating the blue surfaces of the red squares and the black color represents what has to be in shadow. This whenever applied to computer graphics you logically might think that we need to imitate real life **light sources**, this quickly becomes unfeasible because if we cast **rays** from our **light sources** to everywhere in the **scene** and start comparing the closest point to the other points in the **ray**, it impies that we need to iterate through roughly **thousands or millions** of light rays per **light source**, you can quickly realize that this becomes unfeasible whenever we try to apply it to computer graphics.
  
  The goal is to have an algorithm that can be ran at **runtime**, implying that we need to seek for a more optimized and better approach to shadow mapping, and luckily there is a few techniques that will allow us to create shadows and be able to execute them in **runtime** with little to no cost at all!

  To be able to achieve this technique we need to utilize something called the **depth buffer**, it is a buffer that is filled with information of the depth of a fragment in a scene clamped to `[0, 1]`, this everything is done from the **camera's point of view**.
  
  What if I told you that we could pretty much do the same with a depth buffer but instead of rendering the scene from camera we render it from each one of the **light sources** and store that information in a texture. This way we are storing all the information about depth from each perspective of the light, implying that we now can **sample** the **closest depth** and see it from the light's perspective, the values that are the furthest are not stored. Instead of calling it a depth map now, those textures can be called "**shadow maps**".
  
  ![Projected Shadow Directional Light](https://user-images.githubusercontent.com/48097484/119215217-81e07e00-bacc-11eb-9cd1-42932bc898ca.png)

  In the image above we see a **directional light** being casted in the scene on a red square, we can see that all the values that are being stored in the **shadow map** are the yellow part, and the black part represents the projected shadow, take care that this is from the **light's perspective**.
  
  How do we determine if it is a valid value or a shadow? Using the depth values that are stored in the texture you find the **closest** point and use that to determine whether the fragment is in shadow, we render the scene from the **light's perspective** using a **view and projection matrix** specific to that light source, essentially transforming any _3D coordinates_ into **coordinate space**.
  
  In the image we can see that we have two points, `P` and `C`, P being the furthest one and C being the closest one, to determine which one is in shadow and which one is being lit by the light, we need to do a simple comparison between them, the one that is smaller is the one that is being lit, the other one is immediately identified as a shadow.
  
  Of course we need to transform the point `P` to **coordinate space** to be able to compare with point `C` and we can use point `P` to index the shadow map to obtain the closest visible depth from the light's perspective, otherwise known as `C`.
  
  Essentiallyw hat we are doing is doing two passes, we render the depth map and in the second map we render the scene as usual but using the previously generated depth map in the previous pass to calculate whether fragments are in shadow or not.
  
## Implementation
  
  With that explained, we will start explaining how is the actual **implementation** of the actual technique in our **engine**. As previously mentioned, we need to render to a texture from every light's perspective, this implies that we need to create a render to texture for each **light type**, so if we have `50` directional lights, we need to create 50 **render to textures**.
  
```cpp
  RenderToTexture shadowMapRender_[kMaxDirectionalLights];
```
  In addition to creating the renders to texture, we need to do a depth attachment with empty textures to each one of those render to textures, and the following code needs to be executed in the `Initialize()` function in our engine:
  
```cpp
	for (u32 i = 0; i < kMaxDirectionalLights; i++)
	{
		char name[20];
		sprintf(name, "ShadowMapTexture%d", i);
		u32 shadowMapDepthTex = tb.createEmptyTexture(glm::vec2(kScreenWidth, kScreenHeight), 4, Texture::kInternalType_Float, name);
		gpum->shadowMapDepthRender_[i].AddDepthAttachment(shadowMapDepthTex, false);
	}

```
  
  Having the textures ready, we now need to do two passes as previously explained, in the first pass we fill the textures with the depth map information (Z buffer) hence in our engine we utilize a specific shader that **fills** the RenderToTexture's depth attachments from the light's perspective, we need to remember that we are utilizing the structure of our engine with DisplayLists, I recommend reading [this post](https://metro-engine.github.io/2021-05-13-Metro_Engine_Structure/) in case you have still not learnt about the structure of the engine.
  
```cpp

RenderSystem* rs = coordinator_.GetSystem<RenderSystem>();
  std::vector<LightHandler> shadowGeneratingLights = sceneDirectionalLights_;
  std::vector<glm::mat4> lightViewMatrices;
  std::vector<glm::mat4> lightProjectionMatrices;

  //-------------------------------------------------Shadow mapping pass
  ScopedPtr<DisplayList> shadowPassDL;
  shadowPassDL.Alloc();
  //Use shadow pass material
  ScopedPtr<RenderCommand> useMatRC;
  UseMaterialRC* matRCitself = useMatRC.AllocT<UseMaterialRC>();
  matRCitself->SetMaterialID(GetMaterial(ResourceManager::kMaterialType_OnlyDepth)->getMaterialID());
  shadowPassDL->Add(std::move(useMatRC));

  SubmitList(std::move(shadowPassDL));

  rs->shadowMapPass_ = true;
  ```
  
  We first use our render system to define when we are doing a **shadow map pass** and we define it with the `shadowMapPass_` bool. For instance we are utilizing `DisplayLists` with a specific material type which in this case is `OnlyDepth` which inherently passes the `Z buffer` information to the texture we have attached to the depth attachment in our render to texture, afterwards we submit the display list to the **main deque** and we activate the shadow map pass, this was the pre-emptive preparation for shadow mapping alongside the creation of the vectors for the directional lights and their respective **view and projection matrixes**.
  
  The `Fragment Shader` of the `OnlyDepth` material is the following:
  
```glsl
  
  #version 330

  layout (location = 0) out float gDepth;

  in vec4 position;

  void main()
  {
    gDepth = position.z;
  }

```
  
  In fact we are completely ready to jump straight to the for loop that will be utilizing the `RenderToTextureRC` to fill the textures and for us to setup the matrixes and store them in the previously seen vectors so we can later send them to the shader.
  
```cpp
rs->shadowMapPass_ = true;
  if (shadowGeneratingLights.size() > 0)
  {
    for (u32 i = 0; i < shadowGeneratingLights.size(); i++)
    {
      ScopedPtr<DisplayList> innerShadowPassDL;
      innerShadowPassDL.Alloc();

      ScopedPtr<RenderCommand> renderToTextureRC;
      RenderToTargetRC* rendToTextureRCItself = renderToTextureRC.AllocT<RenderToTargetRC>();
      
      // # We set the target to the specific RenderToTexture.
      rendToTextureRCItself->SetTarget(&shadowMapDepthRender_[i]);
      innerShadowPassDL->Add(std::move(renderToTextureRC));

      ScopedPtr<RenderCommand> clearScreenRC;
      
      // # We clean the screen so whenever the next RenderToTexture is being filled it works in a clean canvas for Z Buffering:
      clearScreenRC.AllocT<ClearScreenRC>()->SetDepthClearing(true);
      innerShadowPassDL->Add(std::move(clearScreenRC));

      SubmitList(std::move(innerShadowPassDL));

      // # We construct the matrixes utilizing the `look at` matrix and the `orthographic` matrix and we execute
      glm::vec3 fwd = glm::vec3(shadowGeneratingLights[i].forward_.x,
        shadowGeneratingLights[i].forward_.y,
        shadowGeneratingLights[i].forward_.z);
      glm::vec3 up = glm::vec3(shadowGeneratingLights[i].up_.x,
        shadowGeneratingLights[i].up_.y,
        shadowGeneratingLights[i].up_.z);
      glm::vec3 pos = -fwd * 200.0f;
      rs->vm_ = glm::lookAt(pos, pos + fwd, up);
      rs->pm_ = glm::ortho<float>(-20, 20, -20, 20, 0.01f, 300);
      lightViewMatrices.push_back(rs->vm_);
      lightProjectionMatrices.push_back(rs->pm_);
      
      // # We execute the rendering system (we set the light matrixes inside in uniform blocks):
      rs->Execute();
    } 
  }

```

  A more in depth detail on what is interesting about our execution of our rendering system is that we bind the shader in which we will be utilizing the shadow map textures and light matrixes (_view and projection_) and we bind them to uniform blocks so it is easier to organize whenever in the shader, here is the example code:
  
```cpp
	if (shadowMapPass_) {
        ScopedPtr<RenderCommand> vpRC;
        UniformViewProjectionRC* matRCitself = vpRC.AllocT<UniformViewProjectionRC>();
        matRCitself->SetViewProjection(vm_, pm_, "u_light_v_matrix", "u_light_p_matrix");
        drawObjDisplayList->Add(std::move(vpRC));
    }

```
  
  This would be the internal work that we would be basically doing in our **engine** as far as **CPU** goes, now the shader 
