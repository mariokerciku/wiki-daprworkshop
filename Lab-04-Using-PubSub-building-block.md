In this lab you will add support for the publish/subscribe pattern using Dapr sidecars.
The PubSub building block will be used to publish an event from the frontend when an order is placed. Additionally, the ordering service is going to subscribe to the published message.

## Adding pubsub component
In this exercise we will use the Redis backend to serve as the pubsub broker. Copy the `pubsub.yaml` from the lab-resources folder into the `components\docker-compose` folder. Open the `Dapr/docker-compose/pubsub.yaml` file to inspect the definition of the pubsub component. It uses the 'redis' service defined earlier in the `docker-compose.yml` file. 

You can start up the composition again and inspect the logs for the Dapr sidecar containers. It should indicate that the pubsub component was loaded successfully. This means it is available for you to use in your code.

```
time="2023-03-19T20:45:50.056746385Z" level=info msg="component loaded. name: pubsub, type: pubsub.redis/v1" app_id=frontend instance=d8df7db28771 scope=dapr.runtime type=log ver=1.11.3
```

## Publish events using Dapr
Next, you are going to change the publishing side in the **frontend** service. For this, a new implementation for `IOrderSubmissionService` is created. Copy and paste the `HttpOrderSubmissionService.cs` file and rename it to `PubSubOrderSubmissionService.cs`.

Open the file and rename the class to be `PubSubOrderSubmissionService`, including the constructor.
Locate the call to `InvokeMethodAsync` and replace that line with:

```C#
await orderingClient.PublishEventAsync("pubsub", "orders", order);
```

Here, `pubsub` is the name of the Dapr component, and `orders` is the name of the topic for the publication. The same topic name will be used in the subscriber in a moment.

To use the new implementation of `IOrderSubmissionService` we need to register it with the DI system instead of the `HttpOrderSubmissionService`. Replace the previous registration of the `IOrderSubmissionService` to be:

```C#
builder.Services.AddTransient<IOrderSubmissionService, PubSubOrderSubmissionService>();
```

## Subscribing to events using Dapr
In the project for the **ordering** service you also need to add the NuGet package for `Dapr.AspNetCore`. 

In Visual Studio Code or GitHub Codespaces run the following commands from a terminal window:
```cmd
cd ordering
dotnet add package Dapr.AspNetCore
cd ..
```

Next, you should add middleware in the HTTP pipeline to unwrap the CloudEvents envelope from the incoming message over the topic. 
Add a call to `UseCloudEvents` before adding the routing middleware.

```C#
app.UseCloudEvents(); // Add this line
app.UseRouting();
```

Also, we will add the handler to provide the subscription endpoint `http://<host>/dapr/subscribe` that the sidecar requests at startup to check for the subscriptions it should register.

After the call to `MapControllers`, add a call to `MapSubscribeHandler`.

```C#
app.MapControllers();
app.MapSubscribeHandler(); // Add this line
```

With these preparations, the only thing left to do is add a mapped subscription handler in a controller class. Open the `ordering/Controllers/OrderController` class and find the `Submit` method. Decorate the method with an additional attribute to indicate this is a subscription handler. It should be called whenever the `pubsub` component detects a new message on the `orders` topic. For this, include the `Topic` attribute:

```C#
    [Topic("pubsub", "orders")] // Add this line
    [HttpPost("", Name = "SubmitOrder")]
    public IActionResult Submit(OrderForCreation order)
```

Also, include the `Dapr` namespace at the top of the file:
``` C#
using Dapr;
```

Run your application again, place an order and see whether the order is still being received by the ordering service.

## Finish lab
You are all done. You have Pub/Sub messaging to your project. In the next lab you will continue using a Dapr binding.

Stop running your application. In Visual Studio Code and GitHub CodeSpaces you can stop the composition by pressing Ctrl+C in the terminal window. 

<img src="https://user-images.githubusercontent.com/5504642/173663285-5882128d-08a0-48cc-989a-804047beff89.png" width="400" />

In Visual Studio 2022 you can press Shift+F5 or click on the red square stop icon in the Debug toolbar.