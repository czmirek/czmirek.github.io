+++
title = 'POST only API standard'
date = 2024-09-28T15:21:00+02:00
draft = false
+++

This is a for fun project aimed at creating a sensible API standard.

## Critique of the REST convention

🤨 **REST does not say anything about common topics related to API development.**

As far as conventions go there's not much else in what REST APIs offer for a common API developer. REST implementations vary considerably in practice (in my opinion and experience) because the convention does not go very far in standardizing common needs of common API developers. 

Namely:

- sane HTTP status code logic
- failed validation models and erroneous behaviour

🤨 **Caching**

Using HTTP header `Cache-Control` you can alter behavior on how browsers cache your `GET` responses.

*"The goal of caching is never having to generate the same response twice."* [^1]

This requirement is normally ignored because:
- Caching should be considered only when we optimize things. And we should optimize things only when we find that our things don't run as fast as we want them to.
- Caching is a hard problem. How to cache? When to cache? What to cache? For how long? Do we need to reset the cache reactively? Or on timed manner? How is the cache stored?
- Utilizing `Cache-Control` can be just a single part of multilayered cache solution that utilizes the browser cache and/or the server cache.

🤨 **Hypermedia links**

Hypermedia links are references to URLs of other resources that are in some relation to the data you are returning.

This is an example from the site [What is REST?](https://restfulapi.net/)

```json
{
  "id": 123,
  "title": "What is REST",
  "content": "REST is an architectural style for building web services...",
  "published_at": "2023-11-04T14:30:00Z",
  "author": {
    "id": 456,
    "name": "John Doe",
    "profile_url": "https://example.com/authors/456" //---------- REST link
  },
  "comments": {
    "count": 5,
    "comments_url": "https://example.com/posts/123/comments" //-- REST link
  },
  "self": {
    "link": "https://example.com/posts/123" //------------------- REST link
  }
}
```

This is almost never implemented because it is **difficult** to implement for no benefit.

- Your API now depends on at least a single domain. This complicates architectures considerably. Can the domain change? Who's in charge of configuring this? What if the API is available from multiple hosts?
- You need some kind of mapping between your API endpoints and their URL representations. This complicates API code. Also how do you handle API versioning with this (e.g. when only the `comments` API is updated)?
- ...what is this for anyway? For example, why on earth would you ever need to utilize the `self:link`? Trying to build on this smells like a bad project architecture than anything else.

🤨 **HATEOAS**

According to [Wikipedia](https://en.wikipedia.org/wiki/HATEOAS): *With HATEOAS, a client interacts with a network application whose application servers provide information dynamically through hypermedia. A REST client needs little to no prior knowledge about how to interact with an application or server beyond a generic understanding of hypermedia.*

- This is overengineering. No one in common commercial practice is building applications like that.
- It is difficult to implement. You can either do this manually (`<sarcasm>`*because that's what we developers love, do manual stuff*`</sarcasm>`) or you must implement something for the reflective knowledge about the resources and their relations.
- Clients are commonly hardwired to their APIs in practice because of UI/UX constraints.
- Independent clients depending only on a *generic understanding of hypermedia* means you cannot have meaningful UI/UX

🤨 **The REST convention is very old (2000)**

And from the time where we didn't know much about how IT projects and APIs are going to evolve.

🤨 **[Richardson maturity model](https://en.wikipedia.org/wiki/Richardson_Maturity_Model) is pure bullshit (and also old (2008))**

So according to someone named *Leanord Richardson* the REST convention is so important that there are official **maturity models** to it.

Short version so you don't have to read the Wikipedia article

- Level 0 = one HTTP endpoint equals to multiple functions.
  - For example, you send a POST over for `/bookings` and for creating booking you send `{ "operationId": "createBooking" ...}`, for updating the booking you send `{ "operationId": "updateBooking" ...}`
- Level 1 = one HTTP endpoint equals to a single function (e.g. `/bookings/createBooking`, `/bookings/updateBooking` etc.)
- Level 2 = you use HTTP verbs as defined by REST e.g. `GET` for reading resources, `POST` to update resources, `PUT` to create resources, etc.
- Level 3 = you use HATEOAS

This is complete nonsense built on ideas of another nonsense. I've seen it personally being promoted in corporations by IT architects with no knowledge in modern programming.

🤨 **HTTP protocol is just a protocol**

Complicated reactive architectures have multiple endpoints for doing a single thing. For example, the function of the `/createBooking` endpoint can be invoked by a message comming from a message bus. The function itself can be part of other bigger functions that may or may not have HTTP endpoints at all. Imposing a REST convention on yourself forces you to think about how it fits in a wider context of the whole system but giving you nothing useful in return.

## Identifying the needs for common API developers

❗**Backend - frontend agreements**

Full stack developers are getting less and less common. IT teams are split between backend and frontend developers who need to understand rules for how to handle validations and HTTP status codes.

❗**Business requirements over conventions**

IT projects are driven by business requirements. Business requirements are ideas from people who pay us developers to realize these ideas into products. The standard in this text is described with this in mind.

# ✨ The POST only API standard ✨

**Pretext about JSON casing**

This convention defines JSON models which have properties written in `camelCase`. The casing is not important and you can use the model in any casing that suits you.

## 1. HTTP usage

### 1.1. Client agnostic API
When you build an API, you **SHOULDN'T** care if it's going to be consumed by a browser, a native application or some different API in a kubernetes cluster.

### 1.2. Ignore HTTP request headers
1. Request headers are an implementation detail of the HTTP protocol. You should ignore any incoming HTTP request headers.
1. However, if your API needs to follow some other standard or needs headers for something **not related to the business logic of your API** to work properly then handle the headers accordingly.
1. You must not invent your own proprietary protocols or ideas built on HTTP headers. All data from clients should belong to the HTTP POST request body.

### 1.3. Do not send any HTTP response headers
1. Response headers are an implementation detail of the HTTP protocol. You should never send any HTTP headers from your API.
1. However, if your API needs to follow some other standard or needs headers for something **not related to the business logic of your API** to work properly then return the headers accordingly.
1. You must not invent your own proprietary protocols or ideas built on HTTP headers. All data sent back to clients should belong to the HTTP POST response body.

### 1.4. Accept the HTTP POST verb only
1. Your endpoints should accept the HTTP request only with the HTTP POST verb.
1. If the client sends a different HTTP verb then the HTTP response with status code 405 and **empty body** should be returned.

### 1.5. All communicated data (except files) belong to the POST body in the form of JSON
1. The body of the HTTP request may be empty or contain a valid JSON
1. The body of the HTTP response may be empty or contain a valid JSON

### 1.6. HTTP status codes

Your API should ever return only the following HTTP status codes.

#### 1.6.1 200

1. All API responses should by default always return HTTP status code 200 for all cases.
1. The meaning is same as in HTTP RFC - the call was successful.

#### 1.6.2 400

1. If the HTTP request data are invalid **for any reason** e.g. related to the JSON model itself or related to the business logic provided by the API, return HTTP 400. The content of the body is discussed in detail in chapter 3.
1. Choice of using HTTP 400 is not arbitrary. Clients detecting HTTP 400 can drop any expectations about the response and instead read the validation errors from the standardized model described in chapter 3.

#### 1.6.3 500

1. All errors in your application that do not crash your API on HTTP request should return HTTP 500. This means that HTTP 500 response from your API is always a bug on your API.
1. The body of the HTTP response should by default contain an empty body. You should not normally send details about your exceptions/errors to the client.
1. For debugging purposes the HTTP response CAN contain a body containing a crash log, exception stack trace, etc., in a plain text format.
1. If the UI clients detect HTTP 500 from the API they should always display a clear message that the server errored upon request. If the response contains a plain text error then it must be presented in the UI to the user.

#### 1.6.4 All other status codes
1. Ignore HTTP RFC specifications for any other status codes
1. Do not ever utilize any custom status codes outside the HTTP RFC range in any case.

## 2. API input models and output models
1. You are free to model your input and output models however you want.
1. However - do not build a parent model for handling validations which is commonly used. These models usually have properties like `response` and `error`. Clients using this convention should always first detect the returned HTTP status code. If the returned status code is 400 then the clients should always expect the same validation model in the response. This model is described in the chapter 3.

## 3. First error validation response model

There are two common ways on how validations are normally handled by APIs.
- **first error only**: easier to implement
- **multiple errors**: more descriptive

This convention promotes the **first error only** approach not only because it's easier to implement but because it's also easier to reason about. 

Returning **multiple errors** is very often interpreted as **all errors** but returning all errors cannot be guaranteed for all business cases. Validating something may depend on successful validation of something else. Validation may or may not trigger something that changes the validation behvaiour in the next validation attempt. Business logic should always be treated like **"anything can happen"**.

Another argument is that modern applications implement validations on the frontend side as well. The model returned by the backend should signal either that a validation on frontend is not yet implemented or the validation on frontend is not possible at all.

### 3.1. Failed validation model properties

#### 3.1.1 Error code
Error code is a SCREAMING_SNAKE_CASE text.

- It must not start with number and can contain only upper case alphanumerical english letters and underscore.
- The error code must have at least 5 characters and is recommended to have at most 100 characters.
- It should NOT contain any data.
- Error codes should be hardcoded strings (or constants, enums, etc.) in the codebase of your API.
- The default value is VALIDATION_ERROR. Error code must be ALWAYS present in the failed validation response.

#### 3.1.2 Error message
Error message is a human readable descriptive text which is by default in the default language of your project.

- It can contain data about the request.
- It is not compulsory and is `null` by default.
- The message should follow any logic you have in your system for localization.

#### 3.1.3 References
Array of the input model properties that are responsible for this validation to fail. If used together with `attemptedValues` then the references should line up with the values. This array can be used by UI clients to visually mark fields or areas where the server validation failed.

- It is not compulsory and can be omitted from response.
- It can be `null`
- The array can contain only strings

#### 3.1.3 Attempted values
Array of the input model values that are responsible for this validation to fail. If used together with `references` then the attempted values should line up with the references. This array can be used by UI clients to highlight the erroneous value that was provided.

- It is not compulsory
- It can be `null`
- The array can contain any valid JSON type including objects

#### 3.1.4 JSON schema
```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "errorCode": {
      "type": "string"
    },
    "errorMessage": {
      "type": ["string","null"]
    },
    "references": {
      "type": ["array", "null"]
    },
    "attemptedValues": {
      "type": ["array", "null"]
    }
  },
  "required": [
    "errorCode"
  ]
}
```

#### 3.1.5 Example
```json
{
  "errorCode": "INVALID_REFERENCE",
  "errorMessage": "Please include valid User ID",
  "references": ["UserId"],
  "attemptedValues": [555]
}
```

## 4. Modify but communicate

Take this convention and change it however you like to fit your needs! 

Do take care of communicating your updated rules properly to all of your backend and frontend developers, to your team and to all your API consumers.

[^1]: [Caching your REST API](https://restcookbook.com/Basics/caching/)
