In this lab, you will introduce the final building block from Dapr. We are going to use the state store block.

## Dapr default component definitions
By default Dapr uses the components as defined inside the `.dapr/components` folder. Open this folder as `%USERPROFILE%\.dapr` on Windows or `~/.dapr` on Linux and examine the contents of the components folder that is inside.

There should be two component files there for `pubsub` and `statestore`. Open the `statestore.yaml` file in your favorite editor or viewer. It shows that Redis is used for the state store component.

To leverage the state store we need to be able to interact with the store by means of the Dapr sidecar. By now you know that this is easiest using the Dapr client library. 

## Changing shopping basket implementation
Copy the complete folder named `DaprStateStore` including its contents from the `lab-resources` folder into the `frontend/Services/ShoppingBasket` folder. There should be two files inside with the implementation we need.

The `DaprClientStateStoreShoppingBasket` class is a new implementation of the `IShoppingBasketService` interface. By changing the mapping of `IShoppingBasketService` from `InMemoryShoppingBasket` to `DaprClientStateStoreShoppingBasket` in the dependency injection system of .NET, the injected object of type `IShoppingBasketService` will be the new Dapr based implementation.
Find the line that maps a singleton object of type `InMemoryShoppingBasket`:

```C#
builder.Services.AddSingleton<IShoppingBasketService, InMemoryShoppingBasketService>();
```

and change it to be a mapping for transient lifetime objects of type `DaprClientStateStoreShoppingBasket`:

```C#
builder.Services.AddTransient<IShoppingBasketService, DaprClientStateStoreShoppingBasket>();
```

Inspect the implementation for `DaprClientStateStoreShoppingBasket` and `StateStoreBasket`.

## Adding building block definition
With these changes in place we can start using the state store building block. As before you will also need to copy the component definition from `lab-resources` to the `components/docker-compose` folder. The file is named `statestore.yaml`.

## Check the volume mount for the state store.
We're using a redis cache container for persistance. by default a container does not store it's state after restarting it. So what we can do is add a volume mount to the state store container so the redis cache can store it's data outside of it's container and the data will survive a restart.

At the bottom of the Redis definition in the `docker-compose.override.yaml` add a volume mount to a folder called `data`
```
  redis:
    container_name: "redis"
    image: "redis:7.2-alpine"
    ports:
      - "6379"
    networks:
      - globoticket
    volumes:             # Add this line
      - "./data:/data"   # and this one
```

Now also create a `data` folder in the root of your solution. This is used in the volume mount so redis can store it's files here. It will create a `dump.rdb` file regularly and when stopping the redis cache. 

### Testing the persistence of the state store.

Now you can test your new implementation and see if it is able to store your shopping basket. Try stopping the composition and starting it again to see whether it survives a restart. The state of the shopping basket should be persisted.