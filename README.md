# Using MounteBank MockServer

## Context

* Mountebank provides configure virtual services, which are called imposters.
* Each imposter represents a socket that acts as the virtual service and accepts connections from the real service you
  are testing.
* Spinning up and shutting down imposters is a lightweight operation.

#### Key Concepts

* Predicates help route requests on the way in.
* Responses generate the responses on the way out.
* Behaviors postprocess the responses before shipping them over the wire

#### Setup Node.js(Windows)

- Goto [Node.js downloads](https://nodejs.org/en/download/) and download Node.js for Windows
    - LTS
    - Current
- Run the Node.js Installer(Windows)
    - Welcome to the Node.js setup wizard
        - Select **_Next_**
    - End-User License Agreement (EULA)
        - Check **_I accept the terms in the License Agreement_**
        - Select **_Next_**
    - Destination Folder
        - Select **_Next_**
    - Custom Setup
        - Select **_Next_**
    - Ready to install Node.js
        - Select **_Install_**
        - Note: This step requires Administrator privileges.
        - If prompted, authenticate as an Administrator.
    - Installing Node.js
        - Let the installer run to completion.
    - Completed the Node.js Setup Wizard
        - Click **_Finish_**

#### Setup Node.js(Mac)

- Install Homebrew

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

- Install Node.js

```sh
brew install node
```

## Verify That Node.js was Properly Installed and Install Mountebank

- Run command

```sh
node -v
```

- Update npm

```sh
npm install npm -g
```

- Install mountebank

```sh
npm install mountebank -g
```

#### How mountebank views an HTTP request

```txt
POST /products?page=2&itemsPerPage=2 HTTP/1.1
Host: api.petstore.com
Content-Type: application/json

{
  "key": "asdul7890"
}
```

```json
{
  "method": "POST",
  "path": "/products",
  "query": {
    "page": 2,
    "itemsPrePage": 2
  },
  "headers": {
    "Host": "api.petstore.com",
    "Content-Type": "application/json"
  },
  "body": "{\n  \"key\": \"asdul7890\"\n}"
}
```

#### How mountebank views an HTTP response

```txt
HTTP/1.1 200 OK
Date: Sun, 05 Apr 2020 10:10:10 GMT
Content-Type: application/json

{
  "key": "asdul7890"
}
```

```json
{
  "statusCode": 200,
  "headers": {
    "Date": "Sun, 05 Apr 2020 10:10:10 GMT",
    "Content-Type": "application/json"
  },
  "body": "{\n  \"key\": \"asdul7890\"\n}"
}
```

#### The HTTP response structure in JSON

```json
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "text/plain"
  },
  "body": "Hello, World!"
}
```

---

#### Hello, World Mountebank

##### Run: [Command line]

```sh
mb
```

> open web browser then go to [http://localhost:2525](http://localhost:2525)

##### List imposters: [Command line, Web browser]

> curl

```sh
curl http://localhost:2525/imposters
```

> goto [http://localhost:2525/imposters](http://localhost:2525/imposters)

##### Create First Imposters: [Command line]

> curl

```sh
curl -X POST http://localhost:2525/imposters --data '
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "statusCode": 200,
            "headers": { "Content-Type": "text/plain" },
            "body": "Hello, World!"
          }
        }
      ]
    }
  ]
}'
```

##### Delete: [Command line]

> curl

```sh
curl -X DELETE http://localhost:2525/imposters/3000
```

> response

```sh
info: [mb:2525] DELETE /imposters/3000
info: [http:3000] Ciao for now
```

> list imposters

```sh
curl http://localhost:2525/imposters
```

##### Delete all impostes

> curl

```sh
curl -X DELETE http://localhost:2525/imposters
```

##### Create First Imposters from Config File

> create file call hello-world.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "statusCode": 200,
            "headers": {
              "Content-Type": "text/plain"
            },
            "body": "Hello, World!"
          }
        }
      ]
    }
  ]
}
```

```sh
mb start --configfile hello-world.json
```

---

#### The Basic of Mountebank HTTP Responses

The **_is_** response type, which is the fundamental building block for a stub.

##### The HTTP response structure in JSON

- hello-world.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "statusCode": 200,
            "headers": {
              "Content-Type": "text/plain"
            },
            "body": "Hello, World!"
          }
        }
      ]
    }
  ]
}
```

##### Default Response

- default-response.json

```json
{
  "protocol": "http",
  "port": 3001,
  "name": "Default Response",
  "defaultResponse": {
    "statusCode": 400,
    "headers": {
      "Connection": "Keep-Alive",
      "Content-Length": 0
    }
  },
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "body": "BOOM!!!"
          }
        }
      ]
    }
  ]
}
```

##### Cycling through response

- cycling-through-response.json

