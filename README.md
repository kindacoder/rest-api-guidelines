# Rest API Guidelines

The aim of this document is to highlight good practices for RESTful API design. We do understand that there are no right answers here, but some approaches have fewer cons than others.

## HTTP methods

Best explained by examples:

```
GET /posts - Retrieves a list of posts
GET /posts/12 - Retrieves a specific post
POST /posts - Creates a new post
PUT /posts/12 - Updates post #12
PATCH /posts/12 - Partially updates post #12
DELETE /posts/12 - Deletes post #12
```

Use nouns but no verbs.
Keep it simple and use only plural nouns for all resources.

Good:

```
/photos - Returns a list of photos
/photos/71 - Returns a specific photo
```

Bad:

```
/getAllPhotos
/createNewPhoto
/deleteAllBlurPhotos
/get-all-photos
/get-photo-details

```

If a resource is related to another resource, use subresources whenever possible.

```
GET /posts/711/comments/ (Returns a list of comments for post 711)
GET /posts/711/comments/4 (Returns comment #4 for post 711)
```

## Rate limiting

To prevent abuse, it is now standard practice to add some sort of rate limiting to an API. HTTP status code `429 Too Many Requests` will be returned if you exceed the prescribed rate limit.

## JSON only responses

It's time to leave XML behind in APIs. It's verbose, it's hard to parse, it's hard to read, its data model isn't compatible with how most programming languages model data and its extendibility advantages are irrelevant when your output representation's primary needs are serialization from an internal representation.

## SSL always

One thing to watch out for is non-SSL access to API URLs. We do not redirect these to their SSL counterparts and throw a hard error instead! The last thing you want is for poorly configured clients to send requests to an unencrypted endpoint, just to be silently redirected to the actual encrypted endpoint.

## Enable CORS

We should enable Cross-Origin Resource Sharing (CORS) by default.

## Keep JSON minified in all responses

Extra whitespace adds needless response size to requests, and many clients for human consumption will automatically "prettify" JSON output. It is best to keep JSON responses minified.

## JSON Data vs Form Data

We should use JSON over form data because complicated nested forms are represented more easily in JSON than form data.

## Filtering

```
GET /posts?status=published&sort=desc
GET /posts?fields=title,desc,id
```

## Versioning

Our preferred URL structure would be `/post/api/v1`.

Avoid v1.1 etc.

## Response Codes

| Code    |          Alias          | Description                                                                                                                                                                    |
| ------- | :---------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **200** |          _OK_           | Everything is working as expected.                                                                                                                                             |
| **201** |        _Created_        | Response to a POST that results in a creation. Should be combined with a Location header pointing to the location of the new resource                                          |
| **204** |      _No Content_       | Response to a successful request that won't be returning a body (like a DELETE request)                                                                                        |
| **304** |     _Not Modified_      | The client can use cached data                                                                                                                                                 |
| **400** |      _Bad Request_      | The request was invalid or cannot be served. The exact error will be explained in the error payload. This occurs often due to missing a required parameter.                    |
| **401** |     _Unauthorized_      | The request requires an user authentication.                                                                                                                                   |
| **403** |       _Forbidden_       | The server understood the request, but is refusing it or the access is not allowed / When authentication succeeded but authenticated user doesn't have access to the resource. |
| **404** |       _Not Found_       | There is no resource behind the URI.                                                                                                                                           |
| **405** |   Method not Allowed    | When an HTTP method is being requested that isn't allowed for the authenticated user.                                                                                          |
| **408** |   _Request Timed Out_   | The server timed out while waiting for the request.                                                                                                                            |
| **410** |         _Gone_          | Indicates that the resource at this end point is no longer available. Useful as a blanket response for old API versions.                                                       |
| **422** | _Unprocessable Entity_  | Used for validation errors. Should be used if the server cannot process the enitity, e.g. if an image cannot be formatted or mandatory fields are missing in the payload.      |
| **429** |   _Too Many Requests_   | Too Many Requests - When a request is rejected due to rate limiting.                                                                                                           |
| **500** | _Internal Server Error_ | API developers should avoid this error. If an error occurs in the global catch blog, the stracktrace should be logged and not returned as response.                            |

## Error Responses

Some folks will try to use HTTP status codes exclusively and skip using error codes because they do not like the idea of making their own error codes or having to document them, but this is not a scalable approach.

There will be some situations where the same endpoint could easily return the same status code for more than one different condition. The status codes are there to merely hint at the category of error relying on the actual error code and error message provided in the HTTP response to include more information in case the client is interested.

```
{
  "code" : 1234,
  "message" : "Something bad happened :(",
  "description" : "More details about the error here"
}
```

