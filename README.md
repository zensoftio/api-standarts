# API Specification

The description of the common points for the API development.

## Base CRUD operations
The URLs for the basic CRUD operations must start from the entity's name in the plural form. 
The URL in that form describes operations for the all entities. For example:
> /users

It means that all requests on this URL affect the entities with written name.

The URLs witch affect only one entity must contain ID of the entity in the URL after name:
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
"username" : "Vasya",
"email": "vasya@gmail.com"
}
```

> RESPONSE 200 OK:
```json
{
"object": "user",
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
"username" : "Petya",
"email": "petya@gmail.com"
}
```

> RESPONSE 200 OK:
```json
{
"object": "user",
"id": "afdf-3f4f-65fd",
"username": "Petya",
"email": "petya@gmail.com"
}
```

Updates the entity by id in the URL. The DTO can be different.

#### Get one:
> GET /users/afdf-3f4f-65fd?param=param1

> RESPONSE 200 OK:
```json
{
"object": "user",
"id": "afdf-3f4f-65fd",
"username": "Petya",
"email": "petya@gmail.com"
}
```

Returns the entity by passed id in the URL and parameters(see below). The answer should be the same DTO which is used for PUT method.

#### Get many (search):
> GET /users?filter1=param

>RESPONSE 200 OK:
```json
{
"object": "list",
"total": 1,
"offset": 0,
"limit": 1,
"result": [
    {
        "object": "user",
        "id": "afdf-3f4f-65fd",
        "username": "Petya",
        "email": "petya@gmail.com"
     }
]
}
```
Finds the entities by passed filters and parameters(see below).

#### Delete one:
> DELETE /users/afdf-3f4f-65fd?param=param1

> RESPONSE 200 OK

Deletes one entity by passed id.

#### Delete many:
> DELETE /users

> RESPONSE 200 OK

Deletes all entities.

## Actions
The actions are the custom triggers which are used for triggering processes in the system.
But! Every action can be applied for one entity or for the set of entities:
> POST /users/doRecalculateAddresses - set entities

> POST /users/afdf-3f4f-65fd?param=param1/doRecalculateAddresses - one entity

An action might contain information for processing. The information must be passed to the server via DTO in the payload. 

## URL writing
The naming of entities must be in the snake case and it must be a noun:
> GOOD /user_details

> BAD /user-details

But! The actions must be verbs and should follow SnakeCase. Also, every action must start from the 'do' word:
> GOOD /user_details/doRecalculateAddresses

> BAD /user_details/recalculateAddresses

> BAD /user_details/makeRecalculation

> BAD /user_details/do_recalculate_addresses

## Searching (GET ONE || GET MANY)

####Additional query params
The endpoints for getting entities must support additional query params:
* fields - the description of the fields which are requested. It means that the server must return only the fields which are described in that param. For example:
> GET /users/afdf-3f4f-65fd?fields=email,username

> RESPONSE 200 OK:
```json
{
"object": "user",
"username": "Petya",
"email": "petya@gmail.com"
}
```

>GET /users/afdf-3f4f-65fd?fields=id

> RESPONSE 200 OK:
```json
{
"object": "user",
"id": "afdf-3f4f-65fd"
}
```

* offset & limit - it makes sense only for searching in the set of entities.

offset - describes the offset for the set of results

limit - describes the limit of entities in the result set

## Requests
The requests might be very different. Actually, every request must resolve some problem in the system. It means that you are free to change request DTOs as you need.

There is one exception: every response must contain the special field which is called 'object'. The field must determine the 'name' of the current object.

## Responses
The response must contain the DTO for answer except multi-result case. In the multi-result case, the response must contain next structure:
> GET /users

>RESPONSE 200 OK:
```json
{
"object": "list",
"total": 1,
"offset": 0,
"limit": 1,
"result": [
    {
        "object": "user",
        "id": "afdf-3f4f-65fd",
        "username": "Petya",
        "email": "petya@gmail.com"
     }
]
}
```

In other words, the response with multiple entities must contain next fields:
* total - the count of entities which can be found by passed filters
* offset - offset which has been applied for the current result set
* limit - limit which has been applied for the current result set
