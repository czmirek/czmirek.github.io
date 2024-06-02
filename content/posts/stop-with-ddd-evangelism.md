+++
title = 'Stop with the DDD evangelism'
date = 2024-05-31T14:36:00+02:00
draft = false
+++

**Domain Driven Development** is a software development paradigm focused on the **domain**. The Domain is usually defined broadly:

- *Formally it represents the target subject of a specific programming project, whether narrowly or broadly defined.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *"sphere of knowledge and activity around which the application logic revolves"* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *A sphere of knowledge, influence, or activity. The subject area to which the user applies a program is the domain of the software.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))

So what is it for? It's for isolating the domain to a specific area in your code. All **"knowledge of what the application does for the user"** should exist in one logical area or layer called **the domain layer**. 

Don't we already have this concept in software development? It's called **business logic**. There's a difference though.

- **The business logic** is any code related to the requirements of the "business". This can be anywhere in your code.
- **The domain** is always a business logic code but in a certain "domain scope" which is nothing else but just a logical group of code dedicated purely for your business logic.

### Business logic example

This code contains business logic i.e. code "required by the business" but you wouldn't call it a domain code because there's a lot of infrastructural code.

```csharp
//not a business logic
[HttpPost("users/create")]
public async Task<IActionResult> CreateUser(string name)
{
    //might be business logic
    string nameSanitized = name.RemoveUnsafeChars();
    
    //is valid business logic
    //it's directly linked to a business requirement
    //that name must be between 3 and 10 characters
    if(nameSanitized.Length < 3 || nameSanitized.Length > 10)
        return BadRequest(); //not a business logic 
                             //(business does not care about HTTP)

    //would't call this business logic
    await userRepo.CreateUser(email, name);

    //not a business logic
    //(business does not care about HTTP)
    return Ok();
}
```
### Domain example

The following code is a domain code.

```csharp
public class User
{
    public string Name { get; private set; }

    public User(string name)
    {
        string nameSanitized = name.RemoveUnsafeChars();
        if(nameSanitized.Length < 3 || nameSanitized.Length > 10)
            throw new DomainException("Name must be between 3 and 10 characters long");

        Name = nameSanitized; 
    }

}
```

Everything about the example above is interpretable as the business logic.

- The class has a single only constructor with a parameter. It's obvious that the user cannot be created without the name.
- We can see that the name cannot be changed once the user is created, because there is no setter. This MAY represent a business requirement as well.
- We can easily see that the name must be valid in a certain way. You don't need to analyse which part of code is related to business logic and which is not.
- Invalid state of our code means we throw exception. I personally don't like throwing exceptions to represent invalid business logic state but in this case it is necessary.

## Persistence

The `class User` in the example above is a **aggregate root**. Basically the logical group of our domain or you could say a domain entity.

The nice thing about using root aggregates is that we've isolated our domain from the persistence and our `class User` represents only the business logic, pure and nice. 

...but hang on.

Is that even a right thing to do?

**Does it even make sense to define the domain layer without persistence?** The DDD evangelists say yes, I'm saying it depends.

Continue reading.

## How I define the domain layer

You are a software developer and your role is to transform the business requirements into a working software. The most important thing from DDD is isolation of **the domain layer** which is the part of your code that focuses on **expressing what the user/business wants**. 

Why? **For your own clarity!** Isolating the business logic into a specific area allows you to separate "business" from "technology".

Because software engineering (the "technology" part) is hard. You have to understand and work with so many things that are unrelated to the "business": logging, caching, naming things, exception handling, using right patterns and algorithms, optimizing, monitoring, persistence mechanisms, etc.

Separating "business" to it's own layer helps you in most cases and in most businesses. That's why the idea of the domain layer is so appealing for most. 

## Technology agnosticism

Code in the domain layer is supposed to be written in the most technology-agnostic way possible. Domain layer shouldn't care about caching, logging, exception handling, etc., all of that should be handled in any layer above the domain layer.

The domain layer is the layer at the bottom, it's the code where any state change request eventually goes.

## Clarity & semantics in the domain layer is the main priority

The way the domain layer is written should equal to the way the business defines itself and its requirements. The semantics is extremely important in the domain layer.

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

The code above is okay if you don't use the domain layer. But if you want to isolate the domain layer, you should ask the following questions:

- Is it an explicit business requirement that...
   - ...the product can be created without the name?
   - ...the product can be created with name set to `null` or empty string?
   - ...the ID of the product is defined by `Guid` ID?
   - ...the product has a business defined ID at all?

You can go all extreme with this.

- Does the business even care about anything called `ProductService`?
- What about `ProductEntity`?
- What about `IProductRepository`?

Or even most extreme.

- What about keywords such as `public`, `class`, `void`? Are they defined by the business?

**Of course there's a limit!** You are a software developer and you code using a certain programming language and not a natural language. Where do you set the limit on how to express the business ideas in your code? **That's up to you!** Expressing the business ideas into the domain layer is an art of balance between:

- expressing the business ideas as clearly as possible 
- having a transparent, clean and understandable code architecture

You cannot have both. Going too far with the expression of the domain layer means you are overengineering. Sure, you have your nice and pretty aggregate roots but then you need to have all this complicated architecture around.

The other extreme is that you don't have the domain layer at all and your business logic is mixed with everything else in your application. This equals to unmaintainable code that is very hard to work with, navigate and makes us sad :(

### DDD evangelists

The internet is full of DDD evangelists trying to convince others that the domain layer is so important that it's worth the overengineering. Not only that, they claim all sorts of bullshit, trying to justify hundreds of hours spent into their overengineered projects.

## Does it make sense to define the domain layer without persistence?

