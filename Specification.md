# gqlx 0.1.0

The GraphQL eXtended (abbr. gqlx) language builds upon two existing languages:

- [GraphQL as specified by Facebook](http://facebook.github.io/graphql/June2018/)
- [ECMAScript 2016 as specified by ECMA](https://www.ecma-international.org/ecma-262/7.0/index.html)

This specification refers to terminology and conventions used in the previously mentioned documents.

## Scope

The goal of gqlx is to simplify writing GraphQL powered services and support building service oriented architectures exposing GraphQL powered gateways. Constructing self-contained systems is powerful and should not involve building cross-dependencies that are hard to maintain and difficult to keep stable. Instead, a system should be capable of handling its own resource definitions including a description of how to resolve these resources from it. gqlx helps to define all necessary steps in a single definition.

Usually, standard GraphQL definitions are broken down in four parts:

1. The schema declaration
2. The resolver definition
3. A set of models to be used
4. The respective data connectors

While the whole validation is performed against the schema, the resolver declares the binding from the schema to a model. A model then gets its data via the connectors, e.g., by talking to some RESTful API, database, or some caching system. Now imagine reducing these four parts into a single definition. While this is certainly not the solution for everything, there are many scenarios where such a simplification is not only wanted, but indeed needed.

gqlx also tries to extend GraphQL not only with a tighter, more concise, way of defining resources, but also with advanced meta descriptions.

## Language

The following paragraphs outline the syntax details as specific as possible using something close to BNF.

### Customized GraphQL Part

A gqlx snippet (or document) is denoted as `xdoc` and given by

```plain
xdoc ::= xDefinition+
xDefinition ::= xTypeSystemDefinition | TypeSystemExtension
```

Here the `xTypeSystemDefinition` is an extension to the `TypeSystemDefinition` defined in the GraphQL specification.

```plain
xTypeSystemDefinition ::= xSchemaDefinition | TypeDefinition | DirectiveDefinition
xSchemaDefinition ::= 'type' WhiteSpace+ OperationType (WhiteSpace+ Directives)? WhiteSpace* xFieldsDefinition
OperationType ::= 'query' | 'mutation' | 'subscription'
xFieldsDefinition ::= '{' WhiteSpace* xOperationDefinition* WhiteSpace* '}'
```

The `xOperationDefinition` contains the extended resolver syntax:

```plain
xOperationDefinition ::= Name WhiteSpace* VariableDefinitions? WhiteSpace* ':' WhiteSpace* Type WhiteSpace* xExpression
xExpression ::= '{' AssignmentExpression '}'
```

using the `AssignmentExpression` from the ECMAScript 2016 specification.

### Constrained ECMAScript Part

The content between the curly braces has to be a single valid ECMAScript expression. This expression is properly rewritten, generated, and will lead to a (potentially asynchronously) returned value.

Pretty much everything from the ECMAScript 2016 language is allowed except

- `'delete'` from the `UnaryExpression`
- `FunctionDeclaration`
- `FunctionExpression`
- `'this'` from the `PrimaryExpression`
- Keywords such as `async` or `await`

The API, on the other hand, is much more limited than described in the ECMAScript specification.

## API

gqlx uses four available sources for valid identifiers (e.g., referencing functions).

1. Whitelisted ECMAScript APIs / objects
2. Provided GraphQL operation argument names
3. Specified external names with optional asynchronous use
4. Defined internal helper functions

The following names are whitelisted from the ECMAScript specification:

- `null`
- `undefined`
- `Array`
- `Object`
- `Math`

Anything else, e.g., `Function`, `eval`, ... is not allowed. Accessing the `prototype` or `__proto__` members is also not supported even though the implementation could decide to actually allow access (this is for security reasons not recommended).

As far as the external names are concerned this part can be freely chosen by the user (as long as it does not violate any language rules, e.g., uses an ECMAScript 2016 invalid or reserved name).

By default the following API functions are assumed:

| Name   | Asynchronous | Description (impl. free)      |
|--------|--------------|-------------------------------|
| get    | true         | Performs HTTP GET request.    |
| post   | true         | Performs HTTP POST request.   |
| del    | true         | Performs HTTP DELETE request. |
| put    | true         | Performs HTTP PUT request.    |
| form   | true         | Creates an HTTP form body.    |
| listen | false        | Appends to an event listener. |

The implementation (receiving / handling the evaluated gqlx) can freely choose *how* to implement these functions. As the API can be freely chosen different names are suggested in case of different implementations than described in here.

Finally, we also have some inbuilt helper functions. Once a helper function is used some code will be generated within the current resolver. Essentially, the list of these inbuilt helpers can be fully decided by the implementation.

The following inbuilt functions are currently specified. It is *recommended* to at least implement these.

| Name   | Signature                                               | Description                                          |
|--------|---------------------------------------------------------|------------------------------------------------------|
| either | `(givenValue: T?, defaultValue: T): T`                  | Uses the given value or a default value.             |
| use    | `(value: T, cb: (val: T) => U): U`                      | Uses the first argument to feed the second argument. |
| cq     | `(url: string, query: { [name: string]: any }): string` | Concats a query to an URL.                           |

Implementations may choose to let developers extend the offered set of inbuilt functions.

## Validation

Validation of a provided gqlx needs to follow three steps:

1. Validation of the given syntax (see language section)
2. Validation of the used API (see API section)
3. Validation of the provided types (consistency, see GraphQL specification)

If any step fails an error should be emitted. In language where errors are being thrown as exceptions this mechanism should be used.

A given gqlx snippet has to be self-contained and must define all scalars and functions explicitly.