```json
{
  "protocol": "http",
  "port": 3002,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "body": "1"
          }
        },
        {
          "is": {
            "body": "2"
          }
        },
        {
          "is": {
            "body": "3"
          }
        }
      ]
    }
  ]
}
```

---

#### Saving multiple Imposters in the config file

```json
{
  "imposters": [
    {
      "protocol": "http",
      "port": 3000,
      "stubs": [
        {
          "responses": [
            {
              "is": {
                "statusCode": 200,
                "headers": {
                  "Content-Type": "text/plain"
                },
                "body": "Hello, World!"
              }
            }
          ]
        }
      ]
    },
    {
      "protocol": "http",
      "port": 3001,
      "name": "Default Response",
      "defaultResponse": {
        "statusCode": 400,
        "headers": {
          "Connection": "Keep-Alive",
          "Content-Length": 0
        }
      },
      "stubs": [
        {
          "responses": [
            {
              "is": {
                "body": "BOOM!!!"
              }
            }
          ]
        }
      ]
    },
    {
      "protocol": "http",
      "port": 3002,
      "stubs": [
        {
          "responses": [
            {
              "is": {
                "body": "1"
              }
            },
            {
              "is": {
                "body": "2"
              }
            },
            {
              "is": {
                "body": "3"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

```sh
imposters.ejs
hello-world.json
default-response.json
cycling-through-response.json
```

- imposters.ejs

```json
{
  "imposters": [
    <
    %
    -
    include
    hello-world.json
    %
    >,
    <
    %
    -
    include
    default-response.json
    %
    >,
    <
    %
    -
    include
    cycling-through-response.json
    %
    >
  ]
}
```

#### EJS: Embedded JavaScript

> Mountebank uses a templating language called EJS ([https://ejs.co/](https://ejs.co/))

---

#### Imposter Structure

```json
{
  "port": <port>,
  "protocol": "http",
  "stubs": [
    {
      "predicates": [],
      "responses": [
        {
          "is": {
            "statusCode": <statusCode>,
            "headers": {},
            "body": {}
          }
        }
      ]
    }
  ]
}
```

#### Predicates

![img.png](../images/mountebank_predicate.png)

> Can be used with the following

* Path
* HTTP Method
* HTTP Header
* Query
* Body
    * Plan Text
    * JSON
    * XML

#### Type of Predicates

##### The matches predicates

> matches-predicate.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "predicates": [
        {
          "equals": {
            "method": "POST"
          }
        },
        {
          "startsWith": {
            "body": "smart"
          }
        },
        {
          "endsWith": {
            "body": "robot"
          }
        }
      ],
      "responses": [
        {
          "is": {
            "body": "Hello, smart robot"
          }
        }
      ]
    }
  ]
}
```

> Mountebank matches

```json
{
  "predicates": [
    {
      "startsWith": {
        "body": "smart"
      }
    },
    {
      "contains": {
        "body": "alpha3"
      }
    },
    {
      "endsWith": {
        "body": "robot"
      }
    }
  ]
}
```

> match with regular expressions

```json
{
  "predicates": [
    {
      "matches": {
        "body": "^text to match"
      }
    },
    {
      "matches": {
        "body": ".*text to match.*"
      }
    },
    {
      "matches": {
        "body": "text to match$"
      }
    }
  ]
}
```

##### Matching any identifier on the path

> path-identifier-predicate.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "predicates": [
        {
          "matches": {
            "path": "/items/\\d+"
          }
        }
      ],
      "responses": [
        {
          "is": {
            "body": {
              "name": "43 Piece Dinner Set",
              "price": 12.95,
              "quantity": 10
            }
          }
        }
      ]
    }
  ]
}
```

The metacharacters used here, \d+, represent one or more digits, so the pattern will match /items/123 and /items/2 but
not items/robot.

### Matching object request fields

> request-field-predicate.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "predicates": [
        {
          "equals": {
            "query": {
              "q": "robot"
            }
          }
        }
      ],
      "responses": [
        {
          "is": {
            "body": {
              "items": [
                {
                  "name": "TA Robot",
                  "price": 109.99,
                  "quantity": 50
                },
                {
                  "name": "AlphaBot2",
                  "price": 71.0,
                  "quantity": 73
                }
              ]
            }
          }
        }
      ]
    }
  ]
}
```

### The deep equals predicate

> deep-equals-predicate.json

```json
{
  "deepEquals": {
    "query": {
      "q": "robot",
      "page": 1
    }
  }
}
```

With this predicate, a query string of ?q=robot&page=1 would match, but a query string of ?q=robot&page=1&sort=desc
wouldn’t.

### Matching multivalued fields

> GET /items?q=smart&q=robot

> multivalued-field-predicate-01.json

> requires exact match

```json
{
  "predicates": [
    {
      "deepEquals": {
        "query": {
          "q": [
            "smart",
            "robot"
          ]
        }
      }
    }
  ]
}
```

