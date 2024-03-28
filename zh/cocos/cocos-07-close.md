## Game > GameAnvil > CocosCreator Development Guide > GameAnvil Connector disconnect

## Close GameAnvil Connector

It is recommended that you call the Connector.CloseSocket() function to end the connection before ending the gameplay. If you do not end, the server may not recognize the client's shutdown, which may lead to unnecessary behavior. It's also a good idea to make the components that manage the Connector destroy only at the end of the game and make to call Connector.CloseSocket() in OnDestroy(). 