Are you programming:
- monitoring UI that reads and analyses data from a stock exchange feed but is not supposed to store anything?
- e-shop where you need to store products, prices, warehouse stock, articles? 

Obviously the answer to whether or not you need persistence is **defined by the business** or by the requirements. It's also perfectly reasonable to have a single application in which parts of the domain layer require persistence, other parts don't!

That's very clear and understandable right? You might have worked for a client that might have not immediately realised that the information about the "product" in his e-shop must be physically stored somewhere, somehow. Or the client or your employer expect that the concept of persistence is automatically understood with requirements.

Now comes the DDD evangelist.

He's misinterpreting the lack of clear business requirement about persistence to defend his overengineered project with nice and clean persistenceless aggregate roots. So he'll say something retarded like *"The business does not care about persistence"*. Or something mysterious like *"We must dig deeper into the requirements to understand what the business REALLY wants."* And I've heard or read this variant so many times. 

Like, what the fuck? The business is literally saying *"The product should not be stored if it has no name"* and your response is *"You shouldn't care about persistence"* ???

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

For me the simple code above is already loosing clarity about the domain because the aggregate root is just a single class. Some domain entities tend to be very complex and with all the Handle methods the code can easily grow to thousands of lines.

Also with ES comes other complications. 

- snapshots because you cannot keep loading the full list of events on any change of state
- event versions because the business requiremts just keep changing
- read models because you cannot read from the domain data directly

## Collective rules

DDD breaks when you consider collective rules. One of the typical examples is unique constraint on user emails - in other words, a business requirement that user should have unique emails.

How do you do that? Let's look at what the internet is saying.

### [stackoverflow: DDD - Validation of unique constraint](https://stackoverflow.com/questions/2660817/ddd-validation-of-unique-constraint)

> *So you may want this in your validation mechanism, but you must enforce it in your persistence layer*

Wrong. The email uniqueness is a real business requirement so it belongs into the domain. It calls the peristence layer, because databases use indexes which can be used very effectively for searching. It's the simplest and most sane technological solution.

Let's say I want to create a user. There's some validation in my `User` aggregate root but I also want to enforce an unique email which is something I obviously cannot do in the `User` class

So instead of trying to bend everything for aggregate roots I'll create a service in my domain layer that is responsible for creating the user.

```csharp
public class UserService(IUserRepository userRepo, IAggregateRootLayer aggregateRootLayer)
{
    public async Task CreateUser(CreateUserRequest request)
    {
        //validate unique email
        if(await userRepo.IsUniqueEmail(request.Email))
            throw new DomainException("User must have unique email");

        //forward to your DDD infrastracture
        await aggregateRootLayer.ForwardRequest(request);
    }
}
```

> In essence: Look for the scope of the uniqueness and store an authoritative list of unique values inside an aggregate root representing that scope.

This is a completely retarded. Do I really need to load all emails into memory each time I want to verify uniqueness of emails?

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

> And that there are domain rules that concern this collection as a whole. Uniqueness being one of them. And because the implementation of the actual logic heavily depends on specific persistence mode, there must be some kind of abstraction in the domain that specifies need for this logic.

I agree with this.

The domain layer is representation of all business requirements and it's very normal for the business to have conditional requirements over multiple aggregates even of different types. It's not only about unique constraints but any sort of collective constraint.

### [SoftwareEngineering - How to handle business rules that are "uniqueness" constraints?](https://softwareengineering.stackexchange.com/questions/386671/how-to-handle-business-rules-that-are-uniqueness-constraints)

> The uniqueness of an entity is not a business rule!

DDD evangelist contradicting the obvious. Nothing else, really.

Uniqueness of entity IS a business rule. Let that sink in. There are business rules and constraints across entities, even across different types of domain entities.





## Some problems are not representable well with 

## DDD and ES


## Is this the only way how to write the domain?

Let's look at the request-response handler example.

```csharp
public class CreateUserHandler(IUserRepository userRepository)
{
    public async Task<DomainResponse> CreateUser(CreateUserRequest request)
    {
        string nameSanitized = request.Name.RemoveUnsafeChars();
    
        if(nameSanitized.Length < 3 || nameSanitized.Length > 10)
            return DomainResponse.Invalid("Name must be between 3 and 10 characters");

        await userRepository.CreateUser(request.Name);

        return DomainResponse.Success;
    }
-}
```

What is this? Can you call this a domain? Or is this a business logic implemented?


## Domain layer

How you define the logical group?




The domain is the code we write for **the user**. It's the part of the code where we **isolate** what the user wants from everything else.

The definition of DDD is very broad and it's synonymous with **the business logic**. Business logic is nothing else but a layer or part of your code that defineds "what the business wants" whi



. It is basically a paradigm which says that **the business logic** is the primary focus of your application.

The "domain" is either synonymous with "the business logic of the application" or it points to a logical group of the business logic. 





Look at the definitions above. What does it say? A "spehere of knowledge" or "target subject of a specific programming project". 

Software engineering is hard. You have to understand and work with so many things that are unrelated to "what the user wants": logging, caching, naming things, exception handling, using right patterns and algorithms, optimizing, monitoring, persistence mechanisms, etc.

The idea of DDD is to isolate the domain or **what the user wants** into a part of code that is separated from all that difficult stuff mentioned above.

## Why is DDD confusing

After browsing this topic on the internet for some time I feel like the online developer community keeps forgetting and then is rediscovering why the DDD paradigm is so useful.

We do DDD because it gives us clarity about the code. We separate the "business logic" into 



DDD paradigm is not very useful if:

- the thing we are building is very small in scope. 
- it's hard or meaningless to define **the user**. For example if we are programming an air-to-air rocket navigation system.



## Implementation

How you implement the domain **is not important**. 