> GET /items?q=smart&q=robot

> multivalued-field-predicate-02.json

> requires these elements to be present

```json
{
  "predicates": [
    {
      "equals": {
        "query": {
          "q": [
            "smart",
            "robot"
          ]
        }
      }
    }
  ]
}
```

### The exists predicate

There’s one more primitive predicate to look at. The exists predicate tests for either the existence or nonexistence of
a request field.  
For example, you may decide that for testing purposes, you want to verify that the service handles an HTTP challenge
correctly (represented by a 401 sta- tus code) when the Authorization request header is missing, without worrying about
whether the credentials stored in the Authorization header are correct, as shown here:

> exists-predicate.json

```json
{
  "predicates": [
    {
      "exists": {
        "headers": {
          "Authorization": false
        }
      }
    }
  ],
  "responses": [
    {
      "is": {
        "statusCode": 401
      }
    }
  ]
}
```

### Conjunction junction

> conjunction-without-and-predicate.json

```json
{
  "predicates": [
    {
      "startsWith": {
        "body": "smart"
      }
    },
    {
      "endsWith": {
        "body": "robot"
      }
    }
  ]
}
```

You can reduce the array to a single element by using the and predicate.

> conjunction-with-and-predicate.json

```json
{
  "predicates": [
    {
      "and": [
        {
          "startsWith": {
            "body": "smart"
          }
        },
        {
          "endsWith": {
            "body": "robot"
          }
        }
      ]
    }
  ]
}
```

### A complete list of predicate types

| Operator   | Description                                                                                |
| ---------- | ------------------------------------------------------------------------------------------ |
| equals     | Requires the request field to equal the predicate value                                    |
| deepEquals | Performs nested set equality on object request fields                                      |
| contains   | Requires the request field to contain the predicate value                                  |
| startsWith | Requires the request field to start with the predicate value                               |
| endsWith   | Requires the request field to end with the predicate value                                 |
| Matches    | Requires the request field to match the regular expression provided as the predicate value |
| exists     | Requires the request field to exist as a nonempty value (if true) or not (if false)        |
| not        | Inverts the subpredicate                                                                   |
| or         | Requires any of the subpredicates to be satisfied                                          |
| and        | Requires all of the subpredicates to be satisfied                                          |

---

## Parameterizing predicates

Predicates are case-insensitive by default.

### Making case-sensitive predicates

> case-sensitive-predicate.json

```json
{
  "predicates": [
    {
      "equals": {
        "query": {
          "q": "robot"
        }
      }
    }
  ]
}
```

vs

```json
{
  "predicates": [
    {
      "equals": {
        "query": {
          "q": "robot"
        }
      },
      "caseSensitive": true
    }
  ]
}
```

---

## Using predicates on JSON values

### Using direct JSON predicates

> POST /items

```json
{
  "name": "riderX",
  "price": 100,
  "quantity": 50
}
```

> direct-json-predicate.json

```json
{
  "predicates": [
    {
      "equals": {
        "body": {
          "name": "riderX"
        }
      }
    }
  ],
  "responses": [
    {
      "is": {
        "statusCode": 201,
        "headers": {
          "Location": "/items/123"
        }
      }
    }
  ]
}
```

### Selecting a JSON value with JSONPath

> POST /shipping

```json
{
  "items": [
    {
      "name": "TA Robot",
      "price": 109.99,
      "quantity": 50,
      "location": "Bangkok"
    },
    {
      "name": "AlphaBot2",
      "price": 71.0,
      "quantity": 73,
      "location": "Covid-19 Zone"
    }
  ]
}
```

> select-a-json-value-with-jsonpath-predicate.json

```json
{
  "predicates": [
    {
      "equals": {
        "method": "POST"
      }
    },
    {
      "equals": {
        "path": "/shipping"
      }
    },
    {
      "jsonpath": {
        "selector": "$.items[(@.length-1)].location"
      },
      "equals": {
        "body": "Covid-19 Zone"
      }
    }
  ],
  "responses": [
    {
      "is": {
        "statusCode": 400
      }
    }
  ]
}
```

