In this lab two binding components are introduced. The ordering service will use binding to send an email. For this you will introduce the SMTP binding building block. The catalog service is going to use a cron binding to schedule tickets on sale.

## SMTP binding for sending emails
The implementation for `SendEmailForOrder` inside the `ordering/Services/EmailSender.cs` currently only contains two logging calls. You will change this to use an SMTP binding that sends the email to confirm the order.

Since bindings are used and called via the `DaprClient`, we need to register the `DaprClient` in the dependency injection system at startup. Add a call to `AddDaprClient` in the `Program.cs` class.

```C#
builder.Services.AddDaprClient();
var app = builder.Build();
```

Next, inject the `DaprClient` instance through the `EmailSender` constructor.

```C#
    private readonly ILogger<EmailSender> logger;
    private readonly DaprClient daprClient;

    public EmailSender(ILogger<EmailSender> logger, DaprClient daprClient)
    {
        this.logger = logger;
        this.daprClient = daprClient;
    }
```

We can now use the `DaprClient` object inside the `SendEmailForOrder` method. Remove the call to `logger.LogWarning` and add the following fragment:

```C#
logger.LogInformation($"Sending email");
var metadata = new Dictionary<string, string>
{
  ["emailFrom"] = "noreply@globoticket.shop",
  ["emailTo"] = order.CustomerDetails.Email,
  ["subject"] = $"Thank you for your order"
};
var body = $"<h2>Your order has been received</h2>"
  + "<p>Your tickets are on the way!</p>";
await daprClient.InvokeBindingAsync("sendmail", "create", body, metadata);      
```

The last statement is the actual call to invoke the binding. It uses the `sendmail` component and invokes the `create` action. The actual actions and arguments can vary per binding. The SMTP binding offers a single `create` method with two arguments for the email body and the metadata.

Copy the `email.yaml` file from `lab-resources` to the `components/docker-compose` folder. Open and inspect the file. Notice how it specifies the `binding.smtp` component with a name `sendmail`. The scope is limited to the ordering service, so it is only available for that service.

The SMTP server implementation for the development environment on your laptop is provided by another Docker container. Add the SMTP container based on the `maildev/maildev` image to the `docker-compose.override.yml` file.

```yaml
  maildev:
    container_name: "smtpserver"
    image: "maildev/maildev:2.0.5" # https://hub.docker.com/r/maildev/maildev
    ports:
      - "1025:1025"
      - "1080:1080"
    networks:
      - globoticket
```

Notice how the container is using port 1025 and 1080 inside the container, and is exposed via the same ports outside of the composition. 

Start the orchestration for the Globoticket solution, with a rebuild of the containers. You need to do this whenever you change code or configuration or add files to any of the .NET projects.

```cmd
docker-compose up --build
```

Open the URL for the maildev UI at http://localhost:1080 and see that there are no current emails listed.

![image](https://user-images.githubusercontent.com/5504642/173679790-0849cf86-3fbd-44d8-9226-896c259cbd3f.png)

Place an order and verify that the mail arrives at the SMTP mail server:

![image](https://user-images.githubusercontent.com/5504642/173679843-9366ae3c-e55d-47b4-8187-36f900dbbee1.png)

## CRON binding for scheduling
The catalog service has a scheduled task to calculate a special offer ticket price at an interval of every 5 minutes. You can easily create a binding that uses a cron job schedule to execute a method in a controller. Whenever the cron job trigger based on the elapsed time, the Dapr cron binding will make a call to the container it accompanies on a well-known endpoint based on the name of the cron component.

Copy the `cron.yaml` file from `lab-resources` to `components/docker-compose`. Open the file to inspect the `bindings.cron` type component is defined in it. The schedule is defined at `"@every 5m"` and scoped to the catalog service.
Whenever the period of the schedule is passed, the sidecar will issue an HTTP POST request to the `/scheduled` endpoint of the service it is connected to. In this case it is only the catalog sidecar that has the cron binding, so an endpoint of the service.

To create a simple `/schedule` endpoint we can use a Minimal Web API through a mapped POST request.
In the `Program.cs` find the call to `UseAuthorization` and add a call to `MapPost`:

```C#
app.UseAuthorization();
app.MapPost("scheduled", (ILoggerFactory factory, IEventRepository repository) => 
{
    factory.CreateLogger("GloboTicket.Catalog.Scheduler")
        .LogInformation("Scheduled endpoint called");
    repository.UpdateSpecialOffer();
});
```

Start the application and check the output from the logger in the output window. It should show a logged call to the scheduled endpoint.

![image](https://user-images.githubusercontent.com/5504642/173680430-486bc8e9-93cb-4ccc-a643-53817fd8e431.png)
