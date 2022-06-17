In this lab you will introduce service invocation using Dapr. The service invocation block is used by the frontend service when calling the catalog and ordering service. You will implement two different ways to interact with other services via the Dapr sidecars.

## Inspecting HTTP client implementation
Open the `Program.cs` file and locate the section that adds two HTTP client registrations for the `HttpClientFactory`. 

```C#
builder.Services.AddHttpClient<IEventCatalogService, EventCatalogService>(
    (provider, client) =>{
        client.BaseAddress = new Uri(provider.GetService<IConfiguration>()?["ApiConfigs:EventCatalog:Uri"] ?? throw new InvalidOperationException("Missing config"));
    });

builder.Services.AddHttpClient<IOrderSubmissionService, HttpOrderSubmissionService>(
    (provider, client) => {
        client.BaseAddress = new Uri(provider.GetService<IConfiguration>()?["ApiConfigs:Ordering:Uri"] ?? throw new InvalidOperationException("Missing config"));
    });
```

As you can see, it uses the `ApiConfig` section to retrieve the URLs of the two endpoints for catalog and ordering. This implies that we need to know and configure each service endpoint's URL.

## Add Dapr support
Add the Dapr client SDK NuGet package to the frontend project. 
In Visual Studio 2022 you can right-click the frontend project and select `Manage NuGet packages`. Since the project is an ASP.NET Core project, search for the package `Dapr.AspNetCore`. Accept the license agreements and verify that the package has been added.

![image](https://user-images.githubusercontent.com/5504642/173668283-e02fa3e0-56d2-40c1-9a02-ef674394ee47.png)

In Visual Studio Code or GitHub Codespaces run the following commands from a terminal window:
```cmd
cd frontend
dotnet add package Dapr.AspNetCore
cd ..
```
Now that we have the Dapr client as a means to interact with the sidecars, you can change the `HttpClient` used to call into the catalog service. Instead of calling the service directly, we can use the `DaprClient` to construct a special `HttpClient` that is preconfigured to use the sidecar.

## Replace use of HTTPClient
Replace the call to `AddHttpClient` for the event catalog with the following code:

```C#
builder.Services.AddSingleton<IEventCatalogService>(sc => 
    new EventCatalogService(DaprClient.CreateInvokeHttpClient("catalog")));
```
Make sure you also add the namespace `Dapr.Client` at the top of the file:

```C#
using Dapr.Client;
```

Notice how the Dapr client will do the discovery of the catalog service based on the Dapr application ID `catalog`. This line will make sure that every time a `IEventCatalogService` is injected, the concrete implementation will be an `EventCatalogService` with an `HttpClient` configured by Dapr.

## Replace calls to other services
For the second method of calling a different service we will use the `DaprClient`. To be able to use this client, we need to register it in the dependency injection system at startup.

Open `Program.cs` and add the `DaprClient` to the services, right after the call to `AddControllersWithViews`.

```C#
builder.Services.AddControllersWithViews();
builder.Services.AddDaprClient();
```

Remove the call to `AddHttpClient` for the `IOrderSubmissionService` and add the following line to do regular injection of `HttpOrderSubmissionService`.

```C#
builder.Services.AddTransient<IOrderSubmissionService, HttpOrderSubmissionService>();
```

Next, open the `frontend/Services/Ordering/HttpOrderSubmissionService.cs` file and locate the constructor.

Change the injected client argument type from `HttpClient` to `DaprClient`. Also change the type of the corresponding field. Again, add the namespace for `Dapr.Client`.

Inside the `SubmitOrder` method you can find the call to `PostAsJsonAsync` from the original `HttpClient`. Now that the client is a `DaprClient` object we can change the invocation of the ordering service to be 

```C#
await orderingClient.InvokeMethodAsync<OrderForCreation>("ordering", "order", order);  
```
Also, remove the code that validated the results from the invocation to the original `HttpClient`, since we don't need this anymore.

Run the application again and place an order to see if the new invocation via the sidecar works. Verify the logs to see whether the order was processed.