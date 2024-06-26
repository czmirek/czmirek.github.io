+++
title = 'DDD evangelism'
date = 2024-06-03T16:14:00+02:00
draft = false
+++

## Foreword

This article is criticism of how DDD is usually implemented in software, how DDD evangelists try to convice us (but mostly themselves) about the validity of their views and finally my take on DDD.

## Definition

**Domain Driven Development** is a software development paradigm focused on the **domain**. The Domain is usually defined broadly:

- *Formally it represents the target subject of a specific programming project, whether narrowly or broadly defined.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *"sphere of knowledge and activity around which the application logic revolves"* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *A sphere of knowledge, influence, or activity. The subject area to which the user applies a program is the domain of the software.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))

So what is it for? It's for isolating the domain to a specific area in your code. All **"knowledge of what the application does for the user"** should exist in one logical area or layer called **the domain layer**. 

Don't we already have this concept in software development? It's called **business logic**. There's a difference though.

- **The business logic** is any code related to the requirements of the "business". This can be anywhere in your code.
- **The domain** is always a business logic code but in a certain "domain scope" which is nothing else but just a logical group of code dedicated purely for your business logic.

## What is the "domain"?

You are a software developer and your role is to transform the business requirements into a working software. The most important concept from DDD is isolation of **the domain layer** which is the part of your code that focuses on **expressing what the user/business wants**. 

Why? **For your own clarity!** Isolating the business logic into a specific area allows you to see the most important part of your code in one place.

### Clarity & semantics in the domain layer is the main priority

The way the domain layer is written should equal to the way the business defines itself and its requirements. The semantics is important in the domain layer.

Let's look at examples.

```csharp
public class ProductService(IProductRepository repo)
{
    public void CreateProduct(string? name = null)
    {
        if(string.IsNullOrEmpty(name))
            name = "(no name)";

        repo.Insert(new ProductEntity
        {
            Id = Guid.NewGuid(),
            Name = name
        });
    }
}
```

If you want to isolate the domain layer, you should ask the following questions:

- Is it an explicit business requirement that...
   - ...the product can be created without the name?
   - ...the product can be created with name set to `null` or empty string?
   - ...the ID of the product is defined by `Guid` ID?
   - ...the product has a business defined ID at all?

You can go all extreme with this.

- Does the business even care about anything called `ProductService`?
- What about `ProductEntity`?
- What about `IProductRepository`?

Or even to the absurd.

- What about keywords such as `public`, `class`, `void`? Are they defined by the business? Should I pick a different programming language that is better at expressing business ideas?

Where do you set the limit on how to express the business ideas in your code? **That's up to you!** Expressing the business ideas into the domain layer is an art of balance between:

- expressing the business ideas as clearly as possible 
- having a transparent, clean and understandable code architecture

You cannot have both. Going too far with the expression of the domain layer means you are overengineering. Or you don't have the domain layer at all and your business logic is mixed with everything else in your application. This equals to unmaintainable code that is very hard to work with and navigate. And that makes us software developers sad 😥

## DDD and persistence

DDD is usually implemented with rich classes called **aggregate root** like this.

```csharp
public class User
{
    public string Name { get; }
    public User(string name)
    {
        if(name.Length < 3 || name.Length > 10)
            throw new DomainException("Name must be between 3 and 10 characters long");

        Name = name;
    }
}
```

The nice thing about using root aggregates is that we've isolated our domain from the persistence and our `class User` represents only the in memory business logic, pure and nice.

...but hang on. 🤔

Is that even a right thing to do?

**Does it even make sense to define the domain layer without persistence?**

I'm saying **it depends**.  

Are you programming:
- monitoring UI that reads and analyses data from a stock exchange feed but is not supposed to store anything?
- e-shop where you need to store products, prices, warehouse stock, articles? 

Obviously the answer to whether or not you need persistence is **defined by the business**. It's also perfectly reasonable to have a single application in which parts of the domain layer require persistence and other parts don't.

That's very clear and understandable right? 

You might have worked for a client that might have not immediately realised that the information about the "product" in his e-shop must be physically stored somewhere, somehow. Or the client or your employer expect that the concept of persistence is automatically understood with requirements.

Now comes the DDD evangelist.

He'll say something retarded like *"The business does not care about persistence"*. I've heard this even in a corporation that was explicitly saying "We are data driven" in its IT strategy. The correct reply is that business does not care about exact persistence mechanism but most of the times it cares about persistence implicitly.

