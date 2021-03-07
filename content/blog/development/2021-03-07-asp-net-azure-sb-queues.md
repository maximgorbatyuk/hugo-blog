---
layout: post
title: A little life hack when you work with Azure Service Bus and ASP.NET Core
category: development
tags: [.net, tutorial, azure, asp.net, service bus]
date: "2021-03-07"
---

If you work with Azure infrastructure and have to integrate message queues. It sounds quite simple: just create Azure Resource, write some code and then be happy! But what would you say if the resources are limited? What will you do if there are several teammates in your team, and all of you have to debug queues at the same time?

Well, I know a minor life hack for my teams. I create an InMemory Message queue engine for local development and use Azure Service Bus (or any other external MQ engine) only for remote environments. This solution allows me to not think about paid resources or concurrency access to the single development queue.

Developers just create business logic and do not care about Azure Access or availability. I think the InMemory engine should not become an issue. Most of the business tasks do not depend on the technical implementation of the queue engine. My opinion that they should not do it at all. When you have to develop a technical algorithm that uses, for example, some Kafka or RabbitMQ features, you will debug it using external resources. But in my opinion, business logic should not depend on either RabbitMQ or Kafka, or Azure Service Bus. When you write unites, you do the same, aren’t you? Therefore the logic can use the InMemory solution during the local development.

So, let me show my solution. If you meet a similar task, the solution could be helpful for you. As an example, I will use an email distribution service (EDS) that accepts emails via Queues and then sends them. My apps publish email content, my EDS consumes it and sends using the SMTP server.

Therefore, we need to develop the following items:

1. Settings for our application
2. Queue message publisher
3. Queue consumer.

## Using InMemory Queues engine

### InMemory Setup

