In this lab, you will introduce the final building block from Dapr. We are going to use the state store block.

## Dapr default component definitions
By default Dapr uses the components as defined inside the `.dapr/components` folder. Open this folder as `%USERPROFILE%\.dapr` on Windows or `~/.dapr` on Linux and examine the contents of the components folder that is inside.

There should be two component files there for `pubsub` and `statestore`. Open the `statestore.yaml` file in your favorite editor or viewer. It shows that Redis is used for the state store component.

To leverage the state store we need to be able to interact with the store by means of the Dapr sidecar. By now you know that this is easiest using the Dapr client library. 

Copy the contents of the folder named `DaprStateStore` from the lab-resources inside the `frontend/Services/ShoppingBasket` folder.  