Or he may say something mysterious like *"We must dig deeper into the requirements to understand what the business REALLY wants."* DDD evangelists tend to look for problems where there are none and the requirements are clear.

## Services in your domain layer

### ...according to evangelists
Let's say that the business wants you to use MS Dynamics 365 as a storage for your product images. 

The DDD evangelists say that external services do not belong into the domain layer at all, again, defending the idea of a rich aggregate root that nicely represent a *pure* view on the business logic. You'd have to overengineer and use event driven programming to coordinate data between your aggregates and the image store.

### ...according to me

But in this case, the business is explicitly saying "use MS Dynamics 365 as a storage for product images". Is it a good idea to use MS Dynamics 365 as a storage for product images? I have no idea but let's assume it is what it is.

How would you handle this? How do you name the interface for the image service?

- `IImageStore`?
- `IProductImageStore`?
- `IDynamics365ImageStore`?
- `IDynamics365ProductImageStore`?

If your domain:

- does not already know a service that represents a MS Dynamics 365 image store
- and there are no other requirements related to images OR Dynamics 365

...then in my opinion you should create an interface called `IDynamics365ProductImageStore`. This is the "DDD" way of programming because you are following the semantics of the business requirements as they are right now. You should not create `IImageStore` because semantically it sounds like a general purpose image store for whatever images you want and you risk building something that won't be useful at all. 

You should not predict the future in your domain layer which should represent the **current** state.

## Event sourcing

Aggregate roots are nicely combined with event sourcing (ES). ES is this neat idea that all business logic can be reduced to a list of events that represent change of state.

Your single source of truth can be a single denormalized list of event data. The state of your application is represented by this list replayed over your root aggregates.

With ES the architecture around aggregate roots become more complex and they usually look like this.

```csharp
public class User(IEventSender eventSender) : IEventHandler<UserNameChanged>
{
    public string Name { get; private set; }

    // methods to change the state produce events, these events are stored
    // into the ES store
    public void ChangeName(string newName)
    {
        eventSender.Send(new UserNameChanged
        {
            NewName = newName
        });
    }

    // the aggregate root must be restorable from the ES store
    // therefore it must know how to react to the events that
    // change its state 
    public void Handle(UserNameChanged @event)
    {
        Name = @event.NewName;
    }
}
```

For me the code above is already getting less readable. Some aggregate roots tend to be very complex and with all the Handle methods the code can easily grow to too many lines of code.

ES includes more complications. 

- snapshots because you cannot keep loading the full list of events on any change of state
- event versions because the business requirements just keep changing
- read models because you cannot read from the domain data directly

ES in my opinion is not worth it. It involves too many complexities and architecting.

## Collective rules

DDD breaks when you consider collective rules. One of the typical examples is unique constraint on user emails - in other words, a business requirement that user should have unique emails.

How do you do that? Let's look at what the DDD evangelists are saying.

### [SoftwareEngineering - Is there an elegant way to check unique constraints on domain object attributes without moving business logic into service layer?](https://softwareengineering.stackexchange.com/questions/318705/is-there-an-elegant-way-to-check-unique-constraints-on-domain-object-attributes?rq=1)

> While it is true that domain should be persistence ignorant...

No it should not. The domain layer should account for persistence if the business requirements account for persistence!

The domain layer is supposed to be the code that isolates and expresses the business requirements and ideas. If you define DDD only as *in memory aggregate roots* then you are contradicting the business requirements which expect you to understand that things live in a storage.

> ...it does know that there is "Collection of domain entities". 

Yes, using rich aggregate roots forces me to find a different solution to how to handle constraints over collections. But how we separate the domain layer from persistence is up to us. 

In my opinion the best way to represent repository in the domain is with interface that is as plain as possible.

```csharp
public interface IUserRepository
{
    Task Add(User newUser);
    //etc...
}
```

Repository interfaces defined by the domain layer must be as plain as possible. The implementation can be whatever (sql, mongodb, ...) but you should not do something like `IUserRepository : ICrudRepository` unless you are specifically requested by the business to operate on your entities in a CRUD manner.

You should treat your domain layer as a completely handwritten project with zero infrastructure/framework code. This forces you to write only the code that is necessary in a way that focuses on conveying the idea of the business requirements and nothing else.

> And that there are domain rules that concern this collection as a whole. Uniqueness being one of them. And because the implementation of the actual logic heavily depends on specific persistence mode, there must be some kind of abstraction in the domain that specifies need for this logic.

I agree with this.

The domain layer is representation of all business requirements and it's very normal for the business to have conditional requirements over multiple aggregates even of different types. It's not only about unique constraints but any sort of collective constraint.

