---
title: Hexoskin API Reference

toc_footers:
  - <a href='http://www.hexoskin.com/'>Hexoskin website</a>
  - <a href='http://www.hexoskin.com/pages/developers'>Sign Up for a Developer Key</a>
  - <a href='https://api.hexoskin.com/docs/page/terms-and-conditions/'>Terms and conditions</a>
  - <a href='https://groups.google.com/forum/?hl=fr#!forum/hexoskin-developer'>Developer Forum</a>

includes:

search: true
---
# Introduction

```
  _    _                    _    _
 | |  | |                  | |  (_)
 | |__| | _____  _____  ___| | ___ _ __  
 |  __  |/ _ \ \/ / _ \/ __| |/ / | '_ \
 | |  | |  __/>  < (_) \__ \   <| | | | |
 |_|  |_|\___/_/\_\___/|___/_|\_\_|_| |_|

This column will contain helpful example queries and responses. Neat, huh?
```

This is the **Hexoskin REST API** reference Wiki.

The **Hexoskin REST API** allows you to interact and manipulate your Hexoskin data through HTTP requests. This includes accessing biometrics and user information (both yours and your friend's), but also annotating data, viewing advanced reports, fetching metrics, and more. Follow along as we explore the possibilities, or if you're up to it, jump straight to the resources you want to access.

## Before getting started

### Getting an API key

First, you'll need an API key. If you don't already have one, email us at <a href="mailto:api@hexoskin.com">api@hexoskin.com</a> with your project name and a quick description of what you want to do, and we'll happily create you one.

### So... What's a REST API?

If you are new to using APIs, you're at the right spot. You'll need to understand the concept of REST API before you can do anything. Reading this will help, but you will probably have a little homeworks to do before it all works out perfectly. If you already know what we are talking about, great! Jump ahead to the Hexoskin REST API Overview section to get started.

For those still here, a REST API is of a way to structure data so you can access it. A website like [http://hexoskin.com](http://hexoskin.com) is something you're most likely already comfortable with. It's a page that contains some information. To go to the developer section of the website, you can obviously click on a few button, but you can also add [/pages/developers](http://hexoskin.com/pages/developers) to the previous URL, and you'll end up in the developer section. You just went from the general website to the developer-oriented page.
A REST API works in a very similar fashion. The entry point of the REST API is [https://api.hexoskin.com/api/v1/account/](https://api.hexoskin.com/api/v1/) , and appending information at the end will bring you further down the rabbit hole. For example, asking for [https://api.hexoskin.com/api/v1/account/](https://api.hexoskin.com/api/v1/account/) will return you your own account's information. Try it in your web browser! It'll ask you to identify yourself, because our API needs authentication. Just use your Hexoskin account as credentials. The information is returned as a JSON object, a standard format for such kind of information.

Something very neat thing about REST APIs is that you can add arguments that will modify what the REST API returns to you. For example, take [https://api.hexoskin.com/api/v1/trainingroutine/](https://api.hexoskin.com/api/v1/trainingroutine/). The response you receive will contain all training routines available to perform. But what if you want to look at the training routines that you have used? Well, looking at the trainingroutine documentation ([https://api.hexoskin.com/docs/resource/trainingroutine/](https://api.hexoskin.com/docs/resource/trainingroutine/)), you see that there is a filter just for that. You can then query [https://api.hexoskin.com/api/v1/trainingroutine/?using=True](https://api.hexoskin.com/api/v1/trainingroutine/?using=True), and voilà, you have used a filter to modify what the REST API returns to you.
One final thing to mention : While you can do a lot from the web browser, there is a much bigger universe that you can access using some programming utility. Your web browser will allow you to GET information from the REST API, but using a programming language will allow you to POST information, and even PATCH some resources that you want modified. The Hexoskin REST API Overview documentation will use the cURL tool ([http://curl.haxx.se/](http://curl.haxx.se/)) to demonstrate the REST API functionality, but you can pretty much always find an equivalent in your favorite programming language.

# Getting started

## Overview and Technology

The API is built using Django and Tastypie. As such, the methods of querying the API follow the implementation patterns of Tastypie.

We have attempted to conform to RESTful practices as much as is practical. There are a few instances where a judicious departure has been indulged but overall you can assume things will work RESTfully.

### Making Requests

All requests are conducted over SSL. Any non-SSL requests received are forwarded to HTTPS. All requests must be signed with a valid API key. All requests must be authenticated via basic auth with the exception of createuserrequest.

> By default, all responses are returned as JSON. However, you can set your headers manually. The two accepted headers for data format are:

```
    Content-Type: application/json
    Content-Type: application/octet-stream
```

Currently, only JSON is supported for non-data resources. For data resources, JSON and octet-stream are supported, though JSON have limitations (see [data](data)). Set your headers accordingly.

On the whole, interactions with the API behave as described in Tastypie's documentation. A couple of notable restrictions are:

DELETE and PUT requests to list views are always denied.
PUT to a detail view of a non-existant instance will not attempt to create the instance, it will result in a 404.
POST requests to detail views are always denied.
Otherwise, things should work more-or-less as expected.


> An example POST request would look something like this :

```
POST api.hexoskin.com/api/v1/range/
From: mailer@mail.com
User-Agent: HTTPTool/1.0
Content-Type: application/json
X-HEXOTIMESTAMP: 1234567890
X-HEXOAPIKEY: 3X4mP1ePu6licK3y
X-HEXOSAPISIGNATURE: 3x4mPl3ShA15igN4tUr33x4mPl3ShA15igN4tUr3
```
```json
{"name":"NewRange",
 "user":555,
 "start":369592079271,
 "end":369592079271}
```
When performing any kind of request, they have to be signed with your public API key, the timestamp, and the URL. Your API key has a public and a private portion. Never send the private key, only use it to sign the request. Send the public key.

To use the keys, you add three headers to each request:

* `X-HEXOTIMESTAMP`: a normal UTC timestamp.
* `X-HEXOAPIKEY`: your public key.
* `X-HEXOAPISIGNATURE`: the signature.

The signature is a SHA1 hash of the the private key, timestamp, and the URL (the entire URL, including the GET arguments). The order, of course, is very important. To help you remember, it's alphabetic: Key, Timestamp, Url.


## Retrieving Resources

### List views

> A typical ist view, this one taken from https://api.hexoskin.com/api/v1/datatype/ looks like:

```
{
    "meta": {
        "limit": 20,
        "next": "/api/v1/datatype/?limit=20&offset=40",
        "offset": 20,
        "previous": "/api/v1/datatype/?limit=20&offset=0",
        "total_count": 54
    },
    "objects": [
        {
            "dataid": 208,
            "freq": null,
            "info": "Recorder start annotation.",
            "name": "RECSTR_ANNOT_CHAR",
            "resource_uri": "/api/v1/datatype/208/"
        },
        {
            "dataid": 209,
            "freq": null,
            "info": "Recorder stop annotation.",
            "name": "RECSTP_ANNOT_CHAR",
            "resource_uri": "/api/v1/datatype/209/"
        },
        ...
    ]
}
```
To get a list of resources, you make a GET request to the resource's endpoint. For instance, to retrieve all the datatypes you would make the following request:

`curl -i https://api.hexoskin.com/api/v1/datatype/`

Resources are represented as URIs. URIs generally conform to the following model:

`/api/[API version]/[resource name]/`

For example, all the datatypes are found by issuing a GET request to:

`/api/[API version]/[resource name]/[ID]`

This is a "list view" because it will return a list of all the instances of that resource that the current user is allowed to see.

Note that the resource name is singular and there is a trailing slash.

Requests to lists return an object containing two members; meta containing information about the results and objects which are a list of the results. A sample result from `/api/v1/datatype/?offset=20` is shown on the right:

As you can see, the meta object tells us how to page through the data. You can roll your own paging routines of course, but you may also use the next and previous attributes of the meta object. The limit and offset variables are passed in the GET to determine which page to retrieve.

### Filtering and Ordering

Variables for filtering and ordering the results are also passed through the GET arguments. You can filter on any combination of fields and filter types in the Filtering Options of a resource. Refer to the 'Filtering Options' section of endpoint's documentation to determine what these are for that resource. For example, records support the following filtering options:

* device: exact, in
* start: exact, range, gt, gte, lt, lte
* end: exact, range, gt, gte, lt, lte
* user: exact, in

You use a filter type by passing a GET argument of `[column]__[filtertype]=[value]`. So to view a list of records that contains only records that started on or after a startTimestamp of 347631277170, you would append `startTimestamp__gte=347631277170` to the URL like this:

`https://api.hexoskin.com/api/v1/record/?startTimestamp__gte=347631277170`

If no filtertype is specified, exact is assumed. so `user__exact=/api/v1/user/99` is the same as `user=/api/v1/user/99`.

Ordering works in a similar fashion. Columns which support ordering are listed in each endpoints' documentation under 'Sorting Options'. You may pass `order_by=` followed by a comma-separated list of columns. The sort order is ascending by default, to make the sort order descending prepend the column name with a `-`. So to add a descending order to our record query, you would make the following request:

`https://api.hexoskin.com/api/v1/record/?startTimestamp__gte=347631277170&order_by=-startTimestamp`

###Detail views

>A typical detail view, this one taken from https://api.hexoskin.com/api/v1/datatype/208/, looks like :

```json
{
    "dataid": 208,
    "freq": null,
    "info": "Recorder start annotation.",
    "name": "RECSTR_ANNOT_CHAR",
    "resource_uri": "/api/v1/datatype/208/"
}
```
You may query a specific instance of a resource by accessing its URI. You are strongly encouraged to use the resource_uri attribute that is provided with each resource, however if you prefer to create them, they generally follow the pattern of the list URI with the ID appended:

`/api/[API version]/[resource name]/[ID]`

So to access the Recorder start annotation datatype directly, you would make a GET request to:

`https://api.hexoskin.com/api/v1/datatype/208/`

Note that the detail view does not contain a meta and object attribute, the resource instance is returned directly.


## Create, Update and Delete

Creating an instance of a resource is achieved by issuing a POST or a PATCH request to the list view of the desired resource. If the instance is created is successfully, you will receive a 201 with the location of the new resource in the Location header. For many resources, the instance will be in the body but sometimes, primarily in cases where there is a lot of data involved, the instance representation is omitted.

Not all resources support PATCH requests and some don't support creation at all! Please refer to the documentation for each resource to see what methods are supported and which fields must be supplied.

To update an instance, you may send a PUT or a PATCH to the URI of the resource instance you wish to update. As with creation, support for different methods vary between resources, be sure to check the docs.

If you use the PATCH method, it is not necessary to supply the entire object, you may supply only the fields you wish to change. If you use the PUT method you must provide a complete object to replace the existing one.

Deleting, as one would expect, is done by sending a DELETE request to the resource instance's URI.

## Cached Content

The API caches results to improve performance. If your are receiving a cached response, the X-HexoCache header will be present and will contain a timestamp (normal timestamp, not a HexoTimestamp) of the time it was cached.

## Error Codes

### 400 Bad Request

There is a problem with the data or the format of the data supplied.

### 401 Unauthorized

Your authentication failed.

### 403 Forbidden

Your user is not allowed to do what your are trying to do.

### 404 Not Found

The resource does not exist or you are not allowed to see the resource instance requested.

### 405 Method Not Allowed

You have attempted to use a method which is not allowed on the given resource.

### 500 Server Error

The API has encountered an error.


# Hexoskin REST API Cookbook

This section contains example recipes to display some functionalities of the REST API. But first, a quick introd

## Hexoskin ecosystem overview
At this point, you might have some data collected or are looking at the demo account, and are wondering: What are the different building blocks of this API and how do they interact together? Is there anything important that I should know before getting into this? Before getting in the detailed filters and query descriptions, this section will give you a quick overview of the Hexoskin ecosystem.

For starters, to find all hexoskin-collected data you have to specify a timestamp range and a user. When a user connects his hexoskin to the shirt, a Record is created. A Record consists, among others, of a user, and a start and end timestamp. Because of that, when asking for data, you can either query directly by timestamp, or instead ask for a record's data directly. Since records are always created over data, you should never find any data outside of a record, with the exception of GPS data, since it comes from the mobile phone directly. To find data, you can also go through Ranges, which will be covered a bit later.

All data is separated in datatypes. Each datatype has it's own id, and when accessing data, you pass a list of datatype ids in the request.

## Basic interaction with the API
All requests to the API are account-protected. As such, you always need to provide a basic authentication. With the cURL tool, a simple request should look like this

> https://api.hexoskin.com/api/v1/account/

```json
{
    "meta": {
        "limit": 1,
        "next": null,
        "offset": 0,
        "previous": null,
        "total_count": 1
    },
    "objects": [
        {
            "email": "athlete@hexoskin.com",
            "first_name": "Athlete",
            "id": 1511,
            "is_staff": false,
            "last_name": "Hexoskin",
            "profile": {
                "date_of_birth": "1980-01-01",
                "fitness": {
                    "fitness_percentile": 50,
                    "height": 1.82,
                    "hr_max": 183,
                    "hr_recov": 0,
                    "hr_rest": 73,
                    "vo2_max": 42,
                    "weight": 78
                },
                "gender": "M",
                "height": 1.82,
                "id": 1460,
                "preferences": null,
                "resource_uri": "/api/v1/profile/1460/",
                "unit_system": "metric",
                "weight": 78
            },
            "resource_uri": "/api/v1/user/1511/",
            "username": "athlete@hexoskin.com"
        }
    ]
}
```

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/account/```

The response is issued in JSON format.
Soon, requests will also require to be signed, but that will be covered when it becomes necessary.
Accessing data
Because Hexoskin records lots of data, most of the information is neatly organised in separate datatypes that you can query individually. You can view a list of all existing datatypes by using :

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/datatype/```


By default, the datatype endpoint returns a maximum of 20 objects. Some resources like this one are limited in how many objects they return using the default query. If you want to see a different subset, you can apply filters, change the limit flag, or use an offset to view the rest (see Applying filters).
When you know which datatypes you need, you need to decide for which time frame you want them. To do so, you can pass a range or record. To get a list of records or ranges, you can call :

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/record/```

OR

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/range/```

The difference between a range and a record is very simple : A record represents a time period containing data, while a range is a time annotation that describes what activity was being performed.
So now that you know what data you want, and for which record or range, you can query as follow, using the ids of the resources you want :

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/data/?datatype__in=19,33&record=37700```

The previous query will return the datatype ids 19 and 33 (heart rate and breating rate) for record 37700.

Note that all data is limited to return a maximum of 65535 points. If the time range you are asking for would contain more points (such as 257 seconds or more of 256Hz EKG, for example), the data returned is subsampled appropriately to return the max number of data points.
Metrics
Along with the actual data from the hexoskin, metrics are also available. Metrics are a way to summarize data in a single value or a histogram. A few example of metrics are heart rate average, max activity, minimum breathing rate and so on.

You can access the list of metrics the same way you access datatypes :

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/metric/```

Then, when you have your own list of favorite metrics, you can query

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/report/?include_metric=1,3&record=37700```

###Applying filters
For many resources, you can mix and match filters to help you find more detailed or specific information. For example, say you are a coach, and you want to get the list of records for a specific athlete. The unfiltered [/api/v1/records/](http://api.hexoskin.com/api/v1/records/) query will return you all your athlete's recordings, plus your own.
To receive only your desired athlete's records, you would then go for something along the lines of

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/record/?user=1511```

And of course, you knew his user ID by looking up from

```curl -u athlete@hexoskin.com:hexoskin https://api.hexoskin.com/api/v1/user/```

Look for the available list of filters in each resource's documentation.
