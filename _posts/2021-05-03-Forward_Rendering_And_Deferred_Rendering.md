---
layout: post
title: Metro Engine Rendering Technique
subtitle: Forward & Deferred Rendering
tags: [rendering, forwardrendering, deferredrendering, opengl]
---

## Rendering Technique

  The initial stages of how we rendered everything in Metro Engine was through forward rendering, which meant that for each **fragment**(_pixel_) we calculated lighting from all the light sources in the scene.
  
  This is a pretty straightforward rendering technique that is used in a lot of engines that do not require a lot of heavy lighting and the amount of lightings are controlled and limited.
  
  In contraposition it is very bad because each fragment gets overwritten by the next lighting and all the lighting calculations that were previously done are completely wasted, this means that a lof of computation is just wasted because information is being constantly overwritten in the fragment.
 
 {: .box-warning}
**Warning:** The specific cost for forward rendering is a **constant** cost of `num_fragments * num_lights`

  The pipeline in forward rendering is pretty linear, you need to feed to the GPU the geometry, project it with the **view_projection matrix** and transform it giving it a position, rotation and scale so it can later be split into fragments and the lighting calculation can be done per fragment.

![Linear Example of Forward Rendering](https://i.imgur.com/qztQhS2.png)

  Imagine that if you have 5 objects in a scene and 5 lights you will need to render every objects 5 times to see the image with the objects being affected by lighting from 5 different light sources, this means that it needs to do 25 operations per frame and only the uttermost calculation is the one that will be staying in the screen because it is constantly being **overwritten**. 
  
  Now imagine that instead of 5 lights you had 50, the number goes up to 250 operations per frame, this proves that this rendering technique despite being most used because of its simplicity, it is inefficient in the case of having a scene with a high number of light sources.

  This means that if your engine is going to be utilizing a lot of lighting which is done in most of the games, you will most probably be better utilizing other rendering techniques such as **Deferred Rendering**.



## Deferred Rendering

  As previously mentioned, deferred rendering is the perfect case for whenever you have a software in which you render lighting such as an engine and you do want to have multiple light sources without having any operations wasted.
  
  This of course comes with a drastic change in how we render objects, given that this technique allows us to render **hundreds** or even **thousands** of lights with a rather acceptable framerate without any kind of trottle.
  
  The idea of this rendering technique is to postpone the lighting calculations to a later stage, this means that we need to do passes, first we need to do what is called the **geometry pass** in which we render the scene and retrieve all the geometry visible in the clipping space of the camera, this means that we will be retrieving all the geometrical information and storing them in a collection of textures called the **G-Buffer**.

  This essentially means that we will be sort of doing a snapshot of specific parts of the scene and store them in a canvas, in one single frame, we will be storing values such as:
  
  - Position Vectors.
  - Color Vectors.
  - Normal Vectors.
  - Specular Values (Optionally).

  Once again this implies that we will be constantly doing snapshots and storing the information in a canvas and we will have this information separated in different canvas, think of having like 4 different sheets of paper size A4 in which you draw stuff, in this case whenever you do the geometry pass, in the first sheet of paper you store the positions, in the next one the normals, the other one the albedo calculations and the last one the specular values. 
  
   I know this might be a dumb way to explain it but this is a way to store information separatedly in different places and that allows us as the developers to utilize that information whenever needed for our own purposes like doing a postprocess that requires the normals of our geometries or a postprocess that requires the position of our geometries. It is a very useful way to separate the information and to not be overwriting data anywhere, you centralize and localize it in sheets of paper (_textures_) and use them for your specific purposes.
   
   With that said we know now that we have a G-Buffer (_which has all those sheets filled with different information_) and we use them for what we know as a **second pass** which in this case is the **lighting pass** where we generate a plane/quad and calculate the scene's lighting for each fragment using all the geometrical information that we have stored in the **G-Buffer**.
   
   ![Position,Color,Normals](https://i.imgur.com/eV4SpI8.png)
   
   `(Positions: Up-Left, Normals: Up-Right, Color: Middle)`
   
  For example in our engine we have the following textures (_sheets of paper_) filled with information about positions, color(_albedo_) and normals. This allows us to do a second pass in which we use the g-buffer textures as outputs and we calculate the lighting per fragment. 
  
  ![Deferred Geometry And Lighting Pass Process](https://i.imgur.com/HmbOUP3.png)
  
  The advantage of deferred is that whatever fragment ends up in the G-Buffer is the actual information that ends up shown in the screen pixel, a depth test is already done and it is concluded which fragment to be the top-most one, implying that there is no data overwriting and no wasted calculations, everything is done precisely and to the point.
  
  This means that the cost of the light calculatios is not `num_fragments * num_lights` anymore because this approach ensures that for each pixel processed in the lighting pass, the lighting is only calculated **once**.

  This rendering technique is not perfect, it comes with disadvantages and one of them is that the G-buffer requires a lot of scene data to be stored, this means that this technique is heavy memory wise, but it is all at the expense of being able to have many light sources without a huge reduction in framerate.

## Metro Engine Approach to Deferred

  In our engine we utilize this technique alongside the OpenGL graphic backend but we reassure you that this is pretty much the same process through any other graphic backend such as in PS4.
  
  To be able to create the **G-Buffer** you initially need a **framebuffer**, this is essentially just a chunk of memory in which you can store information, given that you are doing the **geometry pass** for example, you will be storing the position, normal, albedo texture information there.
  
  In this case with the OpenGL graphic backend you can create a framebuffer and this allows us to have multiple render targets to which we can render, think of it as a creation of a "_new screen_" for every **framebuffer** you create.
  
  This means that in Metro Engine we have a class called "**RenderToTexture**" that allows us to do color attachments and depth attachments (by attachments we imply textures that will be used inside that "_new screen_" or as technically speaking, framebuffer.
  
```cpp

namespace Metro {

  class RenderToTexture {

  public:
    RenderToTexture() {
      // ...
    }
    ~RenderToTexture() {
      // ...
    }

    inline u32 GetID() const {
      // ...
    } 
    inline void SetID(u32 newID) {
      // ...
    }

    inline void AddAttachment(u32 tex, u32 depthTex = -1, bool withStencil = false) {
        // ...
    }

    inline void AddDepthAttachment(u32 depthTex, bool withStencil) {
      // ...
    }

    inline u32 GetInternalColorAttachmentSize() {
        // ...
    }

    inline u32 GetTextureIDFromIndex(u32 index) {
        // ...
    }

    inline u32 GetAttachmentsUsed() {
        // ...
    }

    inline void SetAttachmentsUsed(u32 value) {
        // ...
    }

    inline u32 GetCurrentDepthTexture() {
        // ...
    }

    inline void SetLastFrameColorBuffer(std::vector<u32> currentColorAttachmentBuffer) {
        // ...
    }

    inline void SetLastFrameDepthBuffer(u32 currentDepthAttachment) {
        // ...
    }

    inline std::vector<u32> GetInternalColorAttachmentBuffer() {
        // ...
    }

    inline std::vector<u32> GetLastFrameColorAttachmentBuffer() {
        // ...
    }

    inline u32 GetLastFrameDepthTexture() {
        // ...
    }

    inline void FramebufferSnapshot() {
        // ...
    }

    inline bool HasStencil() {
      // ...
    }

  private:
    u32 internalID_;
    std::vector<u32> internalColorAttachmentsBuffer_;
    u32 internalDepthAttachment_;
    u32 numberAttachmentsUsed_;

    std::vector<u32> lastFrameColorAttachmentBuffer_;
    u32 lastFrameDepthAttachment_;

    bool withStencil_;

  };


```

  This means that the sole purpose of this class is to store the textures which you attach to the framebuffer, another **analogy** that we can do for a "framebuffer" is the class name itself, **Render To Texture**, implying that the _new screen_ in this case is a texture, a white canvas in which you can draw freely given information, that is for waht the attachments are.
  
  This class on itself is not that useful given the current structure of our engine, we do utilize **RenderCommands** and **DisplayList** as previously mentioned in the [general structure of our engine](https://metro-engine.github.io/2021-05-13-Metro_Engine_Structure/) post, this means that we need to follow this pipeline for us to be able to render to screen successfully without any trouble.
  
  So we have a render command for the render to texture and the render to screen and those are the following:
  
  ```cpp
  class RenderToTargetRC : public RenderCommand {

  public:

    ~RenderToTargetRC() override {
    }

    void Action() override;

    void SetTarget(RenderToTexture* targetRenderTexture);

  private:

    u32 GenerateColorTexture(u32 texPassed);
    u32 GenerateDepthTexture(u32 depthTextPassed);

    RenderToTexture* fillRenderToTextureObject_;
  };
  
```
```cpp

  class RenderToScreenRC : public RenderCommand {

  public:
    void Action() override;
  };
  
````

  This **RenderCommands** internally create **framebuffers** and alongside the actual textures that we pass to the RenderToTexture we do all the pertinent mixmatching and drawing of textures. For example to do the geometry draw of deferred rendering in our engine we do the following:
  
 ```cpp
 //----------------------------------------Geometry Pass
  ScopedPtr<DisplayList> geometryPassDL;
  geometryPassDL.Alloc();

  ScopedPtr<RenderCommand> renderToTextureRC2;
  RenderToTargetRC* rendToTextureRCItself2 = renderToTextureRC2.AllocT<RenderToTargetRC>();
  rendToTextureRCItself2->SetTarget(&geometryPassRender_);
  geometryPassDL->Add(std::move(renderToTextureRC2));

  ScopedPtr<RenderCommand> clearScreenRC2;
  ClearScreenRC *clearScreenRCItself = clearScreenRC2.AllocT<ClearScreenRC>();
  clearScreenRCItself->SetClearColor(glm::vec4(0.5f, 0.5f, 1.0f, 1.0f));
  clearScreenRCItself->SetDepthClearing(true);
  clearScreenRCItself->SetStencilClearing(true);
  geometryPassDL->Add(std::move(clearScreenRC2));

  SubmitList(std::move(geometryPassDL));

```
  
  {: .box-note}
**Note:** You might see that in this code we have something called `ScopedPtr` and this is explained in a resource management post in the blog which you can find here.

  You can see now that we first create the display list, we create the rendercommand and pass it the pertinent parameters and we later add it to the display list to finally at the end submit the list to the **main deque** that we have in our engine. Of course you can see that in the middle of this process we throw a clean screen render command in the middle so we can do another pass in the future with clean information.
  
## Shader Specifics
  
  Now you might be wondering how do we capture this output through the shader and utilize it, so far we have only seen **CPU** code and we have not seen a single bit of **GLSL**, it is actually pretty simple, OpenGL after having everything binded correctly, you can just capture those based on their attachment number and use them in the shader with no problem. (_You need to remember that you are binding with textures_), this implies that for deferred to work efficiently you need to have **UVS** that you can safely pass to the shader through _attribute binding_ from the **VS shader** to the **FS shader**.
 
```glsl

// Fragment Shader / Pixel Shader Example:

// Imagine that you are binding for example the positions, normals and albedo textures to the shader following the color attachment order of 0, 1, 2

layout (location = 0) out vec4 gPosition;
layout (location = 1) out vec4 gNormal;
layout (location = 2) out vec4 gRawColor;

uniform sampler2D u_texture_normals;
uniform sampler2D u_texture_albedo;

// UVS come from attribute binding from the VS Shader.
in vec2 uvs;

// World Space Position & WorldNormal comes calculated from VS Shader
in vec4 worldSpacePosition;
in vec4 worldSpaceNormal;

void main() 
{

  vec3 albedoColor = texture(u_texture_albedo, uvs).rgb;
  
  gPosition = vec4(worldSpacePosition.xyz, 1.0);
  gRawColor = vec4(albedoColor, 0.0);
  gNormal = vec4(worldSpaceNormal.xyz, 1.0);

}


```

  {: .box-warning}
**Warning:** It is very important to bring all the information in the same **coordinate space** whenever doing lighting calculations for example, else you will get unexpected and/or undesired results.

  In this case we utilize **world-space** coordinate system and we transfer the data to the individual g-buffers which will later be utilized in the **lighting pass**. This is how we essentially capture the information through the geometry pass so it can later be used in the lighting pass in the same old fashioned way, you get the buffer, unwrap it with the uvs and use it in the shape of a `vec3` or `vec4` depending if you are going to store extra information in the 4th coordinate of each buffer.
  
  In the example we did **not** choose to store extra information but as you can see they have an extra coordinate in which you could store for example the **roughness** or **specular** value in case you were doing for example **PBR**(_Physical Based Rendering_).
  
  Now the **lighting pass** is as easy as binding the desired g-buffer textures to the lighting pass shader through uniforms and then use that information to do all pertinent lighting calculations.
  

```glsl

// Fragment Shader / Pixel Shader Example:

// Imagine that you are binding for example the positions, normals and albedo textures to the shader following the color attachment order of 0, 1, 2

layout (location = 0) out vec4 gFinalResult;

uniform sampler2D u_color_texture;
uniform sampler2D u_position_texture;
uniform sampler2D u_normal_texture;

// UVS come from attribute binding from the VS Shader.
in vec2 uvs;

// World Space Position & WorldNormal comes calculated from VS Shader
in vec4 worldSpacePosition;
in vec4 worldSpaceNormal;

void main() 
{

  vec4 finalColor = vec4(0.0, 0.0, 0.0, 1.0);
	vec4 colorTexValue = texture(u_color_texture, uvs);

	vec4 worldSpacePosition = texture(u_position_texture, uvs);
	worldSpacePosition.a = 1.0;
	vec4 normals = texture(u_normal_texture, uvs);
	normals.a = 0.0;

	int numOfDirectionalLights = int(numOfLights.x);
	int numOfSpotLights = int(numOfLights.y);
	int numOfPointLights = int(numOfLights.z);

	vec4 lightSpacePositions[kMaxDirectionalLights];

	for(int i = 0; i < numOfDirectionalLights; ++i){
		lightSpacePositions[i] = u_light_p_matrix[i] * u_light_v_matrix[i] * worldSpacePosition;
		finalColor += CalculateDirectionalLights(directionalLights[i], normals, lightSpacePositions[i], colorTexValue, i);
	}

	for(int i = 0; i < numOfPointLights; ++i){
		finalColor += CalculatePointLight(pointLights[i], worldSpacePosition, normals);
	}

	for(int i = 0; i < numOfSpotLights; ++i){
		finalColor += CalculateSpotLight(pointLights[i], worldSpacePosition, normals);
	}


	gFinalResult = finalColor;

}


```
  
  As you can see, in the light pass for normal lighting, you use the textures that you bind and then uv unwrap them with the `texture(texture, uvs)` function and then do the pertinent calculations for the different types of lightings that your engine might use.
  
  At the end you might have noticed that we once again generate a g-buffer texture which will be filled with the final color texture that will be later drawn to screen with the **RenderToScreen** RenderCommand.
  
```cpp

	ScopedPtr<RenderCommand> renderToScreenRC;
	renderToScreenRC.AllocT<RenderToScreenRC>();
	swapList->Add(std::move(renderToScreenRC));

```
  And we shall not forgive that this process is nothing if we do not draw a full screen quad from 0,0 to 1,1 (width, height) and draw a texture with the final lighting results on it.
  
```cpp

  ScopedPtr<RenderCommand> drawGeoRC;
  DrawGeometryRC* drawGeoRCitself = drawGeoRC.AllocT<DrawGeometryRC>();
  drawGeoRCitself->SetGeometry(resourceManager_->GetGeometry(ResourceManager::kGeometryType_Quad));
  drawGeoRCitself->screenSpaceGeometry_ = true;
  lightPassDL->Add(std::move(drawGeoRC));
  
  ```

  In this case we use a **DrawGeometryRC** that allows us to draw any kind of geometry to the screen, it is done through a render command once again to follow the pipeline of our engine so everything is properly synchronized and with no mishaps. 