```
{
  "code" : 1024,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot have fancy characters"
    },
    {
       "code" : 5622,
       "field" : "password",
       "message" : "Password cannot be blank"
    }
  ]
}
```

- No internal names will be exposed in the API (for example, “node” and “taxonomy term”)

## Pagination

We will allow the client to request the number of items it would like returned per HTTP request.

```
GET /posts?limit=25&page=1
```

If no limit is specified, return results with a default limit.

```
{
  "metadata": {
    "pagination": {
      "total": 227,
      "page": 1,
      "limit": 25
    }
  },
  "data": [...]
}
```

## Successful Responses

```
{
  "data": [
    {
      "name": "Hulk Hogan",
      "id": "100002"
    },
    {
      "name": "Mick Foley",
      "id": "100003"
    }
  ]
}
```

For nesting foreign key relations:

Correct way:

```
{
  "id": 1,
  "title": "title for post one",
  "desc": "Description for post one here..",
  "owner": {
    "id": 3,
    "name":"John doe"
  },
  // ...
}
```

Wrong way:

```
{
  "id": 1,
  "title": "title for post one",
  "desc": "Description for post one here..",
  "owner_id": 3,
  "owner_name":"John doe"
}
```

- A PUT, POST or PATCH call may make modifications to fields of the underlying resource that weren't part of the provided parameters (for example: created_at or updated_at timestamps).
- To prevent an API consumer from having to hit the API again for an updated representation, have the API return the updated (or created) representation as part of the response.

## Naming Conventions

Since we are using JSON as your primary representation format, the "right" thing to do is to follow JavaScript naming conventions - and that means camelCase for field names.

## Including related resources

It can be hard to pick between subresource URLs or embedded data. Embedded data can be rather difficult to pull off

```
GET /posts/12?includes=author,comments
```

```
{
  "id": 1,
  "title": "title for post one",
  "desc": "Description for post one here..",
  "author" : {
    "id":3,
    "name" : "John doe"
  },
  comments: [
    {
      "id": 1,
      "comment": "Comment 1"
    },
    {
      "id": 2,
      "comment": "Comment 2"
    }
  ]
}
```

## No auto increment ID’s

- We do not want people to know how many resources we have.
- We do not want developers to automate a scraping script by going to /posts/1, /posts/2 etc. and scraping everything away.
- Use GUIDs or UUIDs, check this nice article [UUIDs are Popular, but Bad for Performance](https://www.percona.com/blog/2019/11/22/uuids-are-popular-but-bad-for-performance-lets-discuss/) before you decide what to use.

## Enable GZip

Gzip will be enabled by default in production on all API responses.

## Transforming API output

- You want true or false as an actual JSON boolean, not a numeric string or a char(1).
- Never directly expose db fields because if you add new fields later, the api structure will also change.

## Other guidelines

### Git basic Guidelines

1. Limit the subject line to 50 characters
2. Capitalize the subject line
3. Do not end the subject line with a period
4. Use the imperative mood in the subject line. Explain what and why vs. how (The "how" is visible in the code itself)
5. Do not use past-tense like FIXED etc.
6. Do not mix multiple activities in the same commit.

### Git Commit Best Practices

#### A commit message should answer three primary questions:

- Why is this change necessary?
- How does this commit address the issue?
- What effects does this change have?

#### Standard prefixes to be followed for all commits:

- FIX (For bug fixes with Trello card URL if possible)
- ADD (New features)
- CHANGE (Updating some functionality which is not a bug)
- REFACTOR (Changing an existing feature without any change in overall functionality)
- WIP (Temporary commit only for backup sake)
- REMOVE (Remove a feature)

## Documentation

Some good tools and articles:

- [Postman](https://www.getpostman.com/)
- [API docs](http://apidocjs.com/)
- [Swagger](http://swagger.io/)
- [RAML](http://raml.org/index.html)
- [Slate - API docs generator](https://github.com/lord/slate)
- [Dev Docs](https://github.com/Thibaut/devdocs)
- [ProgrammableWeb - Web API Documentation Best Practices](http://www.programmableweb.com/news/web-api-documentation-best-practices/2010/08/12)
- [The Best API Documentation](http://bradfults.com/the-best-api-documentation/)

Some good examples of API documentation:

- [Twitter Docs](https://dev.twitter.com/overview/documentation)
- [Stripe Docs](https://stripe.com/docs/api#charges)
- [Twilio Docs](https://www.twilio.com/docs/api/rest)

### Credits / References

- http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
- https://github.com/squareboat/api-guidelines

---
