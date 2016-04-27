# Biomatters Web API Standards

* [Guidelines](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error handling](#error-handling)
* [Versions](#versions)
* [Record limits](#record-limits)
* [Request & Response Examples](#request--response-examples)
* [Mock Responses](#mock-responses)

## Guidelines

This document provides guidelines and examples for Biomatters Web APIs, encouraging consistency, maintainability, and best practices across applications. Biomatters APIs aim to balance a RESTy API interface with a positive developer experience.

These standard are derived from the [White House' Web API standards](https://github.com/WhiteHouse/api-standards).

Only support JSON (for now at least).  If we ever support different content types, use the `Accept` HTTP header.

This document borrows heavily from:

* [Designing HTTP Interfaces and RESTful Web Services](https://www.youtube.com/watch?v=zEyg0TnieLg)
* [API Facade Pattern](http://apigee.com/about/resources/ebooks/api-fa%C3%A7ade-pattern), by Brian Mulloy, Apigee
* [Web API Design](http://pages.apigee.com/web-api-design-ebook.html), by Brian Mulloy, Apigee
* [Fielding's Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

## URL paths

### General guidelines for URL paths
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Resources type names with multiple words should be separated with dashes. No camel case or underscores.  E.g. "password-tokens" not "passwordTokens" nor "password_tokens".
* Use plural nouns only for consistency (no singular nouns).
* Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* You should rarely go deeper than `resource/[id]`.
* You must not go deeper than `resource/[id]/resource`.
* Put the version number at the base of your URL, for example "http://example.com/*v1*/resource".
* URL vs HTTP header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * Otherwise put it in the header.  E.g.
      * Authentication info
      * Content type
      * Accepted content types
* Specify optional fields in a comma separated list.


### Good URL examples

* List of magazines:
    * `GET http://www.example.com/api/v1/magazines`
* Filtering & sorting is in the query string:
    * `GET http://www.example.com/api/v1/magazines?filters[year]=2011&sort=+year,-topic`
    * `GET http://www.example.com/api/v1/magazines?filters[topic]=economy&filters[year]=2011`
* Use `-` to exclusion and descending sort order:
    * `GET http://www.example.com/api/v1/magazines?sort=year,-topic`
    * `GET http://www.example.com/api/v1/magazines/1234?fields=subtitle,-date`
* Specify fields to include or exclude, in a comma separated list:
    * `GET http://www.example.com/api/v1/magazines/1234?fields=title,-subtitle,date`
* A single magazine in JSON format:
    * `GET http://www.example.com/api/v1/magazines/1234`
* All articles in (or belonging to) this magazine:
    * `GET http://www.example.com/api/v1/magazines/1234/articles`  
      (But prefer `GET http://www.example.com/api/v1/articles?filters[magazine]=1234`)
* Create a new article and add it to a particular magazine:
    * `POST http://example.com/api/v1/magazines/1234/articles`  
      (But prefer `POST http://www.example.com/api/v1/articles` including the magazine ID as a property of the entity.)
* No more than one identifier in the path.
* Any identifier must immediately follow the resource type name.


### Bad URL examples
* Singular noun:
    * `http://www.example.com/magazine`
    * `http://www.example.com/magazine/1234`
* Wrong order:
    * `http://www.example.com/publisher/magazine/1234`
* Verb in URL:
    * `http://www.example.com/magazine/1234/create`
* Filter not in the query string:
    * `http://www.example.com/magazines/2011/desc`

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.
The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context.  Where "Bo" is a dog with id `1234`:

| HTTP METHOD | POST             | GET       | PUT         | DELETE |
| ----------- | ---------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE           | READ      | UPDATE      | DELETE |
| /dogs       | Create a new dog | List dogs | Bulk update | Error  |
| /dogs/1234  | Error            | Show Bo   | Update Bo (if exists) | Delete Bo |

POST creates a new entity.  PUT updates existing entities, including any of the entity's relationships (but not the related entities themselves).

The web API may implement any of these.  It does not need to implement all of them.


## Responses

* No values in keys
* `metadata` should only contain direct properties of the response set, not properties of the members of the response set.

### Good examples

No values in keys:

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],


### Bad examples

Values in keys:

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],


## Error handling

Error responses (400-599 status codes) must not have `results`.  Error responses must include an internal error code and a user-friendly message.  It may contain more detail in a "payload" property, which must be an object.

For example:

    {
      "code": "csrf",
      "message": "The CSRF token has expired.
    }

Or:

    {
      "code": "csrf",
      "message": "The CSRF token has expired.
      "payload: {
         validationErrors: [
            {
               "name": "email",
               "message": "The email address is not valid."
            }
         ]
      }
    }

Use [standard/conventional HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes). Generally;

* 200-299: OK
* 400-499: Bad Request, client error 
* 500-599: Internal server error


## Versions

* Never release an API without a version number.
* Versions should be integers, not decimal numbers, prefixed with ‘v’. For example:
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* Maintain APIs at least one version back.


## Pagination

* If no limit is specified, return results with a default limit.
* To get records 51 through 75 do this:
    * `http://example.com/magazines?limit=25&offset=50`
    * `offset=50` means skip the first 50 records
    * `limit=25` means, return a maximum of 25 records

Information about record limits and total available count should also be included in the response. Example:

    {
        "metadata": {
            "resultSet": {
                "count": 227,
                "offset": 25,
                "limit": 25
            }
        },
        "results": []
    }

## Request & Response Examples

### API Resources

  - [GET /magazines](#get-magazines)
  - [GET /magazines/[id]](#get-magazinesid)
  - [POST /magazines/[id]/articles](#post-magazinesidarticles)

### GET /magazines

Example: http://example.com/api/v1/magazines

Response body:

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
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /magazines/[id]

Example: http://example.com/api/v1/magazines/[id]

Response body:

    {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }



### POST /magazines/[id]/articles

Example: Create – POST http://example.com/api/v1/magazines/[id]/articles

Request body:

        {
            "title": "Raising Revenue",
            "author_first_name": "Jane",
            "author_last_name": "Smith",
            "author_email": "jane.smith@example.com",
            "year": "2012",
            "month": "August",
            "day": "18",
            "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "
        }
