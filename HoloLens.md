# Creating the Unity Project

Create a new Unity project and get it ready for MRTK development. For this, first follow the [Initializing your project and first application](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1) (excluding the [Build and deploy the application](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#2-build-and-deploy-the-application) instructions), which includes the following steps:

1. [Create new Unity project](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#create-new-unity-project) and give it a suitable name
2. [Configure the Unity project for Windows Mixed Reality](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#configure-the-unity-project-for-windows-mixed-reality)
3. [Import TextMesh Pro Essential Resources](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#import-textmesh-pro-essential-resources)
4. [Import the Mixed Reality Toolkit](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#import-the-mixed-reality-toolkit)
5. [Configure the Unity project for the Mixed Reality Toolkit](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#configure-the-unity-project-for-the-mixed-reality-toolkit)
6. [Configure the Mixed Reality Toolkit](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch1#configure-the-unity-project-for-the-mixed-reality-toolkit)

Then follow the [How to configure the Mixed Reality Toolkit Profiles](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-base-ch2#how-to-configure-the-mixed-reality-toolkit-profiles-change-spatial-awareness-display-option) (Change Spatial Awareness Display Option) instructions to change the MRTK configuration profile for your scene to the DefaultHoloLens2ConfigurationProfile and change the display options for the spatial awareness mesh to Occlusion.

Then follow [Adding inbuilt Unity packages](https://docs.microsoft.com/en-gb/windows/mixed-reality/mrlearning-asa-ch1#adding-inbuilt-unity-packages) to add the AR Foundation package  required by the Azure Spatial Anchors SDK and download and import the following Unity custom packages in the order they are listed:

1. [AzureSpatialAnchors.unitypackage (version 2.1.1)](https://github.com/Azure/azure-spatial-anchors-samples/releases/download/v2.1.1/AzureSpatialAnchors.unitypackage)
2. [MRTK.HoloLens2.Unity.Tutorials.Assets.GettingStarted.2.3.0.2.unitypackage](https://github.com/microsoft/MixedRealityLearning/releases/download/getting-started-v2.3.0.2/MRTK.HoloLens2.Unity.Tutorials.Assets.GettingStarted.2.3.0.2.unitypackage)
3. [MRTK.HoloLens2.Unity.Tutorials.Assets.AzureSpatialAnchors.2.3.0.0.unitypackage](https://github.com/microsoft/MixedRealityLearning/releases/download/azure-spatial-anchors-v2.3.0.0/MRTK.HoloLens2.Unity.Tutorials.Assets.AzureSpatialAnchors.2.3.0.0.unitypackage)

In the Unity menu, select **Edit > Project Settings...*** to open the Player Settings window. In the Player Settings window, select **Player** and then **Publishing Settings**. In the **Publishing Settings**, scroll down to the **Capabilities** section and enable the **InternetClient**, **Microphone**, and **SpatialPerception** capabilities. Then, enable the **InternetClientServer**, **PrivateNetworkClientServer**, **RemovableStorage**, and **Webcam** capabilities.