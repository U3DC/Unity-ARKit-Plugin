# Unity-ARKit-Plugin #


This is a native plugin that enables using all the functionality of the ARKit SDK simply within your Unity projects for
iOS.  The plugin exposes ARKit SDK's world tracking capabilities, rendering the camera video input, plane detection and
update, point cloud extraction, light estimation, and hit testing API to Unity developers for their AR projects. This plugin is a preview quality build that
should help to get you up and running quickly with this technology, but the implementation and APIs may change to cater
to the underlying technology.

The plugin is open sourced and is released under the MIT license (see LICENSE file in this folder)

## Requirements: ##
* Unity v5.6.1p1+
* iOS 11+
* Xcode beta 9 with latest iOS SDK that contains ARKit Framework
* iOS device that supports ARKit (iPhone 6S or later, iPad (2017) or later)


## How to use this code drop:##

The code drop is a Unity project that you can load up in any Unity version that is later than v5.6.1p1.  The Unity
project contains the plugin sources and some example scenes and components that you may use in your own projects.  

Here is a summary of the important files in the plugin:

"/Assets/Plugins/iOS/UnityARKit/NativeInterface/ARSessionNative.mm" - this is the Objective-C code that actually interfaces with ARKit SDK


"/Assets/Plugins/iOS/UnityARKit/NativeInterface/UnityARSessionNativeInterface.cs" - this the scripting API to the ARKit, and provides the glue to the native code

This contains the following APIs:
	    

