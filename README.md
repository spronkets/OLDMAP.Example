# developersBliss.OLDMAP Example

On-Line Domain Message Application Platform

This demonstrates the usage of developersBliss.OLDMAP with a "foobar" application.

So what does this platform do? Instead of designing your applications around database schemas and CRUD endpoints like you would with ASP.NET and Entity Framework, you break it down into lifecycles called Aggregate Roots. Each Aggregate Root processes Domain Messages and contains state. You supply the Aggregate Root definitions, and the platform takes care of the rest. This allows you to more accurately model your business logic (because business processes are stateful and lifecycle-driven) while also not having to worry about the complexities of distributed systems (because all of the infrastructure is handled for you). The platform also provides the following guarantees:

- [x] Exactly-Once Processing Guarantees
- [x] Message Deduplication Guarantees
- [x] Message Ordering Guarantees
- [x] ACID Guarantees
- [x] Concurrency Guarantees
- [x] Retry Guarantees
- [x] Recovery Guarantees

The `FooBar` class is an Aggregate Root (also known as the Entity Layer in Clean Architecture) in which each instance is isolated and receives Domain Messages one at a time. The platform uses these facts to provide the above guarantees. For each Domain Message an Aggregate Root instance receives, you should process it by applying the expected behavior, updating any relevant state, and then returning a corresponding Domain Event that describes what occurred and why. Each instance of a FooBar (with a different Aggregate Root ID) will be given its own partition key in Kafka and its own Marten document in PostgreSQL. Thus, the level of parallelism in your application _depends completely_ on the design of your Aggregate Roots and how many instances of each you create. You can also consider these to be actors or state machines, whichever you prefer. The point being that they are the transactional boundary of your application, meaning all transactions are encapsulated within the processing of a Domain Message by an instance of some Aggregate Root.

The `FooBarService` is an Application Service (also known as the Use Case Layer in Clean Architecture). If there are things that you'd like to do before or after your Aggregate Root processes its Domain Message, any external data that you'd like to grab, etc.; this would be the place to do it, so you can keep your Aggregate Root clean while still applying all of your necessary application logic. It is generally not recommended to _write_ to any external systems within your Aggregate Root or Application Service, as they cannot be reasonably brought under the umbrella of all the guarantees (unless they are idempotent). Application Service method signatures must follow a specific convention in order to be recognized by the platform. If you don't like the default convention or if you have special circumstances, you can always write a custom adapter, but the key thing to notice with the default is that `FooBar` and `FooBarService` contain no references to the platform. This is done in order to align with [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) and is why the platform doesn't just have you inherit from some base class or implement a bunch of interfaces. The platform does not require you to have an Application Service if you don't want one, but the option is there both ways. If your application is simply a bunch of actors with no external dependencies (like in this example with the FooBar Aggregate Root), then you can replace `.AddApplicationServiceWithPureStyle<FooBarService, FooBar>()` with `.AddApplicationServiceWithAggregateRootStyle<FooBar>()`.

Normally the client would be an external system but it's bundled here for brevity; so in this example, when the host starts, the `MyApplicationClient` constructs several Domain Messages and sends all of them off to be processed. They will be processed in the exact order sent. Each Domain Message must be addressed to a specific Aggregate Root instance, _including_ the initializer/constructor. Clients must be responsible for generating Aggregate Root IDs, otherwise many of the guarantees would simply not be possible.

Normally Domain Event handlers would also be external, but again included here for the same reason. The `MyEventHandler` methods are all registered with the Service Collection and will be invoked when the corresponding Domain Event is emitted. This is how you can react to change across application boundaries. The same ordering guarantees, concurrency guarantees, retry guarantees, and recovery guarantees apply here as well. For now, however, the platform does not provide any guarantees beyond that since you will often be writing to external systems. Yet you are given access to enough information to implement your own guarantees should you need them.

# Kafka

https://docs.docker.com/guides/kafka/

# PostgreSQL

https://medium.com/@marvinjungre/get-postgresql-and-pgadmin-4-up-and-running-with-docker-4a8d81048aea
