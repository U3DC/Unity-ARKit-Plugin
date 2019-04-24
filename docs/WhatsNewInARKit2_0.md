
# Whats new in Unity ARKit Plugin for ARKit 2.0 


## ARWorldMap  

This allows you to keep track of places you have been during your session.  You can use it to relocalize yourself to a previous session map that you might have saved, or to send the map across to another device so that that device can relocalize itself to your space.

How to use it in Unity: see the _Examples/ARKit2.0/UnityARWorldMap/UnityARWorldMap.unity_ for an example

Every session builds up a **ARWorldMap** as you move around and detect more feature points.  To get the current **ARWorldMap** of a session, you have to use an asynchronous method:

```CS
       session.GetCurrentWorldMapAsync(OnWorldMap);       
```

And then on the callback, you will get an **ARWorldMap**, which is the current world map.  You can keep it in memory somewhere, send it to someone else or save it:

```CS
    void OnWorldMap(ARWorldMap worldMap)
    {
        if (worldMap != null)
        {
            worldMap.Save(path);
            Debug.LogFormat("ARWorldMap saved to {0}", path);
        }
    }

```


You can load the world map if you know where you saved it:

```CS
        var worldMap = ARWorldMap.Load(path);
```

Once you have the **ARWorldMap**, either from loading it, or from memory, or from receiving it from another device, you can align yourself to share coordinate systems with that **ARWorldMap** by setting a parameter in the config and resetting the ARSession:

```CS
            config.worldMap = worldMap;

            Debug.Log("Restarting session with worldMap");
            session.RunWithConfig(config);
```

What this does is reset the session, and as you move around, it tries to matchup the feature points in the worldmap to the feature points it's detecting in your environment.  When they matchup, it relocalizes your device coordinates to match up with the coordinates that were saved in the **ARWorldMap**.


![alt text] (images/UnityARWorldMap.m4v "UnityARWorldMap scene example")

ARWorldMap can also be serialized to a byte array, and sent across to another device using WiFi, Bluetooth or some other means of sharing.  It can also be deserialized on the other side and used to relocalize the other device to the same world mapping as the first device, so that you can have a shared multiplayer experience.

```
		public byte [] SerializeToByteArray();  //ARWorldMap instance method
		
		public static ARWorldMap SerializeFromByteArray(byte[] mapByteArray);  //ARWorldMap static method

```

With the introduction of **ARWorldMap**, ARKit 2.0 also introduces the concept of **ARWorldMappingStatus** that is updated every frame on the **ARFrameUpdatedEvent** of the session.  The meaning of the status is documented in the code:

```
	public enum ARWorldMappingStatus
	{
		/** World mapping is not available. */
		ARWorldMappingStatusNotAvailable,

		/** World mapping is available but has limited features.
     For the device's current position, the session’s world map is not recommended for relocalization. */
		ARWorldMappingStatusLimited,

		/** World mapping is actively extending the map with the user's motion.
     The world map will be relocalizable for previously visited areas but is still being updated for the current space. */
		ARWorldMappingStatusExtending,

		/** World mapping has adequately mapped the visible area.
     The map can be used to relocalize for the device's current position. */
		ARWorldMappingStatusMapped

	}

```
See _Examples/ARKit2.0/UnityARWorldMap/UpdateWorldMappingStatus.cs_ for usage.

## ARReferenceObject and ARObjectAnchor

Similar to **ARReferenceImage** and **ARImageAnchor** that existed in ARKit 1.5, we now have **ARReferenceObject** and **ARObjectAnchor** to do detection of objects.

How to use it in Unity: see the _Examples/ARKit2.0/UnityARObjectAnchor/UnityARObjectAnchor.unity_ for example

Again very similar to **ARReferenceImage**, we're going to set up a **ARReferenceObjectsSetAsset**, which contains references to **ARReferenceObjectAssets**, and add that to the config for **ARSession** so that it tries to detect the **ARReferenceObjects** that correspond to those when in the session.

To create an **ARReferenceObjectAsset**, in the Editor go to your Project view into the directory where you want to create the asset, and bring up the _Create/UnityARKitPlugin/ARReferenceObjectAsset_ menu:

