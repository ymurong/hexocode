---
title: GraphQL Basics
date:  2021-11-04 12:00:00
tags:
- graphql
---

## What is GraphQL?
A spec that describes a declarative query language that your
clients can use to ask an API for the exact data they want. 

This is achieved by creating a strongly typed Schema for your API, ultimate flexibility in how your API can resolve data, and client queries validated against your Schema.

Spec: http://spec.graphql.org/

## Where does GraphQL fit in
* A GraphQL server with a connected DB (most greenfields)
* A GraphQL server as a layer in front of many 3rd party
services and connects them all with one GraphQL API
* A hybrid approach where a GraphQL server has a
connected DB and also communicates with 3rd party
services

## Service Side

* Type Definitions
* Resolvers
* Query Definitions
* Mutation Definitions
* Composition
* Schema

## Schemas

* Using Schema Definition Language (SDL)
* Programmatically Creating a Schema using language
constructs

### Creating Queries

A Type on a Schema that defines operations clients can
perform to access data that resembles the shape of the other
Types in the Schema

* Create Query Type in the Schema using SDL
* Add fields to the Query Type
* Create Resolvers that for the field

### Hello World GraphQL API

```
const { gql } = require('graphql-tag')
const { ApolloServer } = require('apollo-server')

/**
 * Type Definitions for our Schema using the SDL.
 */
const typeDefs = gql`
   type User {
	email: String!
   	avatar: String
   	friends: [User!]!
   }
   type Query {
	me: User!
   }
`;

const resolvers = {
   Query: {
	me() {
    	    return {
		email: 'yoda@masters.com',
		avatar: 'http://yoda.png',
		friends: []
	    }
	}
   }
}

const server = new ApolloServer()

server.listen(4000).then(() => {
  console.log(`running at 4000`);
})

```

## What are Resolvers?

Functions that are responsible for returning values for fields that exist on Types in a Schema. Resolvers execution is dependent on the incoming client Query.

* Resolver names must match the exact field name on your
Schema’s Types
* Resolvers must return the value type declared for the
matching field
* Resolvers can be async
* Can retrieve data from any source

## What are Arguments?

* Allows clients to pass variables along with Queries that
can be used in your Resolvers to get data
* Must be defined in your Schema
* Can be added to any field
* Either have to be Scalars or Input Types

### Arguments in Resolvers

* Arguments will be passed to field Resolvers as the second
argument
* The argument object will strictly follow the argument
names and field types
* Do whatever you want with them

## What are Input Type?
* Just like Types, but used for Arguments
* All field value types must be other Input Types or Scalars

## What are Mutation Type

A Type on a Schema that defines operations clients can
perform to mutate data (create, update, delete).

* Define Mutation Type on Schema using SDL
* Add fields for Mutation type
* Add arguments for Mutation fields
* Create Resolvers for Mutation fields


## Authentication & Authorization in GraphQL

### Authorization 

*  Should not be coupled to a resolver
*  Can provide field level custom rules
*  Can authorize some of your schema and not all

### Authentication

*  provides the user to resolvers
*  Should not be coupled to a resolver
*  can protect some of your schema and not all of it
*  can provide field level protection 


## Subscriptions

* Subscriptions must be added to your Schema like Queries and Mutations
* Setup PubSub protocol server side
* Create Subscription event resolvers
* Add any needed authentication and context 
* Client side setup


## Directives 

Allows you to add logic and metadata to your Schemas, Queries, or Mutations. Directives can act like middleware for your Schemas, or post processing hooks for your Queries and Mutations.

#### schema definition
```
directive @formatDate(format: String = "dd MMM yyy") on FIELD_DEFINITION
```


#### directive definition
```
class FormatDateDirective extends SchemaDirectiveVisitor {
    visitFieldDefinition(field){
        const resolver = field.resolve || defaultFieldResolver
        const {format: schemaFormat} = this.args // this refers to the schemaVisitor, this.args refers to the args of the schema definition

   	// accept custom argument on directive
        field.args.push({
            name: 'format',
            type: GraphQLString
        })

        field.resolve = async (root, {format, ...rest}, ctx, info) => {
            const result = await resolver.call(this, root, rest, ctx, info)
            return formatDate(result, format || schemaFormat)
        }
        field.type = GraphQLString
    }
}
```

#### declare schema directive on apollo server
```
new ApolloServer({
  typeDefs,
  resolvers,
  schemaDirectives: {
    formatDate: FormatDateDirective
  },
...
})
```