I will use the [MassTransit](https://masstransit-project.com) library to make the solution simpler. Here is a code that sets the MassTransit:

```csharp
// IServiceCollection services;

services.AddMassTransit(x =>
{
    x.AddConsumer<MassTransitEmailSendConsumer>();
    x.UsingInMemory((context, cfg) =>
    {
        cfg.TransportConcurrencyLimit = 100;
        cfg.ConfigureEndpoints(context);
        cfg.ReceiveEndpoint(_configuration.EmailMessageTopic.ToString(), e =>
        {
            e.ConfigureConsumer<MassTransitEmailSendConsumer>(context);
        });
    });
});

services.AddMassTransitHostedService();
services.AddScoped<IMessageBroker, InMemoryBrokerPublisher>();
```

Here I use some config values. The class represents MQ settings and is used by both queues: InMemory and Azure Service Bus.

```csharp
using Microsoft.Extensions.Configuration;

namespace YourNamespace
{
    public class MessageBrokerSettings
    {
        public NonNullableString Connection { get; }

        public NonNullableString EmailMessageTopic { get; }

        public NonNullableString HealthCheckConnection { get; }

        public NonNullableString HealthCheckTopic { get; }

        public MessageBrokerSettings(IConfiguration configuration)
        {
            var section = configuration.GetSection("Azure").GetSection("ServiceBus");
            Connection = new NonNullableString(section[nameof(Connection)]);
            EmailMessageTopic = new NonNullableString(section[nameof(EmailMessageTopic)]);
            HealthCheckConnection = new NonNullableString(section[nameof(HealthCheckConnection)]);
            HealthCheckTopic = new NonNullableString(section[nameof(HealthCheckTopic)]);
        }
    }
}
```

`NonNullableString` is a special class that makes me sure that the value inside will never be null. Some kind of ValueObject from DDD, you know. When I invoke `.ToString()` method, the class returns me a value of the config. Otherwise, it will throw an exception. The code of the class you may see at my GitHub gist: [NonNullableString.cs](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-nonnullablestring-cs).

### InMemory Publisher

Now we have created a publisher and consumer. The email publisher will use `IPublishEnpoint` that is given us by MassTransit library:

```csharp
using System.Threading.Tasks;
using MassTransit;
using Microsoft.Extensions.Logging;

namespace YourNamespace
{
    public class InMemoryBrokerPublisher : BrokerPublisherBase
    {
        private readonly IPublishEndpoint _publish;

        public InMemoryBrokerPublisher(IPublishEndpoint publish, ILogger<InMemoryBrokerPublisher> logger)
            : base(logger)
        {
            _publish = publish;
        }

        protected override Task PublishInternalAsync<T>(string topicName, T message)
        {
            return _publish.Publish(message);
        }
    }
}
```

The [BrokerPublisherBase](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-brokerpublisherbase-cs) is a base class and does not depend on queue implementation. The class is inherited by both queue-related publishers as well. It implements a simple IMessageBroker.

```csharp
using System.Threading.Tasks;

namespace YourNamespace
{
    public interface IMessageBroker
    {
        Task PublishAsync<T>(string topicName, T message)
            where T : class;
    }
}
```

This interface gives the other business logic an endpoint to publish any message.

### InMemory Consumer

We will use MassTransit’s ConsumerBase interface for InMemory consumers. Here is a content of the `MassTransitEmailSendConsumer`:

```csharp
using System.Threading.Tasks;
using MassTransit;
using Microsoft.Extensions.Logging;

namespace YourNamespace
{
    public class MassTransitEmailSendConsumer : ConsumerBase<EmailMessage>
    {
        private readonly IEmail _email;

        protected override async Task ConsumeAsync(ConsumeContext<EmailMessage> context)
        {
            await _email.SendAsync(context.Message);
            Logger.LogDebug(“Email sent”);
        }

        public MassTransitEmailSendConsumer(ILogger<MassTransitEmailSendConsumer> logger, IEmail email)
            : base(logger)
        {
            _email = email;
        }
    }
}
```

`IEmail` is my business logic interface who is responsible for sending emails. The content of the class does not related to the article subject, and that’s why I don’t give a content of the class. The `MassTransitEmailSendConsumer` inherits from my own [ConsumerBase.cs](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-consumerbase-cs) class implementing MassTransit’s `IConsumer<T>`.

Now our ASP.NET core app could work with Message Queues using only memory. Let’s continue with Azure services.

## Using Azure Service Bus queues

I will not tell you about how to create an Azure Service Bus (ASB) using portal.azure.com. Here is a [tutorial](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-create-namespace-portal) made by Microsoft. Let’s assume that we have already got a connection string of the Service Bus. How to get it, please read the tutorial from the MS above.

I have created one queue for emailing and a special topic for azure health check. If you don’t need the health-check, you may create only needed queues.

### Azure SB Setup

First, we should set up our application to work with the ASB.

```csharp
// IServiceCollection services;
// MessageBrokerSettings configuration;

services.AddHostedService<AzureBrokerEmailConsumerBackService>();
services.AddScoped<IMessageBroker, AzureServiceBusPublisher>();

services
	.AddHealthChecks()
	.AddAzureServiceBusTopic(
		connectionString: configuration.HealthCheckConnection.ToString(),
		topicName: configuration.HealthCheckTopic.ToString());
```

My app’s `appsettings.json` file contains the following values:

```json
“MessageBroker”: {
  “Connection”: “Endpoint=sb://yournamespace.windows.net/;SharedAccessKeyName=email;SharedAccessKey=awesomesecret”,
  “EmailMessageTopic”: “email-message-queue”,
  “HealthCheckConnection”: “Endpoint=sb://yournamespace.windows.net/;SharedAccessKeyName=healthcheck;SharedAccessKey=awesomesecret”,
  “HealthCheckTopic”: “azuretopic”
},
“UseInMemoryMessageBroker”: true,
```

MessageBroker section is being used by `MessageBrokerSettings` class. `azuretopic` value is a service name of the topic and is used by Health-check library.

### Azure SB Publisher

The ASB accepts a string as the queue message, therefore we have to serialize a message. I use JSON format for the serialization. Here is a code of my publisher:

```csharp
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Services.Infrastructure.Azure;

namespace YourNamespace
{
    public class AzureServiceBusPublisher : BrokerPublisherBase
    {
        private readonly MessageBrokerSettings _config;

        public AzureServiceBusPublisher(MessageBrokerSettings configuration, ILogger<AzureServiceBusPublisher> logger)
            : base(logger)
        {
            _config = configuration;
        }

        protected override async Task PublishInternalAsync<T>(string topicName, T message)
        {
            // create a Service Bus client
            await using var client = new ServiceBusClient(_config.Connection.ToString());

            ServiceBusSender sender = client.CreateSender(topicName);

            // create a message that we can send
            // send the message
            await sender.SendMessageAsync(
                new ServiceBusMessage(JsonConvert.SerializeObject(message)));
        }
    }
}
```

Please pay attention that the class above uses [BrokerPublisherBase](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-brokerpublisherbase-cs) as parent.  We create `ServiceBusClient` for each invocation of the class, and this way is [recommended](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues#add-code-to-send-messages-to-the-queue) by Microsoft.

### Azure SB Consumer

Consuming the SB queue message is not as simple as publishing. We should create a hosted service to consume messages within the background process of the ASP.NET Core app. We will use a `BackgroundService` provided by .net library. We will setup Callbacks for messages and possible errors, and then we will start an endless loop to make the background service working during the main app execution.

```csharp
using System;
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

namespace YourNamespace
{
    public class AzureBrokerEmailConsumerBackService : AzureBusTopicConsumerBase
    {
        public AzureBrokerEmailConsumerBackService(
            ILogger<AzureBrokerEmailConsumerBackService> logger,
            IServiceScopeFactory scopeFactory,
            MessageBrokerSettings brokerSettings)
            : base(
                logger,
                scopeFactory,
                brokerSettings)
        {
        }

        // handle received messages
        protected override NonNullableString MessageTopic => BrokerSettings.EmailMessageTopic;

        protected override Task MessageHandleInternalAsync(IServiceProvider provider, ServiceBusReceivedMessage message)
        {
            string body = message.Body.ToString();
            var email = provider.GetRequiredService<IEmail>();
            return email.SendAsync(body);
        }
    }
}
```

The consumer above inherits from our special class [AzureBusTopicConsumerBase](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-azurebustopicconsumerbase-cs). This class hides most of the code that sets up the background service. Also, the class creates scope for each received message and then provides an instance of `IServiceProvider provider`. The provider is useful to get any business service to execute your task:

```csharp
using var scope = ScopeFactory.CreateScope();
await MessageHandleInternalAsync(scope.ServiceProvider, args.Message);

// complete the message. messages is deleted from the queue.
await args.CompleteMessageAsync(args.Message);
```

## Conclusion

All you need is a config class that will decide what MQ engine will be used for the running application: the InMemory MQ engine either Azure Service Bus. I have created [a helper-class](https://gist.github.com/maximgorbatyuk/b772cb40d5bea823be8dc828e7e4ac27#file-messagebrokerconfig-cs) for this purpose, so you can use it as well. Now you have an application that uses Azure Service Bus for staging and production environments and InMemory engine for the local development.

Hope my article was useful for you. Thank you for the reading!
