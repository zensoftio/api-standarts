# API Specification

The description of the common points for the API development.

## Base CRUD operations
The URLs for the basic CRUD operations must start from the entity's name in the plural form and in the snake_case. 
The URL in that form describes operations for the all entities. For example:
> /users

It means that all requests on this URL affect the entities with written name.

The URLs which affect only one entity must contain ID of the entity in the URL after name:
> /users/{id}

> /users/1

> /users/abcd-qwert-2343

The IDs must be undivided string (number). It's recommended to be a GUID.

**There is the short description, for more details, please, see below:**
#### Add:
> POST /users

>PAYLOAD:
```json
{
  "username": "Vasya",
  "email": "vasya@gmail.com"
}
```

> RESPONSE 200 OK:
```json
{
  "id": "afdf-3f4f-65fd",
  "username": "Vasya",
  "email": "vasya@gmail.com"
}
```

Adds new entity and returns the result. Request must be a DTO which describes all needed information for new entity creation.

#### Update:
> PUT /users/afdf-3f4f-65fd

>PAYLOAD:
```json
{
  "username": "Petya",
  "email": "petya@gmail.com"
}
```

> RESPONSE 200 OK:
```json
{
  "id": "afdf-3f4f-65fd",
  "username": "Petya",
  "email": "petya@gmail.com"
}
```

Updates the entity by id in the URL. The DTO can be different.

#### Patch one:
> PATCH /users/afdf-3f4f-65fd

>PAYLOAD:
```json
{
  "username": "Dimon"
}
```

> RESPONSE 200 OK:
```json
{
  "id": "afdf-3f4f-65fd",
  "username": "Dimon",
  "email": "petya@gmail.com"
}
```

The patching means that the client sends the delta-changes only (field(s)) which must be updated. The field will be updated with passed value.

#### Patch many:
The patching can be applied for sets too. For example:
> PATCH /users

>PAYLOAD:
```json
{
  "username": "Dimon"
}
```

> RESPONSE 200 OK
```json
{
  "affectedCount": 1
}
```

All available users were updated and response contains the count of them. Also, server can apply request and response with 202 HTTP Status. It means that the request will be processed later:
> PATCH /users

>PAYLOAD:
```json
{
  "username": "Ivan"
}
```

> RESPONSE 202 OK

#### Get one:
> GET /users/afdf-3f4f-65fd

> RESPONSE 200 OK:
```json
{
  "id": "afdf-3f4f-65fd",
  "username": "Petya",
  "email": "petya@gmail.com"
}
```

Returns the entity by passed id in the URL and parameters(see below). The answer should be the same DTO which is used for PUT method.

#### Get many (search):
> GET /users?username=Petya&limit=1&offset=0

>RESPONSE 200 OK:
```json
{
  "totalCount": 21,
  "list": [
    {
      "id": "afdf-3f4f-65fd",
      "username": "Petya",
      "email": "petya@gmail.com"
    }
  ]
}
```
Finds the entities by passed filters and parameters(see below).

#### Delete one:
> DELETE /users/afdf-3f4f-65fd

> RESPONSE 200 OK

Deletes one entity by passed id. Also, the payload can be passed for additional information (if needed, for example: reason of removing).

#### Delete many:
> DELETE /users

> RESPONSE 200 OK

Deletes all entities. Also, the payload can be passed for additional information (if needed, for example: reason of removing).

## URL writing
The naming of entities must be in the snake_case and it must be a noun:
> GOOD /user_details

> BAD /user-details

But! The actions must be verbs and should follow camelCase. Also, every action must start from the 'do' word:
> GOOD /user_details/doRecalculateAddresses

> BAD /user_details/recalculateAddresses

> BAD /user_details/makeRecalculation

> BAD /user_details/do_recalculate_addresses

## Actions
The actions are the custom triggers which are used for triggering processes in the system.
But! Every action can be applied for one entity or for the set of entities:
> POST /users/doRecalculateAddresses - set entities

> POST /users/afdf-3f4f-65fd/doRecalculateAddresses - one entity

An action might contain information for processing. The information must be passed to the server via DTO in the payload.

The response might contain (in success case) next statuses:
* 200 - the request has been processed
* 202 - the request was accepted and will be processed later (async)

## Searching (GET ONE || GET MANY)

