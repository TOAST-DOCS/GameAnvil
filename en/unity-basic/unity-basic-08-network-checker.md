<!-- pre-align:aligned sig=e626004b1194 -->

<a id="game-gameanvil-unity-basic-development-guide-connectivity-check"></a>
## Game > GameAnvil > Unity Basic Development Guide > Connectivity Check { #game-gameanvil-unity-basic-development-guide-connectivity-check }

<a id="networkchecker"></a>
## NetworkChecker { #networkchecker }

It is responsible for recognizing when the internet connection is lost (due to LTE, Wifi switching, etc.) and disconnecting from the server.
NetworkChecker must exist in the scene where you want to use this feature.

<a id="create-a-networkchecker"></a>
### Create a NetworkChecker { #create-a-networkchecker }

Create a GameObject and add the NetworkChecker component.
You can add it as a component by choosing **Add Component > GameAnvil > NetworkChecker**.

Alternatively, you can create it directly from the Unity Hierarchy by right-clicking and selecting **GameAnvil > NetworkChecker**.
