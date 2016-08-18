# Ease for Unreal Engine

This is a sample application to demonstrate and test the Ease plugin for Unreal Engine.

The plugin is written in C++ and is in the `EaseUnreal/Plugins/Ease/EasePlugin` folder.

The source files that implement the plugin API are in `Source/EasePlugin/Classes/EaseLibrary.h` and `Source/EasePlugin/Private/EaseLibrary.cpp` (relative to the plugin folder).

The sample app is based on the Blueprints version of the Flying sample that comes with Unreal Engine. The Ease setup is in `EaseUnreal/Content/FlyingBP/Blueprints/FlyingPawn.uasset` in the Ease Setup block at the top of the blueprint. The rest of the blueprint is mostly unchanged from the original.

This block calls several functions from the plugin, along with several blueprint functions defined in the same `.uasset` file. It creates Ease markers for each of the rectangular obstacles that appear in the Flying sample. Here are the Unreal Engine events it handles:

### BeginPlay

The `BeginPlay` event first calls `EaseSessionBegin`. This function must be called before any other Ease functions. The `ApiKey` and `ExperienceID` should be filled in with your actual values. The checkboxes at the bottom are options for logging, enabling the `Presence` event, and enabling `PostData`. If the latter is disabled, nothing is sent to the Ease server; this is useful for initial local testing. Enable either or both of the Log options to log Ease activities to your console.

Next, this event calls the `EaseCreateMarkerArray` blueprint function to create an array of the `Actor` objects that should be treated as markers. In the Flying sample, the rectangular obstacles have names that begin with `TemplateCube_Rounded_`, so the function looks for that name prefix and adds each matching `Actor` to the `EaseMarkers` array.

Finally, it calls the `EaseForEachMarker` blueprint function to enumerate the marker array, and calls the `EaseMarkerAdd` plugin function to register each marker.

In a real app, you may have a completely different way of keeping track of your markers. The blueprint functions in this sample app are just an illustration of one way to add markers to an existing app and call the underlying plugin functions.

### EndPlay

The `EndPlay` event calls the `EaseForEachMarker` blueprint function and `EaseMarkerRemove` plugin function to remove each marker, and finally calls `EaseSessionEnd`. Removing the markers when the session ends is optional and is included here just as an example. You're more likely to use `EaseMarkerRemove` if you are adding and removing markers dynamically. Markers can be added and removed at any time.

### Tick

The existing sample app blueprint handles the `Tick` event already, so we splice in a `Sequence` node to let the original code run first, followed by our extra code.

This code first calls the `EaseTick` plugin function. This function simply uses the incoming `DeltaSeconds` to calculate the current frame rate for reporting. Then, the `EaseMarkerTick` blueprint function is called.

That function uses `LineTraceByChannel` to perform a raycast from the current camera position and direction. The multiplication node after `GetForwardVector` determines the length of the trace.

The function then calls the `EaseGetActorMarker` blueprint function to detect whether any `Actor` hit by the raycast corresponds to an Ease marker, and then calls the `EaseMarkerHit` plugin function with either the `Actor` reference or `null` if no marker was hit.

Because raycasting is a moderately expensive operation, we leave this logic to the application code instead of implementing it inside `EaseTick`. You may have an existing trace that can handle the marker hit test, or you may want to use a specific channel for markers or other techniques.

`EaseMarkerHit` is a convenience function to make Blueprint programming a bit easier. It calls the underlying `EaseMarkerEnter` and `EaseMarkerExit` functions as needed, by keeping track internally of the current marker in view, and using the `Actor` name and location.

You can call those latter two functions directly for more control. This allows you to pass a data string along with the marker name, and also it lets you support "see through" markers. For example, imagine a small marker A with a large marker B behind it, and move the camera across them so it first hits marker B, then marker A, then marker B again and finally exits.

With the `EaseMarkerHit` approach, this fires these events:

    Enter B
    Exit B
    Enter A
    Exit A
    Enter B
    Exit B

By using `MultiLineTraceByChannel` and your own logic, you could alternatively get this sequence of events:

    Enter B
    Enter A  (here we are "in" both A and B)
    Exit A
    Exit B

## RFC

This version of the Ease plugin and sample app is an early work in progress. We welcome your comments, questions, suggestions for improvement, and pull requests. Thanks!
