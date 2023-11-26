In this lab you are going to use the secret store building block in Dapr. It is used to store secrets in a safe way, without a need to know which specific secret store is used and how to retrieve secret from that particular store. 

The catalog uses a fictitious database with a connection string that is read from configuration. So far the connection string was located in appsettings.json and in a readable form.
Now assume that the connection string contains sensitive information such as the username and password of the account used to log on to the database server. This means it should be considered a secret. 
There are several ways to protect the connection string from being read or exposed in the Git repository or in an unprotected configuration store. 

## Adding secret store
Using the secret store building block we can store the secret in a dedicated store. We are going to use a local secrets file for the development setup. Later, you will change this to be more secure stores.

Copy the files `localSecretStore.yaml` and `secrets.json` from `lab-resources` to `components/docker-compose`. Open the `localSecretStore.yaml` file see how it refers to a file `./components/secrets.json`. The nested operator is indicated as a colon ':' so you can use values such as `parent:child:grandchild` to indicate configuration hierarchies, just as in appsettings files.
The contents of the secrets.json file are:

```json
{
  "catalogconnectionstring": "Event Catalog DB Connection string from local secrets file"
}
```

First, let's add the NuGet package for Dapr provided configuration to the Catalog project. The package is called `Dapr.Extensions.Configuration` and can be added through the UI of Visual Studio or by adding

```xml
<PackageReference Include="Dapr.Extensions.Configuration" Version="1.11.0" />
```

to the `<ItemGroup>` element in the `catalog.csproj` file that contains the other NuGet package references. Alternatively, you can run `dotnet add package Dapr.Extensions.Configuration` again from the `catalog` folder:

```cmd
cd catalog
dotnet add package Dapr.Extensions.Configuration
cd ..
```

In the Program.cs file add the following block before the call to `Configure<CatalogOptions>`:

```C#
//Add using
using Dapr.Extensions.Configuration;

[..]

//Add these lines:
builder.WebHost.ConfigureAppConfiguration(config =>
{
    var daprClient = new DaprClientBuilder().Build();
    var secretDescriptors = new List<DaprSecretDescriptor>
    {
        new DaprSecretDescriptor("catalogconnectionstring")
    };
    config.AddDaprSecretStore("secretstore", secretDescriptors, daprClient);
});
builder.Services.Configure<CatalogOptions>(builder.Configuration); 
```

Notice how the call to `AddDaprSecretStore` specifies the name of the building block `secretstore` and uses a list of explicit secret descriptors to fetch the secrets from the store. In this case the list is a single value for `catalogconnectionstring`.

The original value for the event catalog connection string is located in `appsettings.json`, but could also have been placed in the `secrets.json` file for User Secrets. These configuration sources have already been defined through `WebApplication.CreateBuilder`. The call to `ConfigureAppConfiguration` will add an additional configuration source for the secret store and, since it is defined last, its values will override the previous values from `appsettings.json`, `appsettings.<environment>.json` and `<usersecretsfolder>/secrets.json`.

Run the application again to check that the new value from the secrets file of the store is actually used.

Fetch events from the catalog api to trigger a log message. Connect to the forwarded port 5003 in a separate browser tab, using the 'ports' tab in CodeSpaces.
For example:
```
https://cuddly-parakeet-5vdqr15r7dr3pgc6-5003.app.github.dev/event/
```

You should see the output in the log of the catalog service:

```
info: GloboTicket.Catalog.Repositories.EventRepository[0]
      Connection string Event Catalog DB Connection string from local file
```
You can also load or refresh the home page of the GloboTicket website

## Finish lab
You are all done. You have added a secret store to your project. In the next lab you will continue using a Dapr state store building block.

Stop running your application. In Visual Studio Code and GitHub CodeSpaces you can stop the composition by pressing Ctrl+C in the terminal window. 

<img src="https://user-images.githubusercontent.com/5504642/173663285-5882128d-08a0-48cc-989a-804047beff89.png" width="400" />

In Visual Studio 2022 you can press Shift+F5 or click on the red square stop icon in the Debug toolbar.