#### Additional query params
The endpoints for getting entities must support additional query params:
* fields - the description of the fields which are requested. It means that the server must return only the fields which are described in that param. For example:
> GET /users/afdf-3f4f-65fd?fields=email,username

> RESPONSE 200 OK:
```json
{
  "username": "Petya",
  "email": "petya@gmail.com"
}
```

>GET /users/afdf-3f4f-65fd?fields=id

> RESPONSE 200 OK:
```json
{
  "id": "afdf-3f4f-65fd"
}
```

* offset & limit & order params make sense only for searching in the set of entities.

offset - describes the offset for the set of results

limit - describes the limit of entities in the result set

order - describes how the results must be sorted

## Requests
The requests might be very different. Actually, every request must resolve some problem in the system. It means that you are free to change request DTOs as you need.

## Responses
The response must contain the DTO for answer except multi-result case. In the multi-result case, the response must contain next structure:
> GET /users

>RESPONSE 200 OK:
```json
{
  "totalCount": 1,
  "list": [
    {
      "id": "afdf-3f4f-65fd",
      "username": "Petya",
      "email": "petya@gmail.com"
    }
  ]
}
```

In other words, the response with multiple entities must contain next fields:
* totalCount - the count of entities which can be found by passed filters

Also, on PATCHing of entity set the response must contains on field - affectedCount. The field must contains the number of entities which have been updated by the patching.

## Sub-entities (relations, implicit, explicit)
In some cases, you might need to support relations between entities and display that in your API for usability. 

For example. You have an entity of user's posts which can exist only with a user (author). It means that you can't create a post without relation to the existing user. In this case, it would be better to provide only one API which explicitly shows that you should have the user already to create a new post:
> POST /users/afdf-3f4f-65fd/posts

> PAYLOAD:
```json
{
  "title": "I'm cool guy",
  "content": "I'm very cool guy, damn!"
}
```

> RESPONSE:
```json
{
  "id": "bmws-ucks-1234",
  "title": "I'm cool guy",
  "content": "I'm very cool guy, damn!",
  "user": {
    "id": "afdf-3f4f-65fd",
    "username": "Petya",
    "email": "petya@gmail.com"
  }
}
```

That example describes the common approach how to build the API for the relations. But, you don't have to follow it, if you have very small and\or non-logical tied relations. That approach is just a recommendation how to organize relations in your API.

The base CRUD operations and all controller method (API) of the sub-entity must be available via parent part of API and it means that all actions will be processed in the parent's context.

Also, you can build all methods and operations for sub-entities:

> GET /users/afdf-3f4f-65fd/posts

Returns all posts of the user the id of which you passed as root of the URL.

> DELETE /user/afdf-3f4f-65fd/posts

Delete all posts of the user.

> DELETE /user/afdf-3f4f-65fd/posts/bmws-ucks-1234

Delete only one post of the user by id.

And so on. You are free to build sub-entities API in accord to your business logic.

## Versions
If you API has to support versions, you should follow next simple approach. Just put the version after root of your API, starts with 'V' letter. For example:
> /api/v1/users

> /v2/users

## Errors
You must use HTTP Error Codes! There are main principle:
* 2** - request was processed
* 4** - request is bad (client error)
* 5** = request is good, but server made a mistake (server error)

Follow that principle and always return proper error code with detail describe of the error, for example:
```json
{
  "status": 400,
  "message": "The field can't be null!"
}
```

```json
{
  "status": 500,
  "message": "Exception while data recalculating!"
}
```

## SUMMARY
The main point of this document can be presented in the next few rules of building API:
* Follow naming rule: 
    * names of the entities must be in the snake_case and in the plural form
    * names of the actions must start from the 'do' word and be in the camelCase
* Build hierarchical API:
    * put name of the entity in the root
    * put sub-entities after the parent entity
    * follow principle: without ID is bulk action (on set), with ID is a action with one entity
* Use HTTP methods:
    * Use GET for getting and searching info
    * Use POST for creating new entity and trigger an process
    * Use PUT for updating
    * Use DELETE for deleting
* Use HTTP Error Codes
* Use versions (if needed)
* Follow the rules for responses
    * DTO for one-entity response
    * Special container DTO for list
* Use special params for getting
    * fields - for cut answer
    * limit - limit list of answers
    * offset - offset list of answers