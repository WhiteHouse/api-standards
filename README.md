# White House Web API Standards

* [Guidelines](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [URLs](#urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error handling](#error-handling)
* [Versions](#versions)
* [Record limits / Pagers](#limits-pagers)
* [Request & Response Examples](#request-response-examples)
* [Mock Responses](#mock-responses)

## <a id="guidelines"></a>Guidelines

This document provides guidelines and examples for White House Web APIs, encouraging consistency, maintainability, and best practices across applications. White House APIs aim to balance a truly RESTful API interface with a positive developer experience (DX).

This document borrows heavily from:
* [Designing HTTP Interfaces and RESTful Web Services](http://munich2012.drupal.org/program/sessions/designing-http-interfaces-and-restful-web-services)
* API Facade Pattern, by Brian Mulloy, APIGee
* Web API Design, by Brian Mulloy, APIGee
* [Fieldings Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

## <a id="pragmatic-rest"></a>Pragmatic REST

These guidelines aim to support a truly RESTful API. Here are a few exceptions:
* Put the version number of the API in the URL (see examples below). Don’t accept any requests that do not specify a version number.
* Allow users to request formats like JSON or XML like this:
    * http://whitehouse.gov/api/v1/blogs.json
    * http://whitehouse.gov/api/v1/blogs.xml

## <a id="urls"></a>RESTful URLs

### General guidelines for RESTful URLs
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Use plural nouns only for consistency (no singular nouns).
* Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* You shouldn’t need to go deeper than resource/identifier/resource.
* Put the version number at the base of your URL, for example http://example.com/v1/path/to/resource.
* URL v. header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * If it doesn’t change the logic for each response, like OAuth info, put it in the header.
* Specify optional fields in a comma separated list.
* Formats should be in the form of api/v2/resource/{id}.json

### Good URL examples
* List of blog posts:
    * http://www.whitehouse.gov/api/v1/blogs.json
* Filtering is a query:
    * http://www.whitehouse.gov/api/v1/blogs.json?year=2011&sort=desc
    * http://www.whitehouse.gov/api/v1/blogs.json?topic=economy&year=2011
* A single blog post in JSON format:
    * http://www.whitehouse.gov/api/v1/blogs/1234.json
* All photos (belonging to this blog post):
    * http://www.whitehouse.gov/api/v1/blogs/1234/photos.json
* Specify optional fields in a comma separated list:
    * http://www.whitehouse.gov/api/v1/blogs/1234.json?fields=title,by,date
* Get all signatures to a particular petition in XML format:
    * GET http://petitions.whitehouse.gov/api/v1/petitions/1234/signatures.xml
* Create a new signature for a particular petition:
    * POST http://petitions.whitehouse.gov/api/v1/petitions/1234/signatures

### Bad URL examples
* Non-plural noun:
    * http://www.whitehouse.gov/blog
    * http://www.whitehouse.gov/blog/1234
    * http://www.whitehouse.gov/photos/blog/1234
* Verb in URL:
    * http://www.whitehouse.gov/photos/blog/1234/create
* Filter outside of query string
    * http://www.whitehouse.gov/blogs/2011/desc

A good rule of thumb is, there should only be two URLs per resource. For example:
* http://www.whitehouse.gov/blogs.json
* http://www.whitehouse.gov/blogs/12345.json

## <a id="http-verbs"></a>HTTP Verbs

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

@todo Example from Web API Design


## <a id="responses"></a>Responses
@todo Add good v. bad examples below for:
* no values in keys
* no internal-specific names (e.g. "node" and "taxonomy term")
* metadata should only contain direct properties of the response set, not properties of the members of the response set


## <a id="error-handling"></a>Error handling

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined error number), links where developers can find more info. For example:

    {
      "status" : "400",
      "developerMessage" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "userMessage" : "This is a message that can be passed along to end-users, if needed.",
      "errorCode" : "444444",
      "more info" : "http://www.whitehouse.gov/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }

Use three simple, common response codes indicating (1) success, (2) failure due to client-side problem, (3) failure due to server-side problem:
* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error


## <a id="versions"></a>Versions

* Never release an API without a version number.
* Versions should be integers, not decimal numbers, prefixed with ‘v’. For example:
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* Maintain APIs at least one version back.


## <a id="limits-pagers"></a>Record limits / Pagers

* If no limit is specified, return results with a default limit.
* To get records 50 through 75 do this:
    * http://whitehouse.gov/examples?limit=25&offset=50
    * offset=50 means, ‘begin with record number fifty’
    * limit=25 means, ‘return 25 records’

Information about record limits should also be included in the Example resonse. Example:

    {
        "metadata": {
            "resultset": {
                "count": 50,
                "offset": 50,
                "limit": 25
            }
        },
        "results": [
            { .. }
          ]
        }
    }

## <a id="request-response-examples"></a>Request & Response Examples

###Press Articles
Retrieve – GET  whitehouse.gov/api/v1/press-articles/[id].json

    {
        "id": "1234",
        "title": "Example Title",
        "tags": [
            {"id": "125", "name": "Firemen"},
            {"id": "834", "name": "Santa Claus"}
        ],
        "created": "1231621302"
    }

Index – GET  whitehouse.gov/api/v1/press-articles

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": 2457,
                "type": "press_article",
                "title": "Example Title",
                "tags": [
                    {"id": "125", "name": "Firemen"},
                    {"id": "834", "name": "Santa Claus"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "press_article",
                "title": "Example Title Two",
                "tags": [
                    {"id": "125", "name": "Firemen"},
                    {"id": "834", "name": "Santa Claus"}
                ],
                "created": "126251302"
            }
        ]
    }


###Users
Retrieve – GET  whitehouse.gov/api/v1/users/[uid]

    {
        "type": "user",
        "location": {
            "city": "Washington",
            "state": "DC",
            "zip": "20006"
        },
        "created": "1231621302"
    }

Index – GET  whitehouse.gov/api/v1/users

    {
        "metadata": {
            "execution time": "43",
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "type": "user",
                "location": {
                    "city": "Washington",
                    "state": "DC",
                    "zip": "20006"
                },
                "created": "1231621302"
            },
            {
                "type": "user",
                "location": {
                    "city": "Washington",
                    "state": "DC",
                    "zip": "20006"
                },
                "created": "126251302"
            }
        ]
    }


###Petitions
Retrieve – GET  petitions.whitehouse.gov/api/v1/petitions/[pid]

    {
        "id": 241237,
        "type": "petition",
        "title": "Example Title",
        "tags": [
            {"id": 175, "name": "Civil Rights and Liberties"},
            {"id": 156, "name": "Health Care"}
        ],
        "signature count": 104512,
        "signatures needed": 0,
        "deadline": 0,
        "created": "1231621302"
    }

Index – GET  petitions.whitehouse.gov/api/v1/petitions?created=12-17-2012

##Signatures
Create – POST  petitions.whitehouse.gov/api/v1/petitions/[pid]/signatures

    {
        "email": "drupal@acquia.com",
        "first_name": "Drupal",
        "last_name": "Isto",
        "zip": 20006
    }

Retrieve – GET  petitions.whitehouse.gov/api/v1/petitions/[pid]/signatures/[sid]

    {
        "id": 2411,
        "type": "signature",
        "location": {
            "city": "Washington",
            "state": "DC",
            "zip": "20006"
        },
        "created": "1231621302"
    }

Index – GET  petitions.whitehouse.gov/api/v1/petitions/[pid]/signatures

    {
        "metadata": {
            "execution time": "241",
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": 2411,
                "type": "signature",
                "location": {
                    "city": "Washington",
                    "state": "DC",
                    "zip": "20006"
                },
                "created": "1231621302"
            },
            {
                "id": 2412,
                "type": "signature",
                "location": {
                    "city": "Washington",
                    "state": "DC",
                    "zip": "20006"
                },
                "created": "1231621302"
            }
        ]
    }

### Parameters
* createdAt=1355787118
* createdBefore=1355787118
* createdAfter=1355787118

# <a id="mock-responses"></a>Mock Responses
It is suggested that each resource accept a 'mock' parameter on the testing server. Passing this parameter should return a mock data response (bypassing the backend).

Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

Note: if the mock parameter is included in a request to the production environment, an error should be raised.