![alt text] (images/ARReferenceObjectAsset_creation.png)

Then in the inspector, when **ARReferenceObjectAsset** is selected, populate the Reference Object field with the raw .arobject file that has also been moved over to a project folder.

![alt text] (images/ARReferenceObject_Inspector.png)

Finally, add a bunch of these **ARReferenceObjectAssets** to a **ARReferenceObjectsSetAsset** created in the same way as above:

![alt text] (images/ARReferenceObjectsSetAsset_Inpsector.png)

Now add a reference to this **ARReferenceObjectsSetAsset** to your **ARKitWorldTrackingSessionConfiguration.detectionObjects** field and start your session.  It should detect the objects that you have specified.

To carry out some action when a specific object is detected, you first need to subscribe to the event:

```
		UnityARSessionNativeInterface.ARObjectAnchorAddedEvent += AddObjectAnchor;
```

Next, when that event is triggered, you can check that the name of the anchor matches the name of the reference object asset that you specified, and then decide what you want to do (e.g create a prefab at that location):

```
		if (arObjectAnchor.referenceObjectName == referenceObjectAsset.objectName) {
			Vector3 position = UnityARMatrixOps.GetPosition (arObjectAnchor.transform);
			Quaternion rotation = UnityARMatrixOps.GetRotation (arObjectAnchor.transform);

			objectAnchorGO = Instantiate<GameObject> (prefabToGenerate, position, rotation);
		}

```

You may create .arobject files either from the UnityObjectScanner example described below, or from Apple's ARKit Object Scanner app which both produce the same format of file for use in the workflow above.

![alt text] (images/UnityObjectAnchor.m4v "UnityObjectAnchor scene example")


You can also look at a more complete example that does object creation using a pickable bounding box and object detection:

_Examples/ARKit2.0/UnityARObjectScanner/UnityARObjectScanner.unity_

You can save .arobjects that you have scanned using this example.  To get the saved .arobjects to your Mac so that you can populate your app with reference objects, you can use iTunes FileSharing while you have your device connected to your Mac:

![alt text] (images/ARReferenceObject_FileShare.png)

Once you have the files on the Mac, you can rename them before you put them into your Unity project.

This scene has different modes: the scanning mode and the detecting mode.  For the scanning mode, we use a **ARKitObjectScanningSessionConfiguration**, which does a more detailed exploration of the scene, but which uses more CPU and power (so it should be limited in use).  

Using this configuration, you can tap on a plane near the object that you want to scan to produce a red bounding box to cover it.  You can manipulate the box so that it just covers the object of interest.  Then scan all around the box to get as many feature points on the object as possible.  Then you create an **ARReferenceObject** by calling an asynchronous function on the session with a callback to handle the created reference object:

```
		session.ExtractReferenceObjectAsync (objectTransform, center, extent, (ARReferenceObject referenceObject) => {
			if (referenceObject != null) {
				Debug.LogFormat ("ARReferenceObject created: center {0} extent {1}", referenceObject.center, referenceObject.extent);
				referenceObject.name = "objScan_" + objIndex++;
				Debug.LogFormat ("ARReferenceObject has name {0}", referenceObject.name);
				scannedObjects.Add(referenceObject);
				UpdateList();
			} else {
				Debug.Log ("Failed to create ARReferenceObject.");
			}
		});

```


The detecting mode works like the object anchor example above, but uses a method for dynamically adding the **ARReferenceObjects** to the list of detected objects. Once you have the **ARReferenceObjects** in a list, you can create a **ARReferenceObjectsSet** from them and pass that to the session for detection:

```
		//create a set out of the scanned objects
		IntPtr ptrReferenceObjectsSet = session.CreateNativeReferenceObjectsSet(scannedObjects);

		//restart session without resetting tracking 
		var config = m_ARSessionManager.sessionConfiguration;

		//use object set from above to detect objects
		config.dynamicReferenceObjectsPtr = ptrReferenceObjectsSet;

		//Debug.Log("Restarting session without resetting tracking");
		session.RunWithConfigAndOptions(config, UnityARSessionRunOption.ARSessionRunOptionRemoveExistingAnchors | UnityARSessionRunOption.ARSessionRunOptionResetTracking);

```
 

