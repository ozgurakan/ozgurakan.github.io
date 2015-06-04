---
layout: bootstrap_post
title: Asynchronous Operations in RESTful API
date: 2015-05-15 18:10:00
author: Oz Akan
abstract: Do you have long running tasks for which you don't want to block the web server or keep the consumer waiting? You might consider this method to implement asynchronous operations.
categories:
    - Development
---

This article demonstrates how to handle asynchronous operations in a RESTful API.

Let's imagine we have a factory that builds cars. You want to order a car and not want to wait while it is being build.

### Cars Collection [/cars]

So we have `/cars` collection which accepts `POST` to create a car. If successful, this request will return `202 - Accepted` response.

API won't wait the resource to be created. User will read the `Location` header and follow up that URI in order to get updates on the task. `Wait` header could be used to inform user how long to wait before the next query.

#### Create a Car [POST]

`Request`
        
    POST /cars     
    
    Content-Type: application/json
    
    {
        "brand": "Ford",
        "make": "Mustang",
        "model": 2014,
        "color": "Blue"
    }

`Response`

    202
    
    Content-Type: application/json
    
    Location: /tasks/9999
    Wait: 600

This is pretty nice though what if I want to use same approach for any request. Like, even for the `GET` requests which don't really create a new resource but usually just reads a resource from the database. These are not long running jobs in nature, or usually they start like short running tasks. 

### Car Resource [/cars/{car_id}]

With this approach `GET` for `/cars/{car_id}` resource won't return information about the car but instead will return another call-back URI as in the form of `Location` header.

#### Get Details of Car [POST]

`Request`

    GET /cars/9999


`Response`

    202
    
    Content-Type: application/json
    
    Location: /tasks/10001
    Wait: 0
    
Since `GET` will trigger a database read to return details about the car with the UID 9999, wait time could be `0` seconds as seen above.

### Tasks Collection [/tasks]

So it is obvious that `tasks` collection resource is very essential in this design. Let's start with listing all the tasks.


#### Get all tasks [GET]

Here we can implement filtering to list only runnning tasks etc. For the sake of the example, this lists all tasks.

`Request`

    GET /tasks


`Response`

    202
    
    Content-Type: application/json

        [
            {
                "task_id": 9999,
                "status": "active",
                "progress": 50,
                "eta": 600,
                "workflow": {
                    "wf1": "finished",
                    "wf2": "in progress",
                    "wf3": "not started"
                }            
            }, {
                "task_id": 9999,
                "status: "finished",
                "progress": 100,
                "eta": 0,
                "workflow": {
                    "wf1": "finished",
                    "wf2": "finished",
                    "wf3": "finished"
                }
            }, {
                "task_id": 10001,
                "status: "finished",
                "progress": 100,
                "eta": 0,
                "workflow": {
                    "wf1": "done"
                }
            }
        ]
        
## Task API [/tasks/{task_id}]

- `202` response, is for a task that is in progress, `location` header points itself.
- `201` response, is for a task that is completed, response has `location` header set, pointing the resource.
- `200` response, is for a task which is completed, but didn't create a new resource. As it was in  `GET /cars/99999` request.


### Get details of a task [GET]


#### Task is still running

`Request`

    GET /tasks/9999


`Response`

    202
    
    Content-Type: application/json

    Headers
    
            Location: /tasks/9999
    
    Body

            {
                "task_id": 9999,
                "status": "active",
                "progress": 50,
                "eta": 600,
                "workflow": {
                    "wf1": "finished",
                    "wf2": "in progress",
                    "wf3": "not started"
                }            
                "request": {
                    "resource": "/cars/",
                    "verb": "POST",
                    "header": {},
                    "body": {
                        "brand": "Ford",
                        "make": "Mustang",
                        "model": 2014,
                        "color": "Blue"
                    }
                },
                "result": {}
            }


#### New resource created

If task has been finished and a new resource is created.

`Request`

    GET /tasks/9999


`Response`

    201
    
    Content-Type: application/json
    
    Headers
        
            Location: /cars/1234

    Body

            {
                "task_id": 9999,
                "status: "finished",
                "progress": 100,
                "eta": 0,
                "workflow": {
                    "wf1": "finished",
                    "wf2": "finished",
                    "wf3": "finished"
                }
                "request": {
                    "resource": "/cars/",
                    "verb": "POST",
                    "header": {},
                    "body": {
                        "brand": "Ford",
                        "make": "Mustang",
                        "model": 2014,
                        "color": "Blue"
                    }
                },
                "result": {
                    "car_id": 1234,
                    "brand": "Ford",
                    "make": "Mustang",
                    "model": 2014,
                    "color": "Blue"
                }
            }



#### No new resource, just data

If the task has been completed, no new resource has been created but there is data to be returned.

`Request`

    GET /tasks/9999


`Response`

    200
    
    Content-Type: application/json
    
        {
            "task_id": 10001,
            "status: "finished",
            "progress": 100,
            "eta": 0,
            "workflow": {
                "wf1": "done"
            }
            "request": {
                "resource": "/cars/1234",
                "verb": "GET",
                "header": {},
                "body": {}
            },
            "result": {
                "car_id": 1234,
                "brand": "Ford",
                "make": "Mustang",
                "model": 2014,
                "color": "Blue"
            }
        }

Above, request section shows that this task was actually a `GET /cars/1234` request.

## Is This Efficient?

Does it make make sense to create a task even for a database operation? 

Maybe yes, if `tasks` resource is populated quickly and most of the tasks are long running for sure this makes sense. It also lets you change a short running task to a long running one without affecting the application that consumes the API. You can think of a report which takes a second to create. In time when data grows it may take an hour to complete that report. If you were using the design described above, it wouldn't be a big problem for a client which implemented `tasks`.

### Async vs Sync Operations Examples

You can find more details on the API below;

#### Sync Example
[http://docs.syncrestapi.apiary.io/](http://docs.syncrestapi.apiary.io/)

#### Async Example
[http://docs.asyncrestapi.apiary.io/](http://docs.asyncrestapi.apiary.io/)