### [stackoverflow: DDD - Validation of unique constraint](https://stackoverflow.com/questions/2660817/ddd-validation-of-unique-constraint)

> *So you may want this in your validation mechanism, but you must enforce it in your persistence layer*

Wrong. The email uniqueness is a real business requirement so it belongs into the domain. It calls the peristence layer, because databases use indexes which can be used very effectively for searching. It's the simplest and most sane technological solution.

> In essence: Look for the scope of the uniqueness and store an authoritative list of unique values inside an aggregate root representing that scope.

This is a completely retarded. Do I really need to load all emails into memory every time I want to create a new user?

### [SoftwareEngineering - How to handle business rules that are "uniqueness" constraints?](https://softwareengineering.stackexchange.com/questions/386671/how-to-handle-business-rules-that-are-uniqueness-constraints)

> The uniqueness of an entity is not a business rule!
> Read that again. I'll wait for it to sink in...

DDD evangelist contradicting the obvious.

Uniqueness of entity IS a business rule. Let THAT sink in. There are business rules and constraints across entities, even across different types of domain entities.

> The uniqueness of a domain object is a technical invariant.

False. Uniqueness over a domain entity **can be** a normal business requirement. There's nothing wrong with that.

DDD evangelists will argue that uniqueness constraints usually hide a different truth about the domain that the business is not expressing in their requirements. For example: *"Two users cannot have same address"* is a business requirement that hides the real requirement: fix the bug in our marketing platform which cannot work with two different users with the same address.

But that's not a software development issue but issue of humans understanding each other. You cannot guarantee that. Not even a DDD evangelist can although he believes he always knows better.

Programming with DDD forces you to understand the business requirements perfectly because no sane software developer likes to code something he does not understand. But sometimes the constraints are fairly simple to understand and trying to reimagine them with different concepts makes no sense.

> The only reason you need some sort of id value is to allow for persistence/hydration. That is, if you never had to serialize/unserialize your model you wouldn't need any id values!

Business does not care about IDs but that's because the requirement is implicit most of the time. When your client/employer wants you to code a website with product administration it's obvious that the product must be uniquely identifiable somehow because you need to read the product from your API based on some identifier.

#### Using database IDs

Your persistence mechanisms should comply to decisions made in your domain layer and not vice versa. Your persistence layer should **NOT** define the IDs for your domain layer, it must be the other way around. Do not architect a system where the IDs created in the persistence layer somehow propagate into your domain entities - it's not worth it. 

If you code a domain layer, you should NOT use any kind of auto-generated int IDs (and such) in your persistence layer. That way you are delegating the business logic from your domain layer into your persistence layer which destroys the whole purpose of the domain layer --- to isolate all business logic into one place.

#### ID-less domain entities

Not every domain entity needs to have an ID. For example you are working on an application for a logistics company that tracks GPS coordinates of vehicles every single minute. Sometimes the driver stops for a law mandated rest at a gas stations. Then you have series of domain entities with the same coordinates.

```txt
Latitude                Longitude           Time
50.152259269818046      14.49680288733176   2024/06/02 21:04
50.152259269818046      14.49680288733176   2024/06/02 21:05  
50.152259269818046      14.49680288733176   2024/06/02 21:06
...
```

The business wants to know where the vehicle was and when...but that's all. The tracking entities do not need to have IDs at all. They are only listed or filtered by the time.

## What persistence to use with DDD?

Use anything you like. I believe that document databases such as MongoDB fit DDD better because any domain entity can be stored as a single document. I like simple solutions and you cannot go any simpler than that.

Using SQL RDBMS with DDD is more complicated. If your domain entity is complex then the implementation of your persistance layer is complex as well. In document databases it's much easier to implement any given entity in a CRUD manner because for *update* you just completely replace the whole document (and then optimize if needed). In SQL RDBMS you are always forced to care what exactly changed in the given update operation so you don't write the update for all relevant tables (unless it's necessary).
 
## Difference between *the domain layer* and *the business layer*

I believe that before DDD most of software developers learned about the traditional 3-layer architecture that was sitting between UI and database layer.

But there's a difference. In the traditional 3-layer architecture we put everyhing into the business layer. Caching, logging, monitoring and transformations between the UI and the database layer. Business layer was everything that was not UI related and not database related.

The domain layer however does not care whether there are other layers. If your business accounts for persistence then your domain should account for persistence e.g. your domain does not separate "business" or "persistence" layer, it's all domain.