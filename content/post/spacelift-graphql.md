---
author: "Satish tripathi"
date: 2024-03-28
linktitle: "Understanding Spacelift GraphQL Query and Mutation Operations"
title: "Understanding Spacelift GraphQL Query and Mutation Operations"
---


*About GraphQL*:

GraphQL is a query language for your API and a server-side runtime for executing queries using a type system you define for your data. GraphQL isn't tied to any specific database or storage engine and
is instead backed by your existing code and data.

*Types of Operation in GraphQL:*
Query - queries are used to fetch the data.
Mutation- Mutations are used to modify server-side data.

*Spacelift Graphql Query/Mutations:*
If your spacelift account name is deployment then your spacelift graphql endpoint would be:
```text
    https://deployment.app.spacelift.io/graphql
```
*Spacelift graphql api supports two types of operation query and mutation by doing a POST request.*
Query - for fetching the data from spacelift server.
Mutation - Used for updating the existing values for spacelift resources.

Tool Used for testing the API requests: Insomnia

*Setting up the Token on Insomnia:*
Login into spacelift using CLI.
```bash
    spacectl profile login deployment
```
login into Spacelift using below authentication methods:
![Alt text](/image-7.png)
Get the Token from the CLI:
```bash
    spacelctl profile export-token | pbcopy
```

*Add the token to insomnia as a bearer token:*

![Alt text](/image-8.png)

query for listing all the API Keys with admin/non-admin permissions in Spacelift Account:

```graphql
    query {
        apiKeys{id,admin}
    }
```

The output will be listing all the API Keys id exist in the spacelift account:
![Alt text](/image-9.png)

> Spacelift graphql queries are straightforward and easy to use. In this blog, we will be focusing on spacelift graphql mutation. So as mentioned earlier mutation is used to modify/delete/create the existing resources on spacelift(ex: create/modify/delete context).

*Graphql Mutation structure:*

```graphql
    mutation{
        someEditOperation(dataField:"valueOfField"):returnType
    }
```

return type in case of spacelift could be context id, stack id, context name or type of return can be found on the mutation schema.

*Creating a Context using Mutation:*

In the below example, the return type is the id and name of the created context. contextCreate mutation takes data input as name, description, labels, and space.

![Alt text](/image-10.png)

Adding an Environment variable with name and secret to the newly created context.
Modify the Existing Context value:

![Alt text](/image-11.png)

In this case, we will be performing mutation operation contextConfigAdd to update the existing value of the above context to xyz.
Let's understand the schema that we have to pass in for performing this operation.

```graphql
 contextConfigAdd(context: ID!, config: ConfigInput!): ConfigElement!
```


contextConfigAdd needs two data fields contextid: ID!, config: ConfigInput!, and then the return type i.e. the ConfigElement.
```graphql
mutation updateContext ($context: ID!, $ConfigInput: ConfigInput!) {
 contextConfigAdd(context: $context, config: $ConfigInput) {id}
}
```


*Query variables:*
Query variables simplify GraphQL queries and mutations by letting you pass data separately.
```json
{
 "context": "deployment-redis-secret-key",
 "ConfigInput": {
 "id": "REDIS_SECRET_KEY",
 "value": "xyz",
 "writeOnly": true,
 "type": "ENVIRONMENT_VARIABLE"
 }
}
```

Clean Up Activity: Delete the context:

---

Cheers! You have now learned the art of performing SpaceLift GraphQL query and mutation operations. It's hard to explain everything in a single blog post, but by following the steps and guidelines outlined here, you have gained the necessary knowledge and skills to navigate the SpaceLift GraphQL ecosystem with confidence.