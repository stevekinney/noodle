node-scrape-query-language
==========================

nsql is a server which can be used to scrape pages from the client side browser.
It uses the jQuery query selector to extract information from web pages and
returns the data in JSON. The server also supports JSONP (?callback=foo) format
for making calls.

In your call to the server you just specify the source `url`, the `selector`,
and what you want to `extract`.

Features
--------------

- Cross domain DOM querying
- JSONP and JSON POST
- Multiple queries per request
- In memory caching

[Try it out!](http://dharmafly.github.com/nsql/#try-it-out);

Getting started
---------------

Setup

    $ git clone https://github.com/dharmafly/nsql.git
    $ cd nsql
    $ npm install

Start the server by running the binary

    $ bin/nsql-server
     Server running on port 8888

You may specify a port number as a command line argument

    $ bin/nsql-server 9090
     Server running on port 9090

Usage
-----

The server can accept scraping queries in a variety of ways:

### JSONP

As JSONP you can send a URI encoded blob of json for the `q=` querystring key.

`http://dharmafly.nsql-example.com?q=<JSONBLOB>&callback=?`

```JSON
{
  "url": "http://chrisnewtn.com",
  "selector": "ul.social li a",
  "extract": "href"
}
```

If you are unable to stringify JSON you can still use the querystring to make 
one query (but not multiple).

```JavaScript
jQuery.param(query);
// eg. url=http%3A%2F%2Fchrisnewtn.com&selector=ul.social+li+a&extract%5B%5D=text&extract%5B%5D=href
```

### POST

You can also POST your query as simple `application/json` to the nsql server. 
This is preferable if your request is too large or talking to nsql from another 
server.

### Extracting data

The `extract` property could be the HTML element's attribute.

Having `"html"` or `"innerHTML"` as the `extract` value will return the
containing HTML within that element.

Having `"text"` as the `extract` value will return only the text.

Return data looks like this:

```JSON
{
    "results": [
        {
            "href": "http://twitter.com/chrisnewtn"
        },
        {
            "href": "http://plus.google.com/u/0/111845796843095584341"
        }
    ],
    "created": "2012-08-01T16:22:14.705Z"
}
```

### Multiple extract rules

It is also possible to request multiple properties to extract in one query via
array.

Query:

```JSON
{
  "url": "http://chrisnewtn.com",
  "selector": "ul.social li a",
  "extract": ["href", "text"]
}
```

Response:

```JSON
{
    "results": [
        {
            "href": "http://twitter.com/chrisnewtn",
            "text": "Twitter"
        },
        {
            "href": "http://plus.google.com/u/0/111845796843095584341",
            "text": "Google+"
        }
    ],
    "created": "2012-08-01T16:23:41.913Z"
}
```

### Multiple queries per request

Multiple queries can be made per request to the server.

```JSON
[
  {
    "url": "http://chrisnewtn.com",
    "selector": "ul.social li a",
    "extract": ["text", "href"]
  },
  {
    "url": "http://premasagar.com",
    "selector": "#social_networks li a.url",
    "extract": "href"
  }
]
```

Response:

```JSON
[
    {
        "results": [
            {
                "href": "http://twitter.com/chrisnewtn",
                "text": "Twitter"
            },
            {
                "href": "http://plus.google.com/u/0/111845796843095584341",
                "text": "Google+"
            }
        ],
        "created": "2012-08-01T16:23:41.913Z"
    },
    {
        "results": [
            {
                "href": "http://dharmafly.com/blog"
            },
            {
                "href": "http://twitter.com/premasagar"
            }
        ],
        "created": "2012-08-01T16:22:13.339Z"
    }
]
```

### Errors

nsql fails silently and assumes error handling to be handled by the client side.
Consider the following JSON response to a partially incorrect query.

Query:

```JSON
{
  "url": "http://chrisnewtn.com",
  "selector": "ul.social li a",
  "extract": ["href", "nonexistent"]
}
```

Response:

The extract "nonexistent" property is left out because it was not found
on the element.

```JSON
{
    "results": [
        {
            "href": "http://twitter.com/chrisnewtn"
        },
        {
            "href": "http://plus.google.com/u/0/111845796843095584341"
        }
    ],
    "created": "2012-08-01T16:28:19.167Z"
}
```

If the selector is invalid or none of the extract rules match up then you will receive
an empty array.

Query:

```JSON
{
  "url": "http://chrisnewtn.com",
  "selector": "ul.social li a",
  "extract": ["nonexistent", "nonexistent2"]
}
```

Response:

```JSON
{
    "results": [],
    "created": "2012-08-01T16:26:39.734Z"
}
```