Here is a video that shows you this example in action:

![alt text] (images/UnityObjectScanner.m4v "UnityObjectScanner scene example")


## AREnvironmentProbeAnchor

This is a new kind of anchor that can either be generated automatically or you can specify where to create it.  This anchor creates and updates a reflected environment map of the area around it based on the ARKit video frames and world tracking data.  It also uses a machine learning algorithm to approximate the environment texture for parts of the scene it has not seen yet, based on a training model involving thousands of environments.

How to use it in Unity: see the _Examples/ARKit2.0/UnityAREnvironmentTexture_ folder for examples

![alt text] (images/UnityEnvironmentAnchor.m4v "UnityAREnvironmentProbeAnchor scene example")

There is a new parameter on the `ARKitWorldTrackingSessionConfiguration` that controls this feature:

```
            config.environmentTexturing = environmentTexturing;
```

where `environmentTexturing` can be one of the values in the enum:

```
	public enum UnityAREnvironmentTexturing
	{
		UnityAREnvironmentTexturingNone,
		UnityAREnvironmentTexturingManual,
		UnityAREnvironmentTexturingAutomatic
	}
```

With the _UnityAREnvironmentTexturingManual_ mode, you will have to create an **AREnvironmentProbeAnchor** yourself, but ARKit will update the texture that is captured from the environment.  You can see how this is done in the _Examples/ARKit2.0/UnityAREnvironmentTexture/UnityAREnvironmentProbeAnchorManual.unity_ scene.

We use a hit test against any planes to detect a position to create the anchor. Then we actually create the anchor there using this code:

```
		UnityAREnvironmentProbeAnchorData anchorData;
		anchorData.cubemapPtr = IntPtr.Zero;
		anchorData.ptrIdentifier = IntPtr.Zero;
		anchorData.probeExtent = Vector3.one;
		anchorData.transform = UnityARMatrixOps.GetMatrix (worldTransform); //this should be in ARKit coords
		anchorData = UnityARSessionNativeInterface.GetARSessionNativeInterface ().AddEnvironmentProbeAnchor (anchorData);
```


_Note that we need to give the anchor a transform matrix using the ARKit coordinate system._

If you use _UnityAREnvironmentTexturingAutomatic_ mode instead, ARKit will generate the **AREnvironmentProbeAnchors** in procedurally spaced intervals according to the data it infers from your session and your movement through the space.

Both of the examples generate a prefab that contains a Unity **ReflectionProbe** component, and sets the `customBakedTexture` that it uses to the `Cubemap` that the environment probe anchor gets updated with.  This **ReflectionProbe** now participates in the standard Unity rendering pipeline and will enhance any GameObject that uses it.  Read up on [Reflection Probes](https://docs.unity3d.com/Manual/ReflectionProbes.html) in the Unity docs.  In our example prefabs, we also put in a fully reflective sphere to visualize what the reflection probes look like, but you can put any virtual object in the scene and it will be affected by them.


## ARReferenceImage improvement (now tracked)

Reference images work the same as before, but now instead of just recognizing images, ARKit allows you to track them:  when you move the reference image, the Image Anchor associated with them moves with the image, so you can have content that is anchored on those moving images.  There is one extra parameter on the `ARKitWorldTrackingSessionConfiguration` that allows you to do this like so:

```
            config.maximumNumberOfTrackedImages = maximumNumberOfTrackedImages;
```

where `maximumNumberOfTrackedImages` is an integer specigying how many images you want tracked simultaenously during this session.


## Face tracking improvements

ARKit 2.0 also improves face tracking for iPhone X with some new features.  First, there is one new **BlendShapeLocation** called **TongueOut**.  Apple showed this at WWDC on their animojis, and it appeared to be very popular.

The other improvement is that it now does eye gaze tracking.  You receive a transform that describes where each eye on the face is pointed at, as well as the position of the object which is being looked at.

Take a look at _Examples/ARKit2.0/UnityTongueAndEyes/UnityTongueAndEyes.unity_ for an example of how to make use of this new data coming in from the face anchors.

![alt text] (images/UnityTongueAndEyes.gif "UnityTongueAndEyes scene example")