```
#!C#

	    public void RunWithConfigAndOptions(ARKitWorldTackingSessionConfiguration config, UnityARSessionRunOption runOptions)

	    public void RunWithConfig(ARKitWorldTackingSessionConfiguration config)

	    public void Pause()

	    public List<ARHitTestResult> HitTest(ARPoint point, ARHitTestResultType types)

	    public ARTextureHandles GetARVideoTextureHandles()
Unity-ARKit-Plugin
[Latest Update: This plugin now supports new ARKit functionality exposed in ARKit 2.0. See Whats New In ARKit 2.0 for details.]

This is a native Unity plugin that exposes the functionality of Apple’s ARKit SDK to your Unity projects for compatible iOS devices. Includes ARKit features such as world tracking, pass-through camera rendering, horizontal and vertical plane detection and update, face tracking (requires iPhone X), image anchors, point cloud extraction, light estimation, and hit testing API to Unity developers for their AR projects. This plugin is a preview quality build that will help you get up and running quickly, but the implementation and APIs are subject to change. Nevertheless, it is quite capable of creating a full featured ARKit app, and hundreds of ARKit apps on the AppStore already use this plugin.

Please read LICENSE for licensing information.

The code drop is a Unity project, compatible with Unity 2017.4 and later. It contains the plugin sources, example scenes, and components that you may use in your own projects. See TUTORIAL.txt for detailed setup instructions.

Please feel free to extend the plugin and send pull requests. You may also provide feedback if you would like improvements or want to suggest changes. Happy coding and have fun!

Requirements:
Unity v2017.4+
Apple Xcode 10.0+ with latest iOS SDK that contains ARKit Framework
Apple iOS device that supports ARKit (iPhone 6S or later, iPad (2017) or later)
Apple iOS 12+ installed on device
Building
Give it a go yourself. Open up UnityARKitScene.unity — a scene that demostrates ARKit’s basic functionality — and try building it to iOS. Note that UnityARBuildPostprocessor.cs is an editor script that executes at build time, and does some modifications to the XCode project that is exported by Unity. You could also try building the other example scenes in the subfolders of the Examples folder.

API overview
UnityARSessionNativeInterface.cs implements the following:

public void RunWithConfigAndOptions( ARKitWorldTackingSessionConfiguration config, UnityARSessionRunOption runOptions )
public void RunWithConfig( ARKitWorldTackingSessionConfiguration config )
public void Pause()
public List<ARHitTestResult> HitTest( ARPoint point, ARHitTestResultType types )
public ARTextureHandles GetARVideoTextureHandles()
public float GetARAmbientIntensity()
public int GetARTrackingQuality() 
It also contains events that you can provide these delegates for:

public delegate void ARFrameUpdate( UnityARCamera camera )

public delegate void ARAnchorAdded( ARPlaneAnchor anchorData )
public delegate void ARAnchorUpdated( ARPlaneAnchor anchorData )
public delegate void ARAnchorRemoved( ARPlaneAnchor anchorData )

public delegate void ARUserAnchorAdded(ARUserAnchor anchorData)
public delegate void ARUserAnchorUpdated(ARUserAnchor anchorData)
public delegate void ARUserAnchorRemoved(ARUserAnchor anchorData)

public delegate void ARFaceAnchorAdded(ARFaceAnchor anchorData)
public delegate void ARFaceAnchorUpdated(ARFaceAnchor anchorData)
public delegate void ARFaceAnchorRemoved(ARFaceAnchor anchorData)

public delegate void ARImageAnchorAdded(ARImageAnchor anchorData)
public delegate void ARImageAnchorUpdated(ARImageAnchor anchorData)
public delegate void ARImageAnchorRemoved(ARImageAnchor anchorData)

public delegate void ARSessionFailed( string error )
public delegate void ARSessionCallback();
public delegate void ARSessionTrackingChanged(UnityARCamera camera)
These are the list of events you can subscribe to:

public static event ARFrameUpdate ARFrameUpdatedEvent;

public static event ARAnchorAdded ARAnchorAddedEvent;
public static event ARAnchorUpdated ARAnchorUpdatedEvent;
public static event ARAnchorRemoved ARAnchorRemovedEvent;

public static event ARUserAnchorAdded ARUserAnchorAddedEvent;
public static event ARUserAnchorUpdated ARUserAnchorUpdatedEvent;
public static event ARUserAnchorRemoved ARUserAnchorRemovedEvent;

public static event ARFaceAnchorAdded ARFaceAnchorAddedEvent;
public static event ARFaceAnchorUpdated ARFaceAnchorUpdatedEvent;
public static event ARFaceAnchorRemoved ARFaceAnchorRemovedEvent;

public static event ARImageAnchorAdded ARImageAnchorAddedEvent;
public static event ARImageAnchorUpdated ARImageAnchorUpdatedEvent;
public static event ARImageAnchorRemoved ARImageAnchorRemovedEvent;

public static event ARSessionFailed ARSessionFailedEvent;
public static event ARSessionCallback ARSessionInterruptedEvent;
public static event ARSessionCallback ARSessioninterruptionEndedEvent;
public static event ARSessionTrackingChanged ARSessionTrackingChangedEvent;
ARSessionNative.mm contains Objective-C code for directly interfacing with the ARKit SDK.

All C# files in the NativeInterface folder beginning with “AR” are the scripting API equivalents of data structures exposed by ARKit.

ARKit useful components
Physical camera feed. Place this component on the physcial camera object. It will grab the textures needed for rendering the video, set it on the material needed for blitting to the backbuffer, and set up the command buffer to do the actual blit. UnityARVideo.cs

Virtual camera manager. Place this component on a GameObject in the scene that references the virtual camera that you intend to control via ARKit. It will position and rotate the camera as well as provide the correct projection matrix to it based on updates from ARKit. This component also has the code to initialize an ARKit session. UnityARCameraManager.cs

Plane anchor GameObjects. For each plane anchor detected, this component generates a GameObject which is instantiated from a referenced prefab and positioned, scaled and rotated according to plane detected. As the plane anchor updates and is removed, so is the corresponding GameObject. UnityARGeneratePlane.cs

Point cloud visualizer. This component references a particle system prefab, maximum number of particles and size per particle to be able to visualize the point cloud as particles in space. PointCloudParticleExample.cs

Hit test. This component references the root transform of a GameObject in the scene, and does an ARKit Hit Test against the scene wherever user touches on screen, and when hit successful (against HitTest result types enumerated in the script), moves the referenced GameObject to that hit point. UnityARHitTestExample.cs

Light estimation. This component when added to a light in the scene will scale the intensity of that light to the estimated lighting in the real scene being viewed. UnityARAmbient.cs

You can read how some of these components are used in the Examples scenes by checking out SCENES.txt.

Feature documentation
As newer features have been added on to the plugin, they have been documented in detail elsewhere. Here are links to those documents:

Unity ARKit Remote
“Introducing the Unity ARKit Remote”

“ARKit Remote: Now with face tracking!"

Face tracking
“ARKit Face Tracking on iPhone X.” (Yes—It requires an iPhone X.)

"Create your own animated emojis with Unity!"

Plugin Settings file
"ARKit uses Face Tracking API"

"App requires ARKit device"

ARKit 1.5 Update
Unity blog post "Developing for ARKit 1.5 update using Unity ARKit Plugin"

ARKit 2.0 Update
How to use new features from Unity: Whats New In ARKit 2.0

Questions? Bugs? Showcase?
Contact us via the forums for questions.

You may submit issues if you feel there are bugs that are not solved by asking on the forums.

You may submit a pull request if you believe you have a useful enhancement for this plugin.

Follow @jimmyjamjam for various AR related tweets, and showcase your creation there as well.
	    public float GetARAmbientIntensity()

	    public int GetARTrackingQuality()  
```


  
It also contains events that you can provide these delegates for: 