> ref: [JSONPath syntax](https://goessner.net/articles/JsonPath/)

## Selecting XML values

> POST /shipping

```xml

<items>
    <item name="TA Robot">
        <price>109.99</price>
        <location>Bangkok</location>
    </item>
    <item name="AlphaBot2">
        <price>71.50</price>
        <location>Covid-19 Zone</location>
    </item>
</items>
```

> select-a-xml-value-with-xpath-predicate.json

```json
{
  "predicates": [
    {
      "equals": {
        "method": "POST"
      }
    },
    {
      "equals": {
        "path": "/shipping"
      }
    },
    {
      "xpath": {
        "selector": "//item[@name='AlphaBot2']/location"
      },
      "equals": {
        "body": "Covid-19 Zone"
      }
    }
  ],
  "responses": [
    {
      "is": {
        "statusCode": 400
      }
    }
  ]
}
```

#### Behaviors

* Predicates help route requests on the way in.
* Responses generate the responses on the way out.
* Behaviors post-process the responses before shipping them over the wire

![img.png](../images/mountebank_behavior.png)

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "predicates": [],
      "responses": [
        {
          "is": {},
          "_behaviors": {
            "decorate": {},
            "wait": {},
            "copy": []
          }
        }
      ]
    }
  ]
}
```

## Decorating a response

Decoration allows you to post-process the response.

### Using the decorate function

- An **_inject_** response to a **_decorate_** behavior

```json
{
  "responses": [
    {
      "inject": "function (request, state, logger) {}"
    }
  ]
}
```

```json
{
  "responses": [
    {
      "is": {
        "body": {}
      },
      "_behaviors": {
        "decorate": "function (request, response, logger) {}"
      }
    }
  ]
}
```

> file decorate.json

```sh
mb start --allowInjection --configfile decorate.json
```

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "statusCode": 201,
            "headers": { "Content-Type": "application/json" },
            "body": {}
          },
          "_behaviors": {
            "decorate": "function (request, response, logger) { const item = JSON.parse(request.body); response.body.message = item.name"
          }
        }
      ]
    }
  ]
}
```


#### Adding latency to a Response

##### Using a wait behavior to add latency

> file sleep.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "statusCode": 201,
            "headers": { "Content-Type": "application/json" },
            "body": { "name": "sleep" }
          },
          "_behaviors": {
            "wait": 3000
          }
        }
      ]
    }
  ]
}
```

#### Repeating a Response Multiple Times

##### Using a repeat behavior to return an error after a small number of successes

> file repeating_a_response.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "predicates": [
        {
          "matches": { "path": "/items/1" }
        }
      ],
      "responses": [
        {
          "is": {
            "body": {
              "name": "43 Piece Dinner Set",
              "price": 12.95
            }
          },
          "_behaviors": {
            "repeat": 3
          }
        },
        {
          "is": {
            "body": {
              "name": "RiderX",
              "price": 2.95
            }
          }
        },
        {
          "is": {
            "body": {
              "name": "Alpha Bot",
              "price": 33.95
            }
          },
          "_behaviors": {
            "repeat": 5
          }
        }
      ]
    }
  ]
}
```

## Replacing Content in The Response

You can always add dynamic data to a response through an inject response, or through the decorate and shellTransform behaviors. But two additional behaviors support inserting certain types of dynamic data into the response without the overhead of programmatic control.


### Copying Request Data to the Response

- The copy behavior accepts an array, which means you can make multiple replacements in the response.
- Each replacement should use a different token, and each one can select from a different part of the request.
- You never specify where the token is in the response. That’s by design. You could have put the token in the headers or even the status- Code, and mountebank would replace it.


### Using a copy behavior to insert the ID from the URL into the response body

> request

```json
GET /items/1 HTTP/1.1
Host: localhost:3000

```

> response

```json
{
  "id": 1,
  "name": "43 Piece Dinner Set",
  "price": 12.95
}
```

> file replacing_content_in_the_response.json
```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "is": {
        "body": {
          "id": "$ID",
          "name": "43 Piece Dinner Set",
          "price": 12.95
        }
      },
      "_behaviors": {
        "copy": [
          {
            "from": "path",
            "into": "$ID",
            "using": {
              "method": "regex",
              "selector": "\\d+$"
            }
          }
        ]
      }
    }
  ]
}
```

### The regex primitives

- \d A digit, 0–9 (you have to double-escape the backslash in JSON)
- \w A word character
- \+ One or more times
- \$ The end of the string


### Using a Group Match

```txt
items/(\\w+)

items/123

['items/123', '123']
```

> file using_a_grouped_match.json

```json
{
  "protocol": "http",
  "port": 3000,
  "stubs": [
    {
      "responses": [
        {
          "is": {
            "body": {
              "id": "$ID[1]",
              "name": "43 Piece Dinner Set",
              "price": 12.95
            }
          },
          "_behaviors": {
            "copy": [
              {
                "from": "path",
                "into": "$ID",
                "using": {
                  "method": "regex",
                  "selector": "items/(\\w+)"
                }
              }
            ]
          }
        }
      ]
    }
  ]
}
```


#### Basic command and option

```sh
mb start --configfile <configfile>.json --pidfile &
```

```sh
mb stop
```

```sh
mb restart  --configfile hello-world.json --pidfile &
```

**_References_** :

- [mbtest](http://www.mbtest.org/docs/gettingStarted)
- [testing-microservices-with-mountebank](https://www.manning.com/books/testing-microservices-with-mountebank)

