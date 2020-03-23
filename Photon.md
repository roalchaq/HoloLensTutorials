# Setting up Photon Unity Networking

To create a multi-user application, first follow the procedure described [in this link](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-sharing(photon)-ch1#setting-up-photon) to import Photon Unity Networking (PUN) into your Unity project. Then, read the following definitions to better understand what we will be doing next:

### Rooms

The Photon Cloud is built with "room-based games" in mind, meaning there is a limited number of players (let's say: less than 10) per match, separated from anyone else. In a room (usually) everyone receives whatever the others send. Outside of a room, players are not able to communicate, so we always want them in rooms as soon as possible.
All rooms have a name as identifier. Unless the room is full or closed, we can join it by name. Conveniently, the Master Server can provide a list of rooms for our app.

### Lobby

The lobby for your application exists on the Master Server to list rooms for your game. In our case, we will not make use of the lobby and simply join a random room if one is available, or create a new room if no existing room can be joined (rooms can have a maximum capacity, and so they can potentially be all full).

## Setting up Photon 

Import the [MRTK.HoloLens2.Unity.Tutorials.Assets.MultiUserCapabilities.2.1.0.1.unitypackage](https://github.com/microsoft/MixedRealityLearning/releases/download/multi-user-capabilities-v2.1.0.1/MRTK.HoloLens2.Unity.Tutorials.Assets.MultiUserCapabilities.2.1.0.1.unitypackage) Unity custom package. Once finished, save the project by clicking File, then Save.

### Launcher Prefab 

This module covers the critical aspects of connecting and joining a room using PUN.

1. Create a new C# script Launcher.
2. Create an empty GameObject in the Hierarchy named Launcher.
3. Attach the C# script Launcher to the GameObject Launcher.
4. Edit the C# script Launcher to have its content as below:

```cs
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

public class Launcher : MonoBehaviourPunCallbacks
{
    #region Private Serializable Fields

    /// <summary>
    /// The maximum number of players per room. When a room is full, it can't be joined by new players, and so new room will be created.
    /// </summary>
    [Tooltip("The maximum number of players per room. When a room is full, it can't be joined by new players, and so new room will be created")]
    [SerializeField]
    private byte maxPlayersPerRoom = 4;

    [SerializeField]
    private GameObject gameManager = null;
    #endregion

    #region Private Fields

    /// <summary>
    /// Keep track of the current process. Since connection is asynchronous and is based on several callbacks from Photon,
    /// we need to keep track of this to properly adjust the behavior when we receive call back by Photon.
    /// Typically this is used for the OnConnectedToMaster() callback.
    /// </summary>
    bool isConnecting;

    #endregion

    #region MonoBehaviour CallBacks

    void Start()
    {
        Connect();

        int randomuserID = UnityEngine.Random.Range(0, 999999);
        PhotonNetwork.AuthValues = new AuthenticationValues();
        PhotonNetwork.AuthValues.UserId = randomuserID.ToString();

        // Setting up the name of the player over the network.
        PhotonNetwork.NickName = PhotonNetwork.AuthValues.UserId;
    }
    #endregion

    #region Private methods

    /// <summary>
    /// Start the connection process.
    /// - If already connected, we attempt joining a random room
    /// - if not yet connected, Connect this application instance to Photon Cloud Network
    /// </summary>
    private void Connect()
    {
        // keep track of the will to join a room, because when we come back from the game we will get a callback that we are connected, so we need to know what to do then
        //isConnecting = PhotonNetwork.ConnectUsingSettings();
        isConnecting = true;

        // we check if we are connected or not, we join if we are , else we initiate the connection to the server.
        if (PhotonNetwork.IsConnected)
        {
            // #Critical we need at this point to attempt joining a Random Room. If it fails, we'll get notified in OnJoinRandomFailed() and we'll create one.
            PhotonNetwork.JoinRandomRoom();
        }
        else
        {
            // #Critical, we must first and foremost connect to Photon Online Server.
            PhotonNetwork.ConnectUsingSettings();
        }
    }
    #endregion

    #region MonoBehaviourPunCallbacks Callbacks

    public override void OnConnectedToMaster()
    {
        Debug.Log("OnConnectedToMaster - Successful");
        // we don't want to do anything if we are not attempting to join a room.
        // this case where isConnecting is false is typically when you lost or quit the game, when this level is loaded, OnConnectedToMaster will be called, in that case
        // we don't want to do anything.
        if (isConnecting)
        {
            // #Critical: The first we try to do is to join a potential existing room. If there is, good, else, we'll be called back with OnJoinRandomFailed()
            PhotonNetwork.JoinRandomRoom();
            isConnecting = false;
        }
    }

    public override void OnDisconnected(DisconnectCause cause)
    {
        Debug.LogWarningFormat("PUN Basics Tutorial/Launcher: OnDisconnected() was called by PUN with reason {0}", cause);
    }

    public override void OnJoinRandomFailed(short returnCode, string message)
    {
        Debug.Log("Launcher: OnJoinRandomFailed() was called by PUN. No random room available, so we create one.\nCalling: PhotonNetwork.CreateRoom");

        // #Critical: we failed to join a random room, maybe none exists or they are all full. No worries, we create a new room.
        PhotonNetwork.CreateRoom(null, new RoomOptions { MaxPlayers = maxPlayersPerRoom });
    }

    public override void OnJoinedRoom()
    {
        Debug.Log("Launcher: OnJoinedRoom() called by PUN. Now this client is in a room.");
        gameManager.SetActive(true);
    }
    #endregion
}
```

### Player Prefab

To instantiate our "Player" prefab  when we've just entered the room. We can rely on the GameManager Script *Start()* method which will indicated we've loaded the scene (next section), which means by our design that we are in a room. An important rule to know about PUN is that a Prefab that should get instantiated over the network, has to be inside a folder named **exactly** Resources.

1. Create a new c# script *PlayerManager*
2. Create an empty GameObject in the Scene, name it 'Player'
3. Drop the *PlayerManager* script onto the GameObject 'Player'
4. Next, we create spheres to represent each person that joins a shared experience. Right-click the 'Player' object you just created, scroll-down to "3D Object and click Sphere. This will create a sphere game object as a child of the 'Player' object.
5. Scale the sphere down to x=0.06, y=0.06, ad z=0.06.
6. In your "Project Browser", create a folder named exactly "Resources" somewhere, typically it's suggested that you organized your content, so could have something like "Resources\Prefabs" 
7. Turn *Player* into a prefab by dragging it from the scene Hierarchy to the "Resources\Prefabs" folder, it will turn blue in the Hierarchy.
8. Remove the 'Player' object from the Hierarchy

We need to have a **PhotonView component** attached to our *Player* prefab. A **PhotonView** is what connects together the various instances on each computers and define what components to observe and how to observe these components. 

1. Add a *PhotonView* Component to the 'Player' prefab
2. Set the Observe Option to Unreliable On Change
3. Notice *PhotonView* warns you that you need to observe something for this to have any effects. You can ignore this for now, as those observed components will be setup later in the tutorial.

The obvious feature we want to synchronize is the Position and Rotation of the character so that when a player is moving around, the character behave in a similar way on other players' instances of the game. To make this common task easy, we are going to use a **PhotonTransformView** component.

1. Add a *PhotonTransformView* to 'Player' Prefab
2. Drag the *PhotonTransformView* from its header title onto the first observable component entry on the *PhotonView* component
3. Now, check Synchronize Position in *PhotonTransformView*
4. Check Synchronize Rotation
5. Edit *PlayerManager* Script
6. Replace with the following

```cs
using UnityEngine;
using Photon.Pun;

/// <summary>
/// Player manager.
/// </summary>
public class PlayerManager : MonoBehaviourPunCallbacks
{
    #region Public fields

    [Tooltip("The local player instance. Use this to know if the local player is represented in the Scene")]
    public static GameObject LocalPlayerInstance;

    #endregion

    #region MonoBehaviour CallBacks

    void Awake()
    {
        // #Important
        // used in GameManager.cs: we keep track of the localPlayer instance to prevent instantiation when levels are synchronized
        if (photonView.IsMine)
        {
            PlayerManager.LocalPlayerInstance = this.gameObject;
        }
        // #Critical
        // we flag as don't destroy on load so that instance survives level synchronization, thus giving a seamless experience when levels load.
        DontDestroyOnLoad(this.gameObject);
    }
    #endregion
}
```

### Game Manager Prefab

In all cases, the minimum requirement for a User Interface is to be able to quit the room. Let's start by creating what we'll call the *Game Manager prefab*, and the first task it will handle quitting the room the local Player is currently in.

1. Create a new c# script GameManager
2. Create an empty GameObject in the Scene, name it Game Manager
3. Drop the GameManager script onto the GameObject Game Manager
4. Turn Game Manager into a prefab by dragging it from the scene Hierarchy to the Assets Browser, it will turn blue in the Hierarchy.
5. Edit GameManager Script
6. Replace with the following

```cs
using System.IO;
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

public class GameManager : MonoBehaviourPunCallbacks
{
    #region MonoBehaviour Callbacks

    void Start()
    {
        if (PlayerManager.LocalPlayerInstance == null)
        {
            // we're in a room. spawn a character for the local player. it gets synced by using PhotonNetwork.Instantiate
            GameObject agent = PhotonNetwork.Instantiate(Path.Combine("Prefabs", "Player"), Vector3.zero, Quaternion.identity, 0);
            agent.transform.parent = Camera.main.transform;
        }
        else
        {
            Debug.LogFormat("Ignoring scene load for {0}", SceneManagerHelper.ActiveSceneName);
        }
    }
    #endregion

    #region Photon Callbacks

    public override void OnPlayerEnteredRoom(Player other)
    {
        Debug.LogFormat("OnPlayerEnteredRoom() {0}", other.NickName); // not seen if you're the player connecting

        if (PhotonNetwork.IsMasterClient)
        {
            Debug.LogFormat("OnPlayerEnteredRoom IsMasterClient {0}", PhotonNetwork.IsMasterClient); // called before OnPlayerLeftRoom
        }
    }

    public override void OnPlayerLeftRoom(Player other)
    {
        Debug.LogFormat("OnPlayerLeftRoom() {0}", other.NickName); // seen when other disconnects

        if (PhotonNetwork.IsMasterClient)
        {
            Debug.LogFormat("OnPlayerLeftRoom IsMasterClient {0}", PhotonNetwork.IsMasterClient); // called before OnPlayerLeftRoom
        }
    }
    #endregion
}
```
Drag and drop the *Game Manager* GameObject to the 'Game Manager' slot on the Launcher class (part of the 'Launcher' GameObject in the Hierarchy). Then, disable the 'Game Manager' GameObject from the Hierarchy (using the checkbox next to the GameObject name in the Inspector panel).

Once all the steps above are complete follow the [Build and deploy the application](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#2-build-and-deploy-the-application) instructions. When ready, press the Play button and connect your HoloLens 2. You should see a sphere moving around as you move your player using the keyboard arrows in Game view. This will be shown for any user that joins your Unity project!

## Sharing object movements with multiple users

Now we need to share the movements of objects so that all participants of a shared session can collaborate and view each others' interactions.

1. In the Project window, navigate to Assets > MRTK.Tutorials.AzureSpatialAnchors > Prefabs folder.
2. Drag the 'ParentAnchor' prefab into the Hierarchy window as a child of 'MixedRealityPlayspace' game object to add it to the scene.
3. With the 'ParentAnchor' object selected in your Hierarchy, click Add Component and search for the **TableAnchor** script. Select it and add it to the object.
4. Open the **PlayerManager** script.
5. Insert the following code just below the Awake() method:

```cs
    /// <summary>
    /// MonoBehaviour method called on GameObject by Unity during initialization phase.
    /// </summary>
    void Start()
    {
        // As this GameObject is not part of the scene from the begining, we don't use fixed references (public GameObject TableAnchor = null)
        // We find the GameObject needed using FindObjectOfType instead
        // Please note that FindObjectOfType is very slow. It is not recommended to use this function every frame. In most cases you can use the singleton pattern instead.
        this.transform.parent = FindObjectOfType<TableAnchor>().transform;
    }
```

6. Save the **PlayerManager** script.
7. Open the **Game Manager** script.
8. Add the following method at the end of the class, within a region *Private Methods* for clarity.

```cs
    #region Private Methods

    private void CreateInteractableObjects()
    {
        PhotonNetwork.Instantiate(Path.Combine("Prefabs", "Rocket Launcher_Complete Variant"), Vector3.zero, Quaternion.identity);
    }
    #endregion
```

9. In the *Start()* method, insert at the very end

```cs
    if (PhotonNetwork.IsMasterClient)
    {
        CreateInteractableObjects();
    }
```
10. Save the **Game Manager** script.
11. Open the **TableAnchorAsParent** script attached to 'Rocket Launcher_Complete Variant' prefab.
12. Replace with the following:

```cs
using UnityEngine;

public class TableAnchorAsParent : MonoBehaviour
{
    void Start()
    {
        if (TableAnchor.instance != null)
        {
            transform.parent = TableAnchor.instance.transform;
            transform.localPosition = new Vector3(0.0f, 0.45f, 3.75f);
            transform.localRotation = Quaternion.Euler(0, 45, 0);
            transform.localScale = new Vector3(10, 10, 10);
        }
    }
}
```

Once this is complete, look around to find the lunar module. After this, all users that join your Unity project can move the components of the lunar launcher around. All movements are synchronized so that each user can see each others' interactions. These concepts serve as the fundamental building blocks for full-featured, shared collaboration experiences.

Although all users are connected as part of a shared experience and can see the relative movements of objects, the application is still unable to accurately align avatars and objects so that local users were not able see each other and objects in the same place within the physical world. In order to anchor a local shared experiences, every device requires a common understanding of the physical environment. We'll achieve this by using Azure Spatial Anchors (ASA).