## Game > GameAnvil > Unity Basic Development Guide > Connectivity Check

## NetworkChecker

It is responsible for recognizing when the internet connection is lost (due to LTE, Wifi switching, etc.) and disconnecting from the server.
NetworkChecker must exist in the scene where you want to use this feature.

### Create a NetworkChecker

Create a GameObject and add the NetworkChecker component.
You can add it as a component by choosing **Add Component > GameAnvil > NetworkChecker**.

Alternatively, you can create it directly from the Unity Hierarchy by right-clicking and selecting **GameAnvil > NetworkChecker**.
