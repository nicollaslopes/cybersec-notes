# API 

An API (**Application Programming Interface**) basically serves data/information to a client, interacting with some application.

### Types of API:

* Rest
* GraphQL
* SOAP

## REST

A REST API is an architectural style for creating web services. It is based on the use of HTTP to communicate between applications and has some main characteristics:

- **Stateless**: Each request sent to the API must contain all the information necessary to be processed, without relying on previous information (no state storage).

- **Resource usage**: REST APIs use the concepts of resources, which are objects or data accessed through URIs (Uniform Resource Identifiers).

- **HTTP Methods**: To interact with resources, HTTP methods are used, such as:
    - **GET**: Retrieve data.
    - **POST**: Create or send data.
    - **PUT**: Update data.
    - **DELETE**: Delete data.

Let's suppose we want to request information from a specific user, for this we can use the GET method.

```
https://test.com/api/v1/users/1
```

If we want to create a new user, we can use the POST method,

```
https://test.com/api/v1/users
```

Sending the json with the information in the body:
 
```json
{ 
	"nome": "Maria Oliveira", 
	"email": "maria@example.com", 
	"idade": 25 
}
```

## GraphQL

GraphQL is a query language for APIs and a runtime for executing those queries with your data. In a typical REST API, we often end up get more data than necessary, in GraphQL we can specify precisely which fields of data we want to retrieve in a single request.
#### Basic GraphQL Terminology:

- **Query**: A request to fetch data.
- **Mutation**: A request to modify data (like creating, updating, or deleting).
- **Subscription**: A request for real-time updates to data.

Example: Let’s say we have a blogging platform with posts and authors. Here’s a simple GraphQL query to get the titles of posts along with the names of their authors:

```graphql
{
  posts {
    title
    author {
      name
    }
  }
}
```

In this query, we’re requesting the `title` of each `post` and the `name` of the `author` related to that post.

Example response:

```graphql
{
  "data": {
    "posts": [
      {
        "title": "GraphQL Basics",
        "author": {
          "name": "John Doe"
        }
      },
      {
        "title": "Advanced GraphQL Concepts",
        "author": {
          "name": "Jane Smith"
        }
      }
    ]
  }
}
```

Example of a Mutation:

A **mutation** in GraphQL is used to modify data. Here's an example of creating a new post:

```graphql
mutation {
  createPost(title: "GraphQL Advanced Features", content: "Content about advanced GraphQL features") {
    id
    title
    content
  }
}
```


Example response:

```graphql
{
  "data": {
    "createPost": {
      "id": "123",
      "title": "GraphQL Advanced Features",
      "content": "Content about advanced GraphQL features"
    }
  }
}
```

## SOAP

**SOAP (Simple Object Access Protocol)** is a protocol used for exchanging structured information in web services, primarily over HTTP or SMTP. It is an older, more formal standard for web services communication compared to newer technologies like REST or GraphQL. SOAP uses XML to define the message format and relies on other protocols (like HTTP, SMTP, or JMS) for message transmission.

The SOAP protocol follows a strict message format and includes the following key components:

1. **Envelope**: The outermost element that defines the start and end of the message.
2. **Header**: Optional metadata or additional information about the message.
3. **Body**: Contains the actual message or the request/response data.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" 
                  xmlns:web="http://www.example.com/banking">
   <soapenv:Header/>
   <soapenv:Body>
      <web:GetAccountBalance>
         <web:AccountNumber>123456789</web:AccountNumber>
      </web:GetAccountBalance>
   </soapenv:Body>
</soapenv:Envelope>
```