```
#!C#

        public delegate void ARFrameUpdate(UnityARCamera camera)

    	public delegate void ARAnchorAdded(ARPlaneAnchor anchorData)

        public delegate void ARAnchorUpdated(ARPlaneAnchor anchorData)

        public delegate void ARAnchorRemoved(ARPlaneAnchor anchorData)

        public delegate void ARSessionFailed(string error)
```

 

"/Assets/Plugins/iOS/UnityARKit/NativeInterface/AR*.cs" - these are the scripting API equivalents of data structures exposed by ARKit

"/Assets/Plugins/iOS/UnityARKit/Utility/UnityARAnchorManager.cs" - this is a utility that keeps track of the anchor updates from ARKit and can create corresponding Unity gameobjects for them (see GeneratePlanes.cs component on how to use it)

"/Assets/Plugins/iOS/UnityARKit/Editor/UnityARBuildPostprocessor.cs" - this is an editor script that runs at build time on iOS 

## ARKit useful components: ##

"/Assets/Plugins/iOS/UnityARKit/UnityARCameraManager.cs" - this component should be placed on a gameobject in the scene that references the camera that you want to control via ARKit, and it will position and rotate the camera as well as provide the correct projection matrix to it based on updates from ARKit.  This component also has the code to initialize an ARKit session.

"/Assets/Plugins/iOS/UnityARKit/UnityARVideo.cs" - this component should be placed on the camera and grabs the textures needed for rendering the video, and sets it on the material needed for blitting to the backbuffer, and sets up the command buffer to do the actual blit

You should be able to build the UnityARKitScene.unity to iOS to get a taste of what ARKit is capable of.  It demostrates all the basic functionality of the ARKit in this scene.  

See TUTORIAL.txt in this project for more detailed steps on setting up a project step by step.

Please feel free to extend the plugin and send pull requests. You may also provide feedback if you would like improvements or want to suggest changes.  Happy coding and have fun!


## Recent additions ##

Face tracking API:
ARKit face tracking is possible on iPhone X and later (blog)
Settings file that has options for plugin usage:
ARKit uses Face Tracking API (link to usage)
App requires ARKit device (link to usage)
Summary
See TUTORIAL.txt in this project for more detailed steps on setting up a project step by step.
Please feel free to extend the plugin and send pull requests. You may also provide feedback if you would like improvements or want to suggest changes. Happy coding and have fun!
