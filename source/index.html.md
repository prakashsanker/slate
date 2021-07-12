---
title: Climes API v0.1.0

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true
---

# Introduction

Our API has predictable resource-oriented URLs, accepts [form-encoded](https://en.wikipedia.org/wiki/POST_(HTTP)#Use_for_submitting_web_forms) request bodies, returns [JSON-encoded](http://www.json.org/) responses, and uses standard HTTP response codes, authentication, and verbs.

You can use the Climes API in test mode, which does not affect your live data or interact with the banking networks. When you sign up for climes you will be provided a test key and a live key - the data for both of these is separate. The data in production is not visible if you are using your test key and vice versa. 


# Authentication

Climes uses API keys to authenticate and these keys can be found in your Climes dashboard.

Climes exclusively uses Bearer authentication to validate API requests. 

```shell
    curl --request GET \
      --url 'https://api.climes.io/v1/projects?page=1' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```

## Errors

When processing errors, Climes will return a JSON object with a code, a message and relevant data about that error.


    {
      code: "401",
      message: "....",
      data: {} // specific to the object of the request
    }


| Status Code | Meaning                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 200         | Response OK                                                                                                                      |
| 201         | Successfully created a resource (for example a transaction)                                                                      |
| 401         | Unauthorized: the API key that you provided is not valid                                                                         |
| 404         | The resource that you requested cannot be found or does not exist.                                                               |
| 406         | Not Acceptable: You set the `Accept` header to something other than `null` or `application/json`.                                |
| 415         | Unsupported Media Type: You submitted non-JSON data to the api. Remeber to set your `Content-Type` header to `application/json`. |
| 422         | Unprocessible Entity: You submitted invalid data. The API will respond with which data validations failed.                       |
|             |                                                                                                                                  |


# Api Use Cases

## Purchasing an offset

Purchasing an offset is easy. There are two scenarios -


1. You know exactly how much carbon you want to offset.
2. You are trying to **do something.** For example, you want to offer an integration to help the customers of airlines offset their carbon footprint by buying an offset to cover a specific journey.

### Direct Purchase

```shell
    curl --request POST \
      --url 'https://api.climes.io/v1/offset_order' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json' \
      -- data '{ mass: 1000, projects: [project_id] }'
```

In this case you can use the following to create an order based on the amount of carbon that you want to offset. 

This is a less useful case, so we suggest that you leverage our estimation API.  Note that you can pass back a **specific project id** in order to buy offsets from that specific project. 



## Purchasing via an estimate

### Flight Transportation

```shell
    curl --request POST \
      --url 'https://api.climes.io/v1/offset_order/flight' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json' \
      --data '{ \
          "flight_number": "KL123",
          "departure_date": "12/07/21 00:00:00"
        }'
```

To purchase an offset for a flight, simply pass back the flight number and the departure date. We will calculate the carbon offset required and make a purchase on your behalf. 

<br />
<br />
<br />

### Road Transportation

```shell
    curl --request POST \
      --url 'https://api.climes.io/v1/offset_order/vehicle' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json' \
      --data '{ \
          "model": "Honda",
          "make": "Activa",
          "year": 2019,
          "distance": 2000
        }'
```

To purchase an offset for a road trip you can pass back the distance in kilometers of the trip and we will calculate an offset. 

To fine-tune this, you can also pass back the `model`, `make` and `year` of the vehicle so we can calculate a more accurate offset.



> This will return an `Order`. The `Order` looks like this - 


```

    {
      id: UUID,
      quantity_purchased: Number,
      price_cents_usd: Number // the total cost of the order,
      fulfillments: Array<Fulfilment>,
      status: 'placed' | 'fulfilled' | 'canceled'
    }
```

<br />
<br />
<br />
<br />


An order is `fulfilled` when we have been able to make the purchase of the offset. This could take between 1 hour and a couple of weeks.

> As we might take offsets from multiple projects, fulfillments look like this -

```
    {
      id: UUID,
      order_id: UUID,
      project_id: UUID,
      mass: Number,
      price_in_cents: Number
    }
```

And an order can have multiple fulfillments.


# API Reference

You will generally use the above workflow to purchase carbon offsets. However, we also detail the full power of the API below.

## Orders


In all cases, a request to the Orders endpoint returns an `Order` object. 

### Getting back an order

```shell
    curl --request GET \
      --url 'https://api.climes.io/v1/offset_order/{offset_order_id}' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```

To get back an order, simply pass back the order id.

<br />
<br />

<br />

### Listing orders

```shell
    curl --request GET \
      --url 'https://api.climes.io/v1/offset_order/' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```

You may also pass back - `status` to pull back orders by status. 

<br />
<br />
<br />

### Canceling an order

```shell
    curl --request PATCH \
      --url 'https://api.climes.io/v1/offset_order/{offset_order_id}/cancel' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```

## Projects

A Project is a source of carbon offsets. You can’t do much with Projects, except display them in order to help your customers understand what each project does. 

> This is what a Project looks like

```shell
    {
      id: uuid,
      name: string,
      description: string,
      type: 'class-a' | 'class-b',
      address: string,
      longitude: number,
      langitude: number,
      year_created: Date,
      verification_type: "REDD" | "REDD+" //add more here,
      total_carbon_offset_quantity: Number // details the total mass of carbon offsets that this project can generate,
      current_carbon_offset_quantity: Number // the offsets remaining in this project,
      photos: Array<String>,
      price_per_offset_usd: Number
    }
```

### Getting back a Project

```shell
    curl --request GET \
      --url 'https://api.climes.io/v1/project/{project_id}' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```

To get back the details of a specific project, simply pass back the project id.

### Listing projects

```shell
    curl --request GET \
      --url 'https://api.climes.io/v1/project/' \
      --header 'Authorization: Bearer <API KEY>' \
      --header 'Content-Type: application/json'
```
