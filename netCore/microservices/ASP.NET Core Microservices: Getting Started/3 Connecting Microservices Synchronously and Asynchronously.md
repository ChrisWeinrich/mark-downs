# 3 Connecting Microservices Synchronously and Asynchronously

## Adding a Microservice

## Bounded Contexts

Event is in two bounded Contexts => different Properties, not 100% the same

=> the Micro Services/bounded Contexts have to communicate

## Implementing Service-to-service Communication

``` C#
 if (!await _eventRepository.EventExists(basketLineForCreatioEventId)
    {
        var eventFromCatalog = await _eventCatalogService.GetEvent(basketLineForCreation.EventId);
        _eventRepository.AddEvent(eventFromCatalog);
        await _eventRepository.SaveChanges();
    }


public async Task<Event> GetEvent(Guid id)
        {
            var response = await client.GetAsync($"/api/events/{id}");
            return await response.ReadContentAs<Event>();
        }

public static async Task<T> ReadContentAs<T>(this HttpResponseMessage response)
        {
            if (!response.IsSuccessStatusCode)
                throw new ApplicationException($"Something went wrong calling the API: {response.ReasonPhrase}");

            var dataAsString = await response.Content.ReadAsStringAsync().ConfigureAwait(false);

            return JsonSerializer.Deserialize<T>(dataAsString, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        }
```

=> Graph QL if bandwidth is a problem

## Synchronous and asynchronous Communication

Synchronous => Request/Response

Asynchronous 
- Send Message to Queue/storage
- Work can be done in parallel
- Mitigates temporal coupling
- Reliable
- Message is the contract

## Setting up a Microservice to Asynchronous Communication

**Create Worker Service Template** 

Worker Services
- 

```C#
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }


        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureServices((hostContext, services) =>
                {
                    services.AddHostedService<NewOrderWorkerService>();
                });
    }
```

## Receiving Messages with a service Bus

=> Rebus  
=> Azure Storage Queues

``` C#
    public class NewOrderWorkerService : BackgroundService
    {
        private readonly IConfiguration _config;

        public NewOrderWorkerService(IConfiguration config)
        {
            _config = config;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            var storageAccount = CloudStorageAccount.Parse(
                _config["AzureQueues:ConnectionString"]);

            using var activator = new BuiltinHandlerActivator();
            activator.Register(() => new NewOrderHandler());
            Configure.With(activator)
                .Transport(t => t.UseAzureStorageQueues(
                    storageAccount, _config["AzureQueues:QueueName"]))
                .Start();

            await Task.Delay(Timeout.InfiniteTimeSpan, stoppingToken);
        }
    }

    public class PaymentRequestMessage
    {
        public Guid BasketId { get; set; }
    }

```

## sending a message with a Service Bus

``` C#
public async Task<IActionResult> Pay()
{
    var basketId = Request.Cookies.GetCurrentBasketId(settings);
    await bus.Send(new PaymentRequestMessage { BasketId = basketId });
    return View("Thanks");
}
```


