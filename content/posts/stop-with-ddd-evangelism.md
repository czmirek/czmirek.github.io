+++
title = 'Stop with the DDD evangelism'
date = 2024-05-31T14:36:00+02:00
draft = false
+++

## What is DDD?

**Domain Driven Development** is a software development paradigm focused on the **domain**. The Domain is usually defined broadly:

- *Formally it represents the target subject of a specific programming project, whether narrowly or broadly defined.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *"sphere of knowledge and activity around which the application logic revolves"* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))
- *A sphere of knowledge, influence, or activity. The subject area to which the user applies a program is the domain of the software.* ([Wikipedia](https://en.wikipedia.org/wiki/Domain_(software_engineering)))

So what is it for, from the perspective of a software developer? The primary objective is to isolate the domain to a specific area in your code. In other words, all **"knowledge of what the application does for the user"** should exist in one logical area. 

But don't we already have this concept in software development? It's called **the business logic**. I believe there is an important difference though.

- **The business logic** is any code that is related to the requirements of the "business". This can be anywhere in code.
- **The domain** is always a business logic code but in a certain "domain scope" which is nothing else but just a logical group of code dedicated purely for your business logic.

### Business logic example

Let's have an example.

This code contains business logic i.e. code "required by the business" but you wouldn't call it a domain code.

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
- Invalid state of our code means we throw exception. I personally don't like throwing exceptions to represent invalid business logic state but I understand why it's appealing for some.

## Is this the only way how to write the domain?

The `class User` in the example above is a **root aggregate** which is part of the traditional terminology for DDD. You can see that there is no persistence mechanism visible in this code; persistence must be handled somewhere else. Or you can do event sourcing but I'll get to that later.

If I follow the definition that the domain is basically just a logical part of my code that isolates the business logic in one place then no, the domain can be written in a different way.

Let's look at the request-response handler example.

```csharp
public class CreateUserHandler
{
    public async Task<CreateUserResponse> CreateUser(CreateUserRequest request)
    {
        
    }
}

```


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