**IMPORTANT: Before the workshop, please download Unity Hub and Unity version 2020.3 for this project, as it may take a long time to download and install Unity. It could take 30 minutes or longer depending on your internet speed :(.**

**Also, please download the ARFoundationSim.unitypackage AND SimEnvironments.unitypackage from the above repo.**

We will be walking through Part 1 during the workshop.

**Pre-requisites**: It is expected to have a little bit of object oriented programming experience.

Follow us here!\
Website: https://xr.berkeley.edu/ \
FB: https://www.facebook.com/xratberkeley \
Discord: https://discord.gg/GvGUUCN

# Part 1: Building a Unity Game in Simulated AR

## Setting Up Your Project:

Note that the assets used were tested on Unity 2020.3.17f1. It may work in other versions too.

Download the ARFoundationSim.unitypackage AND SimEnvironments.unitypackage from the repo above.

In Unity Hub create a new 3D Project. Import the ARFoundationSim.unitypackage and SimEnvironments.unitypackage using

Assets->Import 

![i1.JPG](/Instructions/i1.JPG)

First delete the main camera that is placed in the scene by default by unity. We will replace this by it’s AR Equivalent. Search for the "AR Camera" Prefab and drag it into the scene hierarchy.


Next, we load a sample simulation scene. Search for "Miniworld_FloorPlan229_physics" prefab and load this into the scene. This is the virtual environment in which you will spawn your AR game. This contains a simplified version of the scene below, as well as any associated AR planes in it.
![i3.JPG](/Instructions/i3.JPG)


We see that, the AR camera is seeing the wrong parts of the scene. Set the transform values through the inspector, that will make sure AR camers gets a good view of the scene. For instance, here's a set of values you may use:
![i2.JPG](/Instructions/i2.JPG)


Now to help interface with all the AR Planes and their associated interations, we will add our ARPlaneManager, and ARRaycastManager into the scene. To do this, create a new game Object in the scene hierarchy and call it "Managers". In the inspection tab select Add Component and in the search box type “AR Plane Manager” and add it.

![i4.JPG](/Instructions/i4.JPG)

Now, save the scene. Click Play. In the scene hierarchy, select "Managers" and Change the Value of "Detection Mode". You should observe that different types of planes are detected in the AR scene.





## Placing Your Game Board:

Go to GameObject -> 3D Object -> Cube and name your new cube, “Game Board”. Lets change this into more of a game board shape. Select the Game Board and in the inspector set the scale x,y and z values to 0.6, 0.02, and 0.6 respectively. Unity is set up such that the values of 1 unit in game coordinates corresponds to 1 meter in physical coordinates. Since the cube model is a 1 unit cube, these scale parameters correspond to a game board that is 60cm width and height with a 2cm thickness. 

For now, we will also deactivate our game board so that will not be present at the start of our game. To do this, deselect the checkbox at the very top of the inspector tab for your game board (right above the word “tag”).

![image14.png](/Instructions/image14.png)

Now we need to set up the code that allows the user to choose a position for the game board. Since we don’t know what the area will look like in advance, we will let the user choose where they want to place the game board. To the Managers, we will add the component ARRaycastManager. Raycasting is how we convert a 2D position on the screen to a 3D position in world space. The ARRaycastManager lets us raycast to the planar regions detected by the AR Plane Manager we created in the previous step. 

Next we will add the script to actually place the game board. To your Managers select Add Component -> New Script and name it PlaceGameBoard. Double click on the script (in the box next to the word script, not the component itself) to open it. 

Set up your script as shown below. For all code is this document, be sure to read the code and comments to understand what the code is doing. 
```C++

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
//This allows us to use the AR Foundation simulator functions
using sim.part1;
public class PlaceGameBoard : MonoBehaviour
{
    // Public variables can be set from the unity UI.
    // We will set this to our Game Board object.
    public GameObject gameBoard;
    // These will store references to our other components.
    private ARRaycastManager raycastManager;
    private ARPlaneManager planeManager;
    // This will indicate whether the game board is set.
    private bool placed = false;

    // Start is called before the first frame update.
    void Start()
    {
        // GetComponent allows us to reference other parts of this game object.
        raycastManager = GetComponent<ARRaycastManager>();
        planeManager = GetComponent<ARPlaneManager>();

        //We want to place our board only on hortizontal planes. So we tell the plane manager only to detect those
        planeManager.detectionMode = PlaneDetectionMode.Horizontal;
    }

    // Update is called once per frame.
    void Update()
    {
        if (!placed)
        {
            if (Input.GetMouseButtonDown(0))
            {
                Vector2 touchPosition = Input.mousePosition;

                // Raycast will return a list of all planes intersected by the
                // ray as well as the intersection point.
                List<ARRaycastHit> hits = new List<ARRaycastHit>();
                if (raycastManager.Raycast(
                    touchPosition, ref hits, TrackableType.PlaneWithinPolygon))
                {
                    // The list is sorted by distance so to get the location
                    // of the closest intersection we simply reference hits[0].
                    var hitPosition = hits[0].hitPosition;
                    // Now we will activate our game board and place it at the
                    // chosen location.
                    gameBoard.SetActive(true);
                    gameBoard.transform.position = hitPosition;
                    placed = true;
                    // After we have placed the game board we will disable the
                    // planes in the scene as we no longer need them.
                    
                    planeManager.detectionMode = PlaneDetectionMode.None;

                }
            }
        }
        else
        {
            // The plane manager will set all detected planes to active by 
            // default so we will continue to disable these.
            //planeManager.SetTrackablesActive(false); //For older versions of AR foundation
            planeManager.detectionMode = PlaneDetectionMode.None;
        }
    }

    // If the user places the game board at an undesirable location we 
    // would like to allow the user to move the game board to a new location.
    public void AllowMoveGameBoard()
    {
        placed = false;
        //planeManager.SetTrackablesActive(true);
        planeManager.detectionMode = PlaneDetectionMode.Horizontal;
    }

    // Lastly we will later need to allow other components to check whether the
    // game board has been places so we will add an accessor to this.
    public bool Placed()
    {
        return placed;
    }
}

```
When you return to the unity editor you should see your script component now has a field for “Game Board”. Drag your game board object from your scene hierarchy into this field. 


You will notice we haven’t actually used our function to allow the user to move the game board once it is placed. So let’s implement that. For this we are going to add a button to the screen that calls “AllowMoveGameBoard” when pressed. 

Select GameObject->UI->Canvas to add a Canvas to the scene. The Canvas is where we will place all 2D UI elements that are meant to appear attached to the screen. Now create a button GameObject->UI->Button. It should automatically appear under the canvas in the hierarchy.

![image5.png](/Instructions/image5.png)

First let’s set the button position and size to be more reasonable than the default. Select your button and inside the inspector change the width and height to 160 and 80 respectively. Then click on the icon showing multiple boxes and lines, above the word “Anchors”. This allows you to choose the general position of your button on the screen. Select bottom+center.  Now change the X and Y values of the Pivot field to 0.5 and 0 respectively. Lastly zero out the transform position, as these will have changed as you were modifying the other values to try and keep the button at its original position.

![image11.png](/Instructions/image11.png)

If you build and run now you should see a button present at the bottom of your screen, but clicking it doesn’t actually do anything. 

To change this, in your button inspector, scroll down to the field labeled “On Click”. Press the + selection at the bottom of this field. Where the word “none” appears now, drag your Managers from your scene hierarchy into this location. Lastly change the “No Function” selection to PlaceGameBoard->AllowMoveGameBoard(). If you build and run now, after placing your game board, pressing this button should bring back the place visualization and allow you to move your game board to a new location. 

![i6.PNG](/Instructions/i6.PNG)
Lastly let’s change the button label to something more intuitive than “Button”. Expand your Button object in the scene hierarchy and select the Text object that appears below it. Change the text field under “Text (Script)” in the inspector to “Move Board”. Build and Run to see your changes. 

Making A Simple Interactable Object:

Making interactable objects in AR is fairly easy. The short version is, we just have to check if a raycast from a user’s touch intersects with an interactable object, and call a function from that object. 

So let’s make a couple objects to interact with. Create two new cubes in the scene, rescale them to be 10cm x 10cm x 10cm, and rename them “AR Button 1” and “AR Button 2”. Drag them to be part of the Game Board in your hierarchy. Reactivate your game board so you can see it, and then reposition your buttons to be at two separate locations on your game board. Don’t forget to deactivate your game board again once you do this. 

![image6.png](/Instructions/image6.png)

To make it easy for us to call different functions for each button we will create an abstract class which each button will inherit from. In the project window at the bottom of the screen, Right Click->Create->C# Script and name it OnTouch3D. All you need to place in this script is:

```C++
public interface OnTouch3D
{
    void OnTouch();
}  
```

And then our AR Button 1 will inherit from this script and implement this function. For a simple interaction we will make the object move up by 10cm when pressed. In AR Button 1, Add Component -> New Script and name it “ARButton1”. Open this script and place the following code:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Adding OnTouch3D here forces us to implement the 
// OnTouch function, but also allows us to reference this
// object through the OnTouch3D class.
public class ARButton1 : MonoBehaviour, OnTouch3D
{
    // Debouncing is a term from Electrical Engineering referring to 
    // preventing multiple presses of a button due to the physical switch
    // inside the button "bouncing".
    // In CS we use it to mean any action to prevent repeated input. 
    // Here we will simply wait a specified time before letting the button
    // be pressed again.
    // We set this to a public variable so you can easily adjust this in the
    // Unity UI.
    // While this is not so much useful in simulation where we just do click usign a mouse, it is 
    // essential when we dpeloy our app to a phone, where the input is through touch.
    
    public float debounceTime = 0.3f;
    // Stores a counter for the current remaining wait time.
    private float remainingDebounceTime;

    void Start()
    {
        remainingDebounceTime = 0;
    }

    void Update()
    {
        // Time.deltaTime stores the time since the last update.
        // So all we need to do here is subtract this from the remaining
        // time at each update.
        if (remainingDebounceTime > 0)
            remainingDebounceTime -= Time.deltaTime;
    }

    public void OnTouch()
    {
        // If a touch is found and we are not waiting,
        if (remainingDebounceTime <= 0)
        {
            // Move the object up by 10cm and reset the wait counter.
            this.gameObject.transform.Translate(new Vector3(0, 0.1f, 0));
            remainingDebounceTime = debounceTime;
        }
    }
}
```


Next let’s add a tag to our object to make it easy to tell that this object is interactable. Select your AR Button 1 and at the top of the inspector window, expand the “Tag” dropdown list and select “Add Tag”. Select + and add the tag “Interactable”. This tag will show up on the Tag dropdown list from now on. Select this as the tag for AR Button 1.

![image4.png](/Instructions/image4.png)

Now we need to create the script to actually perform the raycasting to this object. In Managers, Add Component -> New Script and name it “ARButton Manager”. In this script place the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using sim.part1;

public class ARButtonManager : MonoBehaviour
{
    private Camera arCamera;
    private PlaceGameBoard placeGameBoard;

    void Start()
    {
        // Here we will grab the AR camera 
        // This camera acts like any other camera in Unity.
        arCamera = FindObjectOfType<ARCamera>().GetComponent<Camera>();
        // We will also need the PlaceGameBoard script to know if
        // the game board exists or not.
        placeGameBoard = GetComponent<PlaceGameBoard>();
    }

    void Update()
    {
        if (placeGameBoard.Placed() && Input.GetMouseButtonDown(0))
        {
            Vector2 touchPosition = Input.mousePosition;
            // Convert the 2d screen point into a ray.
            Ray ray = arCamera.ScreenPointToRay(touchPosition);
            // Check if this hits an object within 100m of the user.
            //RaycastHit hit;
            //if (Physics.Raycast(ray, out hit,100))
            RaycastHit[] hits;
            hits = Physics.RaycastAll(ray, 100.0F);
            for (int i = 0; i < hits.Length; i++)
            {
                // Check that the object is interactable.
                if(hits[i].transform.tag=="Interactable")
                    // Call the OnTouch function.
                    // Note the use of OnTouch3D here lets us
                    // call any class inheriting from OnTouch3D.
                    hits[i].transform.GetComponent<OnTouch3D>().OnTouch();
            }
        }
    }
}
```
Build and run your game. When you place your game board, you should now see that this button moves up when you touch it. 

It is worth noting that that the button does not actually have to be visible for you to interact with this. This is useful if you want the user to be able to interact with empty spaces on the game board, or just want the selectable region to be bigger than the object that is displayed. 

Let’s turn AR Button 2 into an invisible button. First, don’t forget to add the “Interactable” tag since we are now going to set up interaction for this button. Then disable the checkbox next to the “Mesh Renderer” component. This stops the button from being rendered, making it essentially invisible. 

![image12.png](/Instructions/image12.png)

Since moving an invisible object doesn’t make much sense, let’s instead have this object display a message on the screen. Add a new script to AR Button 2 and name it “ARButton2”. Open the script and add the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ARButton2 : MonoBehaviour, OnTouch3D
{
    public Text messageText;

    public void OnTouch()
    {
        messageText.gameObject.SetActive(true);
        messageText.text = "Button2Pressed";
    }
}
```

We will then need to create the Text object for this to reference. Add a Text object to the scene GameObject->UI->Text and rename it “Message Text”. We will leave this object centered, but be sure the (X,Y,Z) positions are all set to 0. Change the width and height to 250 and 60 respectively and change the font size to 28.  As with our Game Board we are going to set this to inactive to start. 

![image13.png](/Instructions/image13.png)

To make this a proper message, let’s also add a script to make the text go inactive again after a specified time. Add a new script component and name it DisappearingText. Open it and add the following:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DisappearingText : MonoBehaviour
{
    // displayTime will be set to a public float
    // so that you can easily change it in the Unity UI
    public float displayTime = 1;

    private float timeRemaining;

    // Start is called before the first frame update
    void Start()
    {
        timeRemaining = displayTime;
    }

    // Update is called once per frame
    // but only while the object is active
    void Update()
    {
        timeRemaining -= Time.deltaTime;
        if (timeRemaining < 0)
        {
            timeRemaining = displayTime;
            this.gameObject.SetActive(false);
        }
    }
}
```

Next let’s add a panel to make the text more visible against the background scene. Right Click on your Message Text object in the scene hierarchy and select UI->Panel. This should automatically place a panel under the Message Text object. 

![image5.png](/Instructions/image5.png)

Select the panel and change the “Source Image” in the inspector to UISprite. 

![image1.png](/Instructions/image1.png)

Finally, select your AR Button 2 again and drag your Message Text object into the Message Text section of your AR Button 2 script. 

![image2.png](/Instructions/image2.png)

Build and run your game. After placing your game board you should now be able to click on the location where the button would be (if it were visible) and a message should appear on screen. 

## Finish your game:

The rest is up to you! Using these techniques you need to now design your own game that can run in this simulated environment.

# Part 2: Deploying Onto a Mobile Device

## Before You Start:

Make sure you have a compatible AR device - that is, a device that supports ARKit on iOS or ARCore on Android. 
A list of supported Android devices can be found here:
https://developers.google.com/ar/discover/supported-devices 
A list of supported iOS devices is at the very bottom of this page: https://www.apple.com/ios/augmented-reality/ 

Note: For ios devices you will also need to be developing on an Apple computer.
If you do not have a supported device combination (either a supported Android device or an iOS device + an Apple computer), you may borrow an Android phone from us. Our supply of Android phones is limited so please use your own device if possible. Plus if you use your own device you will always be able to show off your app to friends and family.  

### For Android users: 
You will need to download and install Android Studio to deploy to your device.
You will need to include Android Build Support in your Unity install
To do this: In Unity Hub go to Installs->...->Add Modules and make sure Android Build Support is checked.

### For iOS users:
You will need to download and install XCode to deploy to your device.
You will need to include iOS Build Support in your Unity install
To do this: In Unity Hub go to Installs->...->Add Modules and make sure iOS Build Support is checked.


## Setting Up Your Project:

In Unity Hub create a new 3D Project.

### For Android users:
* Go to Window -> Package Manager 
    * Select Packages -> Unity registry
    * Select ARCore XR Plugin and install it.
    * Select AR Foundation and install it. 
* Go to File -> Build Settings
   * Select Android and press “Switch Platform” (this may take about a minute to complete)
   * Go to Player Settings -> Other Settings
      * Change Package Name to: com.<your name>.<your project> example: com.johndoe.arboardgame
      * Set Minimum API Level to Android 7.0
   * Select Vulkan in Graphic APIs, and remove it by pressing the '-' button
 
 ![](/Instructions/vulkan.PNG)

### For iOS users:
* Go to Window -> Package Manager 
   * Select Packages -> Unity registry
   * Select ARKit XR Plugin, ensure the version is set to 2.1.1, and install it.
   * Select AR Foundation and install it. 
* Go to File -> Build Settings
   * Select iOS and press “Switch Platform” (this may take about a minute to complete)
   * Go to Player Settings -> Other Settings
      * Change Package Name to: com.<your name>.<your project> example: com.johndoe.arboardgame
      * Set Signing Team ID to the ID for your Apple account. To find this open Keychain Access application (Applications > Utilities > Keychain Access on your Apple computer) under My Certificates double click on your iPhone Developer certificate. Your Signing Team ID is the value listed under Organization Unit.
         * If this is not present, sign in to developer.apple.com with your Apple ID, you may be asked to accept the Developer Agreement. Do this, but do NOT sign up for a paid account. 
         * Open XCode->Preferences->Accounts and add your Apple ID. You will then need to create an XCode project, XCode->File->New->Project and create any iOS project, to initialize a certificate.
      * Check: Automatically Sign
      * Set target minimum iOS Version to: 11.0
      * Select: Requires ARKit support 
      * Set Architecture to: ARM64

At this point you should be able to build a blank API by pressing File-> Build Settings -> Build


## Identifying Planar Surfaces in AR:

Now you are ready to build your first AR App. The first thing we will do is create a visualizer for planar surfaces detected by your device. Later you will use these planes as locations where you can place your game board. 

First delete the main camera that is placed in the scene by default by unity. We will replace this by it’s AR Equivalent. This requires two objects: AR Session and AR Session Origin. At the bottom of your screen you should see a window labeled Project. In the search box type “ARSession” and drag both the ARSession and ARSessionOrigin objects into the scene hierarchy. 

In the newest version of AR Foundation these will instead be in GameObject (or '+' or Right Click in 'Hierarchy') -> XR -> ARSession/ ARSessionOrigin

Select AR Session Origin. In the inspection tab select Add Component and in the search box type “AR Plane Manager” and add it. You will notice that the plane prefab field is empty. We will fill this field by creating our own plane prefab. 

In you scene hierarchy create an empty game object and name it “Plane Visualization Object”. Select this object. We are going to add several components to this: 
AR Plane, AR Plane Mesh Visualizer, Mesh Renderer, Mesh Collider and Mesh Filter

![image15.png](/Instructions/image15.png)

This handles all the aspects for modifying and visualizing the plane, however it is currently using the default material for visualization which is opaque and not great for visualization. So let’s create our own material.  

In your project window, in the Assets folder, create a new folder and name it Materials. In this folder, Right Click->Create->Material and name it whatever you like. Select this material, set the rendering mode to: Transparent and click on the color selector next to Albedo. You can choose whatever color you like but set the Alpha Channel (the slider labeled A) to 75. Lastly change the source from “Metallic Alpha” to “Albedo Alpha”. This allows whatever object we attach this material to to appear semi-transparent. 

![image7.png](/Instructions/image7.png)

Select our Plane Visualization Object and drag your new material into its component list. This completes our object. To turn it into a prefab, first create a Prefabs folder under Assets. Then simply drag our Plane Visualization Object from the hierarchy into this folder. You can now delete the object from the scene hierarchy. 

Lastly drag your new prefab over to the Plane Prefab section of AR Session Origin-> AR Plane Manager.

![image10.png](/Instructions/image10.png) 

Now it is time to build and run your application. Simply plug in your Mobile Device via USB and in Unity select File->Build And Run. You will likely get an error. 

### For Android Users:
You will likely need to enable developer options on your device. On your phone go to Settings -> System -> About Phone and tap the Build Number repeatedly until you get the message “You are a Developer” 
### For iOS Users: 
You will need to tell your device to trust your developer certificate. On your phone go to Settings -> General -> Device Management->Your Apple ID->Trust your Apple ID

Either redo “Build and Run”, or at this point you can just open the installed application on your device. If you move your device slowly around your area you should see semi-transparent planes overlaid over planar regions of the scene such as the floor, your desk, or your keyboard. Note that it may take a few seconds for planes to be detected and they will only appear in regions that have visible texture (sorry no playing AR games in the dark or on white walls). 


### Possible Issues

 - If you see a black screen: Select Edit-> Project Settings from Menu and make sure ARCore/ARKit is checked

![image20.PNG](/Instructions/image20.PNG) 

## Changes from part1:

The following sections will walk you through code snippets for developing AR applications for mobile devices. Most of the code snippets is similar to the corresponding counter parts in part1. Wherever, a change of code is present, it is marked either with a `TO_COMMENT' or 'TO_ADD' tag. 
'TO_COMMENT' indicates the line/block of code that needs to be commented out of scripts that you used in part1
'TO_ADD' marks the line/block of code that needs to be added.


## Placing Your Game Board:

Go to GameObject -> 3D Object -> Cube and name your new cube, “Game Board”. Lets change this into more of a game board shape. Select the Game Board and in the inspector set the scale x,y and z values to 0.6, 0.02, and 0.6 respectively. Unity is set up such that the values of 1 unit in game coordinates corresponds to 1 meter in physical coordinates. Since the cube model is a 1 unit cube, these scale parameters correspond to a game board that is 60cm width and height with a 2cm thickness. 

For now, we will also deactivate our game board so that will not be present at the start of our game. To do this, deselect the checkbox at the very top of the inspector tab for your game board (right above the word “tag”).

![image14.png](/Instructions/image14.png)

Now we need to set up the code that allows the user to choose a position for the game board. Since we don’t know what the area will look like in advance, we will let the user choose where they want to place the game board. To the AR Session Origin we will add the component ARRaycastManager. Raycasting is how we convert a 2D position on the screen to a 3D position in world space. The ARRaycastManager lets us raycast to the planar regions detected by the Plane Manager we created in the previous step. 

Next we will add the script to actually place the game board. To your AR Session Origin select Add Component -> New Script and name it PlaceGameBoard. Double click on the script (in the box next to the word script, not the component itself) to open it. 

Set up your script as shown below. For all code is this document, be sure to read the code and comments to understand what the code is doing. 
```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
// These allow us to use the ARFoundation API.
//using sim.part1; //TO_COMMENT
using UnityEngine.XR.ARFoundation; //TO_ADD
using UnityEngine.XR.ARSubsystems; //TO_ADD

public class PlaceGameBoard : MonoBehaviour
{
    // Public variables can be set from the unity UI.
    // We will set this to our Game Board object.
    public GameObject gameBoard;
    // These will store references to our other components.
    private ARRaycastManager raycastManager;
    private ARPlaneManager planeManager;
    // This will indicate whether the game board is set.
    private bool placed = false;

    // Start is called before the first frame update.
    void Start()
    {
        // GetComponent allows us to reference other parts of this game object.
        raycastManager = GetComponent<ARRaycastManager>();
        planeManager = GetComponent<ARPlaneManager>();
    }

    // Update is called once per frame.
    void Update()
    {
        if (!placed)
        {
            if (Input.touchCount > 0)
            {
                Vector2 touchPosition = Input.GetTouch(0).position;

                // Raycast will return a list of all planes intersected by the
                // ray as well as the intersection point.
                List<ARRaycastHit> hits = new List<ARRaycastHit>();
                //if (raycastManager.Raycast(//TO_COMMENT
                //    touchPosition, ref hits, TrackableType.PlaneWithinPolygon)) //TO_COMMENT
                 if (raycastManager.Raycast(//TO_ADD
                    touchPosition, hits, TrackableType.PlaneWithinPolygon)) //TO_ADD
                {
                    // The list is sorted by distance so to get the location
                    // of the closest intersection we simply reference hits[0].
                    //var hitPose = hits[0].pose; //TO_COMMENT
                    var hitPosition = hits[0].pose.position; //TO_ADD
                    // Now we will activate our game board and place it at the
                    // chosen location.
                    gameBoard.SetActive(true);
                    //gameBoard.transform.position = hitPose.position;//TO_COMMENT
                    gameBoard.transform.position = hitPosition; //TO_ADD
                    placed = true;
                    // After we have placed the game board we will disable the
                    // planes in the scene as we no longer need them.
                    //planeManager.SetTrackablesActive(false); //For older versions of AR foundation
                    planeManager.detectionMode = PlaneDetectionMode.None;

                }
            }
        }
        else
        {
            // The plane manager will set newly detected planes to active by 
            // default so we will continue to disable these.
            //planeManager.SetTrackablesActive(false); //For older versions of AR foundation
            planeManager.detectionMode = PlaneDetectionMode.None;
        }
    }

    // If the user places the game board at an undesirable location we 
    // would like to allow the user to move the game board to a new location.
    public void AllowMoveGameBoard()
    {
        placed = false;
        //planeManager.SetTrackablesActive(true);
        planeManager.detectionMode = PlaneDetectionMode.Horizontal;
    }

    // Lastly we will later need to allow other components to check whether the
    // game board has been places so we will add an accessor to this.
    public bool Placed()
    {
        return placed;
    }
}
```
When you return to the unity editor you should see your script component now has a field for “Game Board”. Drag your game board object from your scene hierarchy into this field. 

![image8.png](/Instructions/image8.png)

You should now build and run to test what you have done so far. By selecting a location on a detected plane, you should be able to place your game board. 

You will notice we haven’t actually used our function to allow the user to move the game board once it is placed. So let’s implement that. For this we are going to add a button to the screen that calls “AllowMoveGameBoard” when pressed. 

Select GameObject->UI->Canvas to add a Canvas to the scene. The Canvas is where we will place all 2D UI elements that are meant to appear attached to the screen. Now create a button GameObject->UI->Button. It should automatically appear under the canvas in the hierarchy.

![image5.png](/Instructions/image5.png)

First let’s set the button position and size to be more reasonable than the default. Select your button and inside the inspector change the width and height to 160 and 80 respectively. Then click on the icon showing multiple boxes and lines, above the word “Anchors”. This allows you to choose the general position of your button on the screen. Select bottom+center.  Now change the X and Y values of the Pivot field to 0.5 and 0 respectively. Lastly zero out the transform position, as these will have changed as you were modifying the other values to try and keep the button at its original position.

![image11.png](/Instructions/image11.png)

If you build and run now you should see a button present at the bottom of your screen, but clicking it doesn’t actually do anything. 

To change this, in your button inspector, scroll down to the field labeled “On Click”. Press the + selection at the bottom of this field. Where the word “none” appears now, drag your AR Session Origin from your scene hierarchy into this location. Lastly change the “No Function” selection to PlaceGameBoard->AllowMoveGameBoard(). If you build and run now, after placing your game board, pressing this button should bring back the place visualization and allow you to move your game board to a new location. 

![image3.png](/Instructions/image3.png)

Lastly let’s change the button label to something more intuitive than “Button”. Expand your Button object in the scene hierarchy and select the Text object that appears below it. Change the text field under “Text (Script)” in the inspector to “Move Board”. Build and Run to see your changes. 


## Making A Simple Interactable Object

Making interactable objects in AR is fairly easy. The short version is, we just have to check if a raycast from a user’s touch intersects with an interactable object, and call a function from that object. 

So let’s make a couple objects to interact with. Create two new cubes in the scene, rescale them to be 10cm x 10cm x 10cm, and rename them “AR Button 1” and “AR Button 2”. Drag them to be part of the Game Board in your hierarchy. Reactivate your game board so you can see it, and then reposition your buttons to be at two separate locations on your game board. Don’t forget to deactivate your game board again once you do this. 

![image6.png](/Instructions/image6.png)

To make it easy for us to call different functions for each button we will create an abstract class which each button will inherit from. In the project window at the bottom of the screen, Right Click->Create->C# Script and name it OnTouch3D. All you need to place in this script is:

```C++
public interface OnTouch3D
{
    void OnTouch();
}  
```

And then our AR Button 1 will inherit from this script and implement this function. For a simple interaction we will make the object move up by 10cm when pressed. In AR Button 1, Add Component -> New Script and name it “ARButton1”. Open this script and place the following code:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Adding OnTouch3D here forces us to implement the 
// OnTouch function, but also allows us to reference this
// object through the OnTouch3D class.
public class ARButton1 : MonoBehaviour, OnTouch3D
{
    // Debouncing is a term from Electrical Engineering referring to 
    // preventing multiple presses of a button due to the physical switch
    // inside the button "bouncing".
    // In CS we use it to mean any action to prevent repeated input. 
    // Here we will simply wait a specified time before letting the button
    // be pressed again.
    // We set this to a public variable so you can easily adjust this in the
    // Unity UI.
    public float debounceTime = 0.3f;
    // Stores a counter for the current remaining wait time.
    private float remainingDebounceTime;

    void Start()
    {
        remainingDebounceTime = 0;
    }

    void Update()
    {
        // Time.deltaTime stores the time since the last update.
        // So all we need to do here is subtract this from the remaining
        // time at each update.
        if (remainingDebounceTime > 0)
            remainingDebounceTime -= Time.deltaTime;
    }

    public void OnTouch()
    {
        // If a touch is found and we are not waiting,
        if (remainingDebounceTime <= 0)
        {
            // Move the object up by 10cm and reset the wait counter.
            this.gameObject.transform.Translate(new Vector3(0, 0.1f, 0));
            remainingDebounceTime = debounceTime;
        }
    }
}
```


Next let’s add a tag to our object to make it easy to tell that this object is interactable. Select your AR Button 1 and at the top of the inspector window, expand the “Tag” dropdown list and select “Add Tag”. Select + and add the tag “Interactable”. This tag will show up on the Tag dropdown list from now on. Select this as the tag for AR Button 1.

![image4.png](/Instructions/image4.png)

Now we need to create the script to actually perform the raycasting to this object. In AR Session Origin, Add Component -> New Script and name it “ARButton Manager”. In this script place the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
//using sim.part1; //TO_COMMENT
using UnityEngine.XR.ARFoundation; //TO_ADD
using UnityEngine.XR.ARSubsystems; //TO_ADD

public class ARButtonManager : MonoBehaviour
{
    private Camera arCamera;
    private PlaceGameBoard placeGameBoard;

    void Start()
    {
        // Here we will grab the camera from the AR Session Origin.
        // This camera acts like any other camera in Unity.
        //arCamera = FindObjectOfType<ARCamera>().GetComponent<Camera>();//TO_COMMENT
        arCamera = GetComponent<ARSessionOrigin>().camera; //TO_ADD
        // We will also need the PlaceGameBoard script to know if
        // the game board exists or not.
        placeGameBoard = GetComponent<PlaceGameBoard>();
    }

    void Update()
    {
        //if (placeGameBoard.Placed() && Input.GetMouseButtonDown(0))//TO_COMMENT
        if (placeGameBoard.Placed() && Input.touchCount > 0) //TO_ADD
        {
            //Vector2 touchPosition = Input.mousePosition; //TO_COMMENT
            Vector2 touchPosition = Input.GetTouch(0).position; //TO_ADD
            // Convert the 2d screen point into a ray.
            Ray ray = arCamera.ScreenPointToRay(touchPosition);
            // Check if this hits an object within 100m of the user.

/*TO_COMMENT
            RaycastHit[] hits;
            hits = Physics.RaycastAll(ray, 100.0F);
            for (int i = 0; i < hits.Length; i++)
            {
                // Check that the object is interactable.
                if(hits[i].transform.tag=="Interactable")
                    // Call the OnTouch function.
                    // Note the use of OnTouch3D here lets us
                    // call any class inheriting from OnTouch3D.
                    hits[i].transform.GetComponent<OnTouch3D>().OnTouch();
            }
*/

//TO_ADD ================================================
            RaycastHit hit; 
            if (Physics.Raycast(ray, out hit,100))
            {
                // Check that the object is interactable.
                if(hit.transform.tag=="Interactable")
                    // Call the OnTouch function.
                    // Note the use of OnTouch3D here lets us
                    // call any class inheriting from OnTouch3D.
                    hit.transform.GetComponent<OnTouch3D>().OnTouch();
            }
//================================================
        }
    }
}
```
Build and run your game. When you place your game board, you should now see that this button moves up when you touch it. 

It is worth noting that that the button does not actually have to be visible for you to interact with this. This is useful if you want the user to be able to interact with empty spaces on the game board, or just want the selectable region to be bigger than the object that is displayed. 

Let’s turn AR Button 2 into an invisible button. First, don’t forget to add the “Interactable” tag since we are now going to set up interaction for this button. Then disable the checkbox next to the “Mesh Renderer” component. This stops the button from being rendered, making it essentially invisible. 

![image12.png](/Instructions/image12.png)

Since moving an invisible object doesn’t make much sense, let’s instead have this object display a message on the screen. Add a new script to AR Button 2 and name it “ARButton2”. Open the script and add the following: 

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ARButton2 : MonoBehaviour, OnTouch3D
{
    public Text messageText;

    public void OnTouch()
    {
        messageText.gameObject.SetActive(true);
        messageText.text = "Button2Pressed";
    }
}
```

We will then need to create the Text object for this to reference. Add a Text object to the scene GameObject->UI->Text and rename it “Message Text”. We will leave this object centered, but be sure the (X,Y,Z) positions are all set to 0. Change the width and height to 250 and 60 respectively and change the font size to 28.  As with our Game Board we are going to set this to inactive to start. 

![image13.png](/Instructions/image13.png)

To make this a proper message, let’s also add a script to make the text go inactive again after a specified time. Add a new script component and name it DisappearingText. Open it and add the following:

```C++
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DisappearingText : MonoBehaviour
{
    // displayTime will be set to a public float
    // so that you can easily change it in the Unity UI
    public float displayTime = 1;

    private float timeRemaining;

    // Start is called before the first frame update
    void Start()
    {
        timeRemaining = displayTime;
    }

    // Update is called once per frame
    // but only while the object is active
    void Update()
    {
        timeRemaining -= Time.deltaTime;
        if (timeRemaining < 0)
        {
            timeRemaining = displayTime;
            this.gameObject.SetActive(false);
        }
    }
}
```

Next let’s add a panel to make the text more visible against the background scene. Right Click on your Message Text object in the scene hierarchy and select UI->Panel. This should automatically place a panel under the Message Text object. 

![image5.png](/Instructions/image5.png)

Select the panel and change the “Source Image” in the inspector to UISprite. 

![image1.png](/Instructions/image1.png)

Finally, select your AR Button 2 again and drag your Message Text object into the Message Text section of your AR Button 2 script. 

![image2.png](/Instructions/image2.png)

Build and run your game. After placing your game board you should now be able to click on the location where the button would be (if it were visible) and a message should appear on screen. 




## Using your game from before:

Make sure to complete the steps in the setting up your project section (of this Readme)

Delete "Miniworld_FloorPlan229_physics" gameobject from the scene (we no longer require this simulated environment), and delete the folders - "ARFoundationSim" and ":"SimEnvironments".  

Delete the "ARCamera" GameObject and Follow the steps outlined in this readme. When making changes to your code, you may use 'TO_COMMENT' and 'TO_ADD' tags to see what functions need to be changed.

The image below indicated the stuffs that need to be deleted from your project.

![image16.PNG](/Instructions/image16.PNG)


## (NEW) Adding physically tracked virtual objects into the scene:
In this section, we will learn how to track images in a physical scene and use that for controlling virtual elements in the scene. To do this, we will leverage ARFoundation's image tracking ability. 

First we will setup the image that we need AR kit to track. 

We will add all image(s) we want our AR applicaiton to track. We will first add images to the project. In the project window right click -> Import new assets. Now add all the images that you want to track.

In the project window, right click -> Create -> XR -> ReferenceImageLibrary

![image17.PNG](/Instructions/image17.PNG)

Now select the Reference Image Library and in the inspector, use 'Add Image' to add images that you would like to track. Select the texture and choose the image(s) that you imported.

Next check 'specify size' and type in the physical size of your printed image. Note that, aspect ratio is determined based on the texture dimensions. So make sure to type in the physical measurements on the printout that corresponds to the image you uploaded. Note: cropping any unneccssary white borders on the uploaded image could help you get this right.

![image18.PNG](/Instructions/image18.PNG)

Now select the AR session Origin Game Object, and through the inspector, Add 'AR Tracked Image Manager' component. Drag your ReferenceImageLibrary from the project window to `SerializedLibrary' variable of the manager. This ensures that the AR application tracks the images that we have in the referece image library. Make sure to set the maximum number of moving components as the number of images that you want to track.

![image19.PNG](/Instructions/image19.PNG)

Now we will need a script that will help us figure out which images are currently being tracked in the scene. Below is a template script that needs to be attached to the 'AR session Origin' game object.  Note that this is only a template script, and we use it to attach cubes to every uniquely tracked images. You  will certainly need to modify this to suit the needs of your game. 

```C++
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.XR.ARSubsystems;
using UnityEngine.XR.ARFoundation;

// This component listens for images detected by the <c>XRImageTrackingSubsystem</c>
// and overlays a cube on each of the tracked image
// Note that this code assumes that all tracked images are unique and named differently

[RequireComponent(typeof(ARTrackedImageManager))]
public class TrackedImageInfoManager : MonoBehaviour
{
    

    ARTrackedImageManager m_TrackedImageManager;
    Dictionary<string,GameObject> gameobjectDict = new Dictionary<string,GameObject>();
    private TrackingState prevTrackingState = TrackingState.None; //For newer versions of ARFoundation
    void Awake()
    {
        //This gets a reference to the AR Tracked Image Manager attached to the 'AR session Origin' gameobject
        m_TrackedImageManager = GetComponent<ARTrackedImageManager>();
    }

    void OnEnable()
    {
        m_TrackedImageManager.trackedImagesChanged += OnTrackedImagesChanged;
    }

    void OnDisable()
    {
        m_TrackedImageManager.trackedImagesChanged -= OnTrackedImagesChanged;
    }


    void OnTrackedImagesChanged(ARTrackedImagesChangedEventArgs eventArgs)
    {
        
        //eventArgs.added contains all the newly trackedImages that were found this frame
        foreach (var trackedImage in eventArgs.added)
        {

            //Write code here to deploy stuff whenever a new image is found in the tracking
            //e.g. Create a new virtual object and/or attach it to the tracked image
            //trackedImage.referenceImage.name -> Name of the tracked image
            //trackedImage.transform.position -> Position of the tracked image in the real world 
            //trackedImage.transform.rotation -> Rotation of the tracked image in the real world 

            //For example, here when we find a new tracked Image, we create a cube, and place it at the location of the tracked image. We add the gameobject to a dictionary, using the name of the image as a key.
            GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
            cube.transform.localScale = new Vector3(0.1f,0.1f,0.1f);
            cube.transform.position = trackedImage.transform.position;
            cube.transform.rotation = trackedImage.transform.rotation;
            gameobjectDict.Add(trackedImage.referenceImage.name,cube);

            prevTrackingState = trackedImage.trackingState; //For newer versions of ARFoundation
        }

        //eventArgs.removed contains all the trackedImages that were not found by the AR camera, either because it was removed from the camera's views or becuase the camera could not detect it.
        foreach (var trackedImage in eventArgs.removed)
        {
            //If we loose tracking of the image, we destroy the conrresponding cube and remove the image name's entry from the dictionary
            Destroy(gameobjectDict[trackedImage.referenceImage.name]);
            gameobjectDict.Remove(trackedImage.referenceImage.name);
           
            prevTrackingState = trackedImage.trackingState; //For newer versions of ARFoundation
        }
   
        //eventArgs.updated contains all the trackedImages which are currently being tracked, but its position and/or rotation changed
        foreach (var trackedImage in eventArgs.updated)
        {
            //trackedImage.transform.position -> Updated Position of the tracked image in the real world 
            //trackedImage.transform.rotation -> Updated Rotation of the tracked image in the real world 

            //if tracked image moves, we move the corresponding gameobject to match it's position.
            gameobjectDict[trackedImage.referenceImage.name].transform.position = trackedImage.transform.position;
            gameobjectDict[trackedImage.referenceImage.name].transform.rotation = trackedImage.transform.rotation;
  
            //Add the Below if else code  block For newer versions of ARFoundation
            if(prevTrackingState == TrackingState.Tracking && trackedImage.trackingState!= prevTrackingState)
            {
                //We lost and then regained tracking so we add cube again
                
                GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
               cube.transform.localScale = new Vector3(0.1f,0.1f,0.1f);
               cube.transform.position = trackedImage.transform.position;
               cube.transform.rotation = trackedImage.transform.rotation;
               gameobjectDict.Add(trackedImage.referenceImage.name,cube);

            }
            else if(prevTrackingState != TrackingState.Tracking && trackedImage.trackingState != prevTrackingState)
            {
                //We lost tracking so we remove the cube
                Destroy(gameobjectDict[trackedImage.referenceImage.name]);
                gameobjectDict.Remove(trackedImage.referenceImage.name);
            }
   
            prevTrackingState = trackedImage.trackingState; //For newer versions of ARFoundation
            
        }
    }
}

```

That's it! Try to think of ways in which these tracked images can interact with the different elements in your game. A couple of examples, include:

- In a pokemon game you may track images of pokemon (cards). When a player pushes a pokemon card into the view of the camera, the corresponding pokemon would be spawned into the scene over the card. E.g. - https://www.youtube.com/watch?v=3ziNTo-BCT0&ab_channel=Fyone . If required, this may also activate the fire arena (P.S. Charizard is a fire type pokemon). 

- In a battle game, you may track images of a sword. When a player brings in the image of a sword into the scene, a virtual sword will spawn at the location of the image. The player can use the virtual sword to attack virtual enemies or virtual objects in the scene. 

VERY IMPORTANT: In all these interactions, design your on-screen and tracked object interactions keeping in mind that the player will have to use one hand to hold the mobile and see the scene. So, only one hand might be free to interact using tracked objects. It important to ensure that these interactions are feasible, ergonomic and enjoyable.

Credit: Thank you to Bala Kumaravel and the CS294-137 instructors for allowing us to borrow some assets and instructions for this workshop.
