# WampAPI Specification

## Version 0.1.0

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT
RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described
in [BCP 14][BCP-14] [RFC2119][RFC2119] [RFC8174][RFC8174] when, and only when, they appear in all capitals,
as shown here.

This document is licensed under [The MIT License](../LICENSE).

## Introduction

The WampAPI Specification (WAS) defines a standard, language-agnostic interface to WAMP APIs which allows both humans
and computers to discover and understand the capabilities of the service without access to source code, documentation,
or through network traffic inspection. When properly defined, a consumer can understand and interact with the remote
service with a minimal amount of implementation logic.

A WampAPI definition can then be used by documentation generation tools to display the API, code generation tools to
generate clients in various programming languages, testing tools, and many other use cases.

## Table of Contents



## WampAPI Document

A self-contained or composite resource which defines or describes a Wamp-based API or elements of a Wamp-based API.
The WampAPI document MUST contain at least one [WAMP URI RPC Action](#uri-rpc-action-object) or
[WAMP URI Topic Action](#uri-topic-action-object) field and [components](#Components-Object) field. A WampAPI document
uses and conforms to the WampAPI Specification.

## WAMP Uri Templating

WAMP Uri templating refers to the usage of template expressions, delimited by curly braces ({}), to mark a section
of a URI component as replaceable using uri parameters.

Each template expression in the uri MUST correspond to an uri parameter that is included in the
[WAMP URI RPC Action Object](#uri-rpc-action-object) or [WAMP URI Topic Action Object](#uri-topic-action-object)
itself. An exception is if the component item is empty, for example with pattern-based URIs, matching uri parameters
are not required.

The value for these uri parameters MUST NOT contain any unescaped "generic syntax" characters described
by [RFC3986][RFC3986-sec3]: forward slashes (`/`), question marks (`?`), or hashes (`#`).

## Specification

### Versions

The WampAPI Specification is versioned using a `major`.`minor`.`patch` versioning scheme. The `major`.`minor` portion of
the version string (for example `1.1`) SHALL designate the WAS feature set. *`.patch`* versions address errors in, or
provide clarifications to, this document, not the feature set. Tooling which supports WAS 1.1 SHOULD be compatible with
all WAS 1.1.\* versions. The patch version SHOULD NOT be considered by tooling, making no distinction between `1.1.0`
and `1.1.1` for example.

Occasionally, non-backwards compatible changes may be made in `minor` versions of the WAS where impact is believed to be
low relative to the benefit provided.

A WampAPI document compatible with WAS 1.\*.\* contains a required `WampAPI Version` field which designates the
version of the WAS that it uses.

### Format

A WampAPI document that conforms to the WampAPI Specification is itself a JSON object, which may be represented either
in JSON or YAML format.

All field names in the specification are **case-sensitive**. This includes all fields that are used as keys in a map,
except where explicitly noted that keys are **case-insensitive**.

The schema exposes two types of fields: `Fixed` fields, which have a declared name, and `Patterned` fields, which
declare a regex pattern for the field name.

Patterned fields MUST have unique names within the containing object.

In order to preserve the ability to round-trip between YAML and JSON formats, [YAML version 1.2][Yaml-1.2] is
RECOMMENDED along with some additional constraints:

- Tags MUST be limited to those allowed by the [JSON Schema ruleset][JSON Schema ruleset].
- Keys used in YAML maps MUST be limited to a scalar string, as defined by
  the [YAML Failsafe schema ruleset][YAML Failsafe schema ruleset].

**Note:** While APIs may be defined by WampAPI documents in either YAML or JSON format, the API request and response
payloads and other content are not required to be JSON or YAML.

### Document Structure

A WampAPI document MAY be made up of a single document or be divided into multiple, connected parts at the discretion
of the author. In the latter case, [Reference Objects](#reference-Object) and [Schema Object](#schema-Object) `$ref`
keywords are used.

It is RECOMMENDED that the root WampAPI document be named: `WampAPI.json` or `WampAPI.yaml`.

### Data Types

Data types in the WAS are based on the types supported by
the [JSON Schema Specification Draft 2020-12][JSON Schema Spec 2022-06].
Note that `integer` as a type is also supported and is defined as a JSON number without a fraction or exponent part.
Models are defined using the [Schema Object](#schema-Object), which is a superset of JSON Schema Specification Draft
2020-12.

As defined by the
[JSON Schema Validation vocabulary][JSON Schema Validation vocabulary], data types can have an optional modifier
property: `format`. WAS defines additional formats to provide fine detail for primitive data types.

The formats defined by the WAS are:

| Common Name | Type      | Format      | Comments                       |
|-------------|-----------|-------------|--------------------------------|
| integer     | `integer` | `int32`     | signed 32 bits                 |
| long        | `integer` | `int64`     | signed 64 bits                 |
| float       | `number`  | `float`     |                                |
| double      | `number`  | `double`    |                                |
| string      | `string`  |             |                                |
| string      | `string`  | `password`  | A hint to UIs to obscure input |
| byte        | `string`  | `byte`      |                                |
| boolean     | `boolean` |             |                                |
| date        | `string`  | `date`      |                                |
| dateTime    | `string`  | `date-time` |                                |

### Rich Text Formatting

Throughout the specification `description` fields are noted as supporting CommonMark Markdown formatting.
Where WampAPI tooling renders rich text it MUST support, at a minimum, Markdown syntax as described
by [CommonMark 0.27][CommonMark 0.27]. Tooling MAY choose to ignore some CommonMark features to
address security concerns.

### Relative References in URIs

Unless specified otherwise, all properties that are URIs MAY be relative references as defined
by [RFC3986][RFC3986-sec4.2].

Relative references, including those in [Reference Objects](#reference-Object),
[Uri RPC Action Object](#uri-rpc-action-object) `$ref` fields, [Uri Topic Action Object](#uri-topic-action-object)
`$ref` fields and [Example Object](#example-Object) `externalValue` fields, are resolved using the referring document
as the Base URI according to [RFC3986][RFC3986-sec5.2].

If a URI contains a fragment identifier, then the fragment should be resolved per the fragment resolution mechanism of
the referenced document. If the representation of the referenced document is JSON or YAML, then the fragment identifier
SHOULD be interpreted as a JSON-Pointer as per [RFC6901][RFC6901].

Relative references in [Schema Objects](#schema-Object), including any that appear as `$id` values, use the nearest
parent `$id` as a Base URI, as described by [JSON Schema Specification Draft 2020-12][JSON Schema Spec 2022-06 sec8.2].
If no parent schema contains an `$id`, then the Base URI MUST be determined according to [RFC3986][RFC3986-sec5.1].

### Relative References in URLs

Unless specified otherwise, all properties that are URLs MAY be relative references as defined
by [RFC3986][RFC3986-sec4.2]. Unless specified otherwise, relative references are resolved using the URLs defined in
the [Server Object](#server-Object) as a Base URL. Note that these themselves MAY be relative to the referring
document.

### Schema

In the following description, if a field is not explicitly **REQUIRED** or described with a MUST or SHALL, it can be
considered OPTIONAL.

#### WampAPI Object

This is the root object of the [WampAPI document](#WampAPI-Document).

##### Fixed Fields

| Field Name        | Type                                                            | Description                                                                                                                                                                                                                                                                                                                                                                                               |
|-------------------|-----------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| WampAPI           | `string`                                                        | **REQUIRED**. This string MUST be the [version number](#versions) of the WampAPI Specification that the WampAPI document uses. The `WampAPI` field SHOULD be used by tooling to interpret the WampAPI document. This is *not* related to the API [Info Object](#info-Object) version attribute.                                                                                                           |
| info              | [Info Object](#info-Object)                                     | **REQUIRED**. Provides metadata about the API. The metadata MAY be used by tooling as required.                                                                                                                                                                                                                                                                                                           |
| jsonSchemaDialect | `string`                                                        | The default value for the `$schema` keyword within [Schema Objects](#schema-Object) contained within this WAS document. This MUST be in the form of a URI.                                                                                                                                                                                                                                                |
| servers           | [[Server Object](#server-Object)]                               | An array of Server Objects, which provide connectivity information to a target server(s). If the `servers` property is not provided, or is an empty array, the default value would be a [Server Object](#server-Object) with an `url` value of `/`.                                                                                                                                                       |
| components        | [Components Object](#components-Object)                         | An element to hold various schemas for the document.                                                                                                                                                                                                                                                                                                                                                      |
| uris              | [URIs Object](#uris-Object)                                     | The available WAMP URIs Actions within the API.                                                                                                                                                                                                                                                                                                                                                           |
| security          | [[Security Requirement Object](#security-Requirement-Object)]   | A declaration of which security mechanisms can be used across the API. The list of values includes alternative security requirement objects that can be used. Only one of the security requirement objects need to be satisfied to authorize a request. Individual operations can override this definition. To make security optional, an empty security requirement (`{}`) can be included in the array. |
| tags              | [[Tag Object](#tag-Object)]                                     | A list of tags used by the document with additional metadata. The order of the tags can be used to reflect on their order by the parsing tools. Not all tags that are used by the URI Action Object must be declared. The tags that are not declared MAY be organized randomly or based on the tools' logic. Each tag name in the list MUST be unique.                                                    |
| externalDocs      | [External Documentation Object](#external-Documentation-Object) | Additional external documentation.                                                                                                                                                                                                                                                                                                                                                                        |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

#### Info Object

The object provides metadata about the API.
The metadata MAY be used by the clients if needed, and MAY be presented in editing or documentation generation tools
for convenience.

##### Fixed Fields

| Field Name     | Type                              | Description                                                                                                                                                    |
|----------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| title          | `string`                          | **REQUIRED**. The title of the API.                                                                                                                            |
| summary        | `string`                          | A short summary of the API.                                                                                                                                    |
| description    | `string`                          | A description of the API. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                     |
| termsOfService | `string`                          | A URL to the Terms of Service for the API. This MUST be in the form of a URL.                                                                                  |
| contact        | [Contact Object](#contact-Object) | The contact information for the exposed API.                                                                                                                   |
| license        | [License Object](#license-Object) | The license information for the exposed API.                                                                                                                   |
| version        | `string`                          | **REQUIRED**. The version of the WampAPI document (which is distinct from the [WampAPI Specification version](#Versions) or the API implementation version).   |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Info Object Example

```json
{
    "title": "Sample Pet Store App",
    "summary": "A pet store manager.",
    "description": "This is a sample server for a pet store.",
    "termsOfService": "https://example.com/terms/",
    "contact": {
        "name": "API Support",
        "url": "https://www.example.com/support",
        "email": "support@example.com"
    },
    "license": {
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html"
    },
    "version": "1.0.1"
}
```

```yaml
title: Sample Pet Store App
summary: A pet store manager.
description: This is a sample server for a pet store.
termsOfService: https://example.com/terms/
contact:
    name: API Support
    url: https://www.example.com/support
    email: support@example.com
license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html
version: 1.0.1
```

#### Contact Object

Contact information for the exposed API.

##### Fixed Fields

| Field Name | Type     | Description                                                                                         |
|------------|----------|-----------------------------------------------------------------------------------------------------|
| name       | `string` | The identifying name of the contact person/organization.                                            |
| url        | `string` | The URL pointing to the contact information. This MUST be in the form of a URL.                     |
| email      | `string` | The email address of the contact person/organization. This MUST be in the form of an email address. |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Contact Object Example

```json
{
    "name": "API Support",
    "url": "https://www.example.com/support",
    "email": "support@example.com"
}
```

```yaml
name: API Support
url: https://www.example.com/support
email: support@example.com
```

#### License Object

License information for the exposed API.

##### Fixed Fields

| Field Name | Type     | Description                                                                                                                                |
|------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------|
| name       | `string` | **REQUIRED**. The license name used for the API.                                                                                           |
| identifier | `string` | An [SPDX][SPDX] license expression for the API. The `identifier` field is mutually exclusive of the `url` field.                           |
| url        | `string` | A URL to the license used for the API. This MUST be in the form of a URL. The `url` field is mutually exclusive of the `identifier` field. |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### License Object Example

```json
{
    "name": "Apache 2.0",
    "identifier": "Apache-2.0"
}
```

```yaml
name: The MIT License
url: https://opensource.org/licenses/MIT
```

#### Server Object

An object representing a WAMP Server (Router) Connection.

##### Fixed Fields

| Field Name  | Type                                                             | Description                                                                                                                                                                                                                                                                                 |
|-------------|------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| url         | `string`                                                         | **REQUIRED**. A URL to the target host.  This URL supports Server Variables and MAY be relative, to indicate that the host location is relative to the location where the WampAPI document is being served. Variable substitutions will be made when a variable is named in `{`brackets`}`. |
| realm       | `string`                                                         | **REQUIRED**. A WAMP Realm to join at the specified URL endpoint. Realm supports user input, so it's value may be omitted in the specification and may be entered by user if document generation tools provide such functionality.                                                          |
| description | `string`                                                         | An optional string describing the host designated by the URL. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                              |
| variables   | Map[`string`, [Server Variable Object](#server-Variable-Object)] | A map between a variable name and its value.  The value is used for substitution in the server's URL template.                                                                                                                                                                              |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Server Object Example

A single server would be described as:

```json
{
    "url": "https://development.gigantic-server.com/ws/",
    "realm": "organization_realm",
    "description": "Development server"
}
```

```yaml
url: https://development.gigantic-server.com/ws
realm: organization_realm
description: Development server
```

The following shows how multiple servers can be described, for example, at the [WampAPI
Object's servers attribute](#WampAPI-Object):

```json
{
    "servers": [
        {
            "url": "https://development.gigantic-server.com:8080/ws",
            "realm": "dev",
            "description": "Development server"
        },
        {
            "url": "https://staging.gigantic-server.com/ws",
            "realm": "stage",
            "description": "Staging server"
        },
        {
            "url": "https://api.gigantic-server.com/ws",
            "realm": "prod",
            "description": "Production server"
        }
    ]
}
```

```yaml
servers:
    -   url: https://development.gigantic-server.com:8080/ws
        realm: dev
        description: Development server
    -   url: https://staging.gigantic-server.com/ws
        realm: stage
        description: Staging server
    -   url: https://api.gigantic-server.com/ws
        realm: prod
        description: Production server
```

The following shows how variables can be used for a server configuration:

```json
{
    "servers": [
        {
            "url": "https://{username}.gigantic-server.com:{port}/{basePath}",
            "realm": "my-env-realm",
            "description": "The production API server",
            "variables": {
                "username": {
                    "default": "demo",
                    "description": "The username used for http authorization to access the WAMP endpoint"
                },
                "port": {
                    "enum": [
                        "8443",
                        "443"
                    ],
                    "default": "8443"
                },
                "basePath": {
                    "default": "ws"
                }
            }
        }
    ]
}
```

```yaml
servers:
    -   url: https://{username}.gigantic-server.com:{port}/{basePath}
        realm: prod-env-realm
        description: The production API server
        variables:
            username:
                # note! no enum here means it is an open value
                default: demo
                description: The username used for http authorization to access the WAMP endpoint
            port:
                enum:
                    - '8443'
                    - '443'
                default: '8443'
            basePath:
                # open meaning there is the opportunity to use special base paths as assigned by the provider, default is `ws`
                default: ws
```

#### Server Variable Object

An object representing a Server Variable for server URL template substitution.

##### Fixed Fields

| Field Name  | Type     | Description                                                                                                                                                                                                                                                                                                                                                    |
|-------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| enum        | [string] | An enumeration of string values to be used if the substitution options are from a limited set. The array MUST NOT be empty.                                                                                                                                                                                                                                    |
| default     | `string` | **REQUIRED**. The default value to use for substitution, which SHALL be sent if an alternate value is _not_ supplied. Note this behavior is different than the [Schema Object's](#schema-Object) treatment of default values, because in those cases parameter values are optional. If the _enum_ field is defined, the value MUST exist in the enum's values. |
| description | `string` | An optional description for the server variable. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                                                              |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

#### Components Object

Holds a set of reusable objects for different aspects of the WAS. All objects defined within the components object
will have no effect on the API unless they are explicitly referenced from properties outside the components object.

##### Fixed Fields

| Field Name      | Type                                                                                                      | Description                                                                   |
|-----------------|-----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| schemas         | Map[`string`, [Schema Object](#schema-Object)]                                                            | An object to hold reusable [Schema Objects](#schema-Object)                   |
| parameters      | Map[`string`, [Parameter Object](#parameter-Object) or [Reference Object](#reference-Object)]             | An object to hold reusable [Parameter Objects](#parameter-Object)             |
| requests        | Map[`string`, [Request Object](#request-Object) or [Reference Object](#reference-Object)]                 | An object to hold reusable [Request Objects](#request-Object)                 |
| responses       | Map[`string`, [Response Object](#response-Object) or [Reference Object](#reference-Object)]               | An object to hold reusable [Response Objects](#response-Object)               |
| events          | Map[`string`, [Event Object](#event-Object) or [Reference Object](#reference-Object)]                     | An object to hold reusable [Event Objects](#event-Object)                     |
| errors          | Map[`string`, [Error Object](#error-Object) or [Reference Object](#reference-Object)]                     | An object to hold reusable [Error Objects](#error-Object)                     |
| examples        | Map[`string`, [Example Object](#example-Object) or [Reference Object](#reference-Object)]                 | An object to hold reusable [Example Objects](#example-Object)                 |
| securitySchemes | Map[`string`, [Security Scheme Object](#security-Scheme-Object) or [Reference Object](#reference-Object)] | An object to hold reusable [Security Scheme Objects](#security-Scheme-Object) |
| links           | Map[`string`, [Link Object](#link-Object) or [Reference Object](#reference-Object)]                       | An object to hold reusable [Link Objects](#link-Object)                       |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

All the fixed fields declared above are objects that MUST use keys that match the regular
expression: `^[a-zA-Z0-9\.\-_]+$`.

Field Name Examples:

```
User
User_1
User_Name
user-name
my.org.User
```

##### Components Object Example

*****FIXME*****: Adopt Components Object Example

```json
{
    "components": {
        "schemas": {
            "GeneralError": {
                "type": "object",
                "properties": {
                    "code": {
                        "type": "integer",
                        "format": "int32"
                    },
                    "message": {
                        "type": "string"
                    }
                }
            },
            "Category": {
                "type": "object",
                "properties": {
                    "id": {
                        "type": "integer",
                        "format": "int64"
                    },
                    "name": {
                        "type": "string"
                    }
                }
            },
            "Tag": {
                "type": "object",
                "properties": {
                    "id": {
                        "type": "integer",
                        "format": "int64"
                    },
                    "name": {
                        "type": "string"
                    }
                }
            }
        },
        "parameters": {
            "skipParam": {
                "name": "skip",
                "in": "query",
                "description": "number of items to skip",
                "required": true,
                "schema": {
                    "type": "integer",
                    "format": "int32"
                }
            },
            "limitParam": {
                "name": "limit",
                "in": "query",
                "description": "max records to return",
                "required": true,
                "schema": {
                    "type": "integer",
                    "format": "int32"
                }
            }
        },
        "responses": {
            "NotFound": {
                "description": "Entity not found."
            },
            "IllegalInput": {
                "description": "Illegal input for operation."
            },
            "GeneralError": {
                "description": "General Error",
                "content": {
                    "application/json": {
                        "schema": {
                            "$ref": "#/components/schemas/GeneralError"
                        }
                    }
                }
            }
        },
        "securitySchemes": {
            "api_key": {
                "type": "apiKey",
                "name": "api_key",
                "in": "header"
            },
            "petstore_auth": {
                "type": "oauth2",
                "flows": {
                    "implicit": {
                        "authorizationUrl": "https://example.org/api/oauth/dialog",
                        "scopes": {
                            "write:pets": "modify pets in your account",
                            "read:pets": "read your pets"
                        }
                    }
                }
            }
        }
    }
}
```

*****FIXME*****: Adopt Components Object Example

```yaml
components:
    schemas:
        GeneralError:
            type: object
            properties:
                code:
                    type: integer
                    format: int32
                message:
                    type: string
        Category:
            type: object
            properties:
                id:
                    type: integer
                    format: int64
                name:
                    type: string
        Tag:
            type: object
            properties:
                id:
                    type: integer
                    format: int64
                name:
                    type: string
    parameters:
        skipParam:
            name: skip
            in: query
            description: number of items to skip
            required: true
            schema:
                type: integer
                format: int32
        limitParam:
            name: limit
            in: query
            description: max records to return
            required: true
            schema:
                type: integer
                format: int32
    responses:
        NotFound:
            description: Entity not found.
        IllegalInput:
            description: Illegal input for operation.
        GeneralError:
            description: General Error
            content:
                application/json:
                    schema:
                        $ref: '#/components/schemas/GeneralError'
    securitySchemes:
        api_key:
            type: apiKey
            name: api_key
            in: header
        petstore_auth:
            type: oauth2
            flows:
                implicit:
                    authorizationUrl: https://example.org/api/oauth/dialog
                    scopes:
                        write:pets: modify pets in your account
                        read:pets: read your pets
```

#### URIs Object

Holds the Map of WAMP Topic and RPC URIs and their actions.
The URIs MAY be empty, due to [Access Control List (ACL) constraints](#security-Filtering).

##### Patterned URI Actions

WAMP URI Action may include parameterized URI components based on [WAMP Uri Templating](#WAMP-Uri-Templating).
When matching URIs, concrete (non-templated) components would be matched before their templated counterparts. Templated
components with the same hierarchy but different templated names MUST NOT exist as they are identical. In case of
ambiguous matching, it's up to the tooling to decide which one to use.

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Uri templating Matching

Assuming the following uris, the concrete definition, `com.store.pets.mine`, will be matched first if used:

```
  com.store.pets.{petId}
  com.store.pets.mine
```

The following uris are considered identical and invalid:

```
  com.store.pets.{petId}
  com.store.pets.{name}
```

The following may lead to ambiguous resolution:

```
  com.store.{entity}.me
  com.store.books.{id}
```

##### URIs Object Example

*****FIXME*****: adopt URIs Object Example

```json
{
    "com.store.pets.get": {
        "get": {
            "description": "Returns all pets from the system that the user has access to",
            "responses": {
                "200": {
                    "description": "A list of pets.",
                    "content": {
                        "application/json": {
                            "schema": {
                                "type": "array",
                                "items": {
                                    "$ref": "#/components/schemas/pet"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

```yaml
com.store.pets.get:
    get:
        description: Returns all pets from the system that the user has access to
        responses:
            '200':
                description: A list of pets.
                content:
                    application/json:
                        schema:
                            type: array
                            items:
                                $ref: '#/components/schemas/pet'
```

#### URI RPC Action Object

Describes the single WAMP RPC Call.

##### Fixed Fields

| Field Name                 | Type                                                                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|----------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type                       | `string`                                                                         | **REQUIRED**. The type of WAMP URI. Must be set to "rpc".                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| summary                    | `string`                                                                         | A short summary of what the operation does.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| description                | `string`                                                                         | A verbose explanation of the operation behavior. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                                                                                                                                                                                                                                |
| tags                       | [string]                                                                         | A list of tags for API documentation control. Tags can be used for logical grouping of actions by resources or any other qualifier.                                                                                                                                                                                                                                                                                                                                                                                              |
| deprecated                 | `boolean`                                                                        | Declares this action to be deprecated. Consumers SHOULD refrain from usage of the declared action. Default value is `false`.                                                                                                                                                                                                                                                                                                                                                                                                     |
| externalDocs               | [External Documentation Object](#external-Documentation-Object)                  | Additional external documentation for this operation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| parameters                 | [[Parameter Object](#parameter-Object) or [Reference Object](#reference-Object)] | A list of parameters that are applicable for this operation. The list MUST NOT include duplicated parameters. A unique parameter is defined by a name. The list can use the [Reference Object](#reference-Object) to link to parameters that are defined at the [WampAPI Object's components](#Components-Object) `parameters`.                                                                                                                                                                                                  |
| request                    | [Request Object](#request-Object) or [Reference Object](#reference-Object)       | The request payload applicable for this operation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| response                   | [Response Object](#response-Object)                                              | The list of possible responses as they are returned from executing this action.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| errors                     | [[Error Object](#error-Object)]                                                  | The list of possible errored responses that can be thrown during executing this action.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| security                   | [[Security Requirement Object](#security-Requirement-Object)]                    | A declaration of which security mechanisms can be used for this operation. The list of values includes alternative security requirement objects that can be used. Only one of the security requirement objects need to be satisfied to authorize a request. To make security optional, an empty security requirement (`{}`) can be included in the array. This definition overrides any declared top-level [WampAPI Object](#WampAPI-Object) `security`. To remove a top-level security declaration, an empty array can be used. |
| supportsProgressiveCalls   | `boolean`                                                                        | Determines if the RPC supports `Progressive calls` Advanced Profile WAMP Feature. Defaults to `false`.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| supportsProgressiveResults | `boolean`                                                                        | Determines if the RPC supports `Progressive call results` Advanced Profile WAMP Feature. Defaults to `false`.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| supportsE2EE               | `boolean`                                                                        | Determines if the RPC supports `End-2-End Encryption` Advanced Profile WAMP Feature. Defaults to `false`.                                                                                                                                                                                                                                                                                                                                                                                                                        |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### URI RPC Action Example

*****FIXME*****: adopt URI RPC Action Example

```json
{
    "tags": [
        "pet"
    ],
    "summary": "Updates a pet in the store with form data",
    "operationId": "updatePetWithForm",
    "parameters": [
        {
            "name": "petId",
            "in": "path",
            "description": "ID of pet that needs to be updated",
            "required": true,
            "schema": {
                "type": "string"
            }
        }
    ],
    "requestBody": {
        "content": {
            "application/x-www-form-urlencoded": {
                "schema": {
                    "type": "object",
                    "properties": {
                        "name": {
                            "description": "Updated name of the pet",
                            "type": "string"
                        },
                        "status": {
                            "description": "Updated status of the pet",
                            "type": "string"
                        }
                    },
                    "required": [
                        "status"
                    ]
                }
            }
        }
    },
    "responses": {
        "200": {
            "description": "Pet updated.",
            "content": {
                "application/json": {},
                "application/xml": {}
            }
        },
        "405": {
            "description": "Method Not Allowed",
            "content": {
                "application/json": {},
                "application/xml": {}
            }
        }
    },
    "security": [
        {
            "petstore_auth": [
                "write:pets",
                "read:pets"
            ]
        }
    ]
}
```

```yaml
tags:
    - pet
summary: Updates a pet in the store with form data
operationId: updatePetWithForm
parameters:
    -   name: petId
        in: path
        description: ID of pet that needs to be updated
        required: true
        schema:
            type: string
requestBody:
    content:
        'application/x-www-form-urlencoded':
            schema:
                type: object
                properties:
                    name:
                        description: Updated name of the pet
                        type: string
                    status:
                        description: Updated status of the pet
                        type: string
                required:
                    - status
responses:
    '200':
        description: Pet updated.
        content:
            'application/json': { }
            'application/xml': { }
    '405':
        description: Method Not Allowed
        content:
            'application/json': { }
            'application/xml': { }
security:
    -   petstore_auth:
            - write:pets
            - read:pets
```

#### URI Topic Action Object

Describes the single WAMP Events Topic Publication/Subscription.

##### Fixed Fields

| Field Name                 | Type                                                                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|----------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type                       | `string`.                                                                        | **REQUIRED**. The type of WAMP URI. Must be set to "topic".                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| summary                    | `string`                                                                         | A short summary of what the operation does.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| description                | `string`                                                                         | A verbose explanation of the operation behavior. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                                                                                                                                                                                                                                |
| tags                       | [string]                                                                         | A list of tags for API documentation control. Tags can be used for logical grouping of actions by resources or any other qualifier.                                                                                                                                                                                                                                                                                                                                                                                              |
| deprecated                 | `boolean`                                                                        | Declares this action to be deprecated. Consumers SHOULD refrain from usage of the declared action. Default value is `false`.                                                                                                                                                                                                                                                                                                                                                                                                     |
| externalDocs               | [External Documentation Object](#external-Documentation-Object)                  | Additional external documentation for this operation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| parameters                 | [[Parameter Object](#parameter-Object) or [Reference Object](#reference-Object)] | A list of parameters that are applicable for this operation. The list MUST NOT include duplicated parameters. A unique parameter is defined by a name. The list can use the [Reference Object](#reference-Object) to link to parameters that are defined at the [WampAPI Object's components](#Components-Object) `parameters`.                                                                                                                                                                                                  |
| event                      | [Event Object](#event-Object) or [Reference Object](#reference-Object)           | The event payload applicable for publishing to topic or that can be received by subscription.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| errors                     | [[Error Object](#error-Object)]                                                  | The list of possible errored responses that can be thrown during executing this action.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| security                   | [[Security Requirement Object](#security-Requirement-Object)]                    | A declaration of which security mechanisms can be used for this operation. The list of values includes alternative security requirement objects that can be used. Only one of the security requirement objects need to be satisfied to authorize a request. To make security optional, an empty security requirement (`{}`) can be included in the array. This definition overrides any declared top-level [WampAPI Object](#WampAPI-Object) `security`. To remove a top-level security declaration, an empty array can be used. |
| supportsE2EE               | `boolean`                                                                        | Determines if the URI Action (RPC or events topic) supports `End-2-End Encryption` Advanced Profile WAMP Feature. Defaults to `false`.                                                                                                                                                                                                                                                                                                                                                                                           |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### URI Topic Action Example

*****FIXME*****: adopt URI RPC Action Example

```json
{
    "tags": [
        "pet"
    ],
    "summary": "Updates a pet in the store with form data",
    "operationId": "updatePetWithForm",
    "parameters": [
        {
            "name": "petId",
            "in": "path",
            "description": "ID of pet that needs to be updated",
            "required": true,
            "schema": {
                "type": "string"
            }
        }
    ],
    "requestBody": {
        "content": {
            "application/x-www-form-urlencoded": {
                "schema": {
                    "type": "object",
                    "properties": {
                        "name": {
                            "description": "Updated name of the pet",
                            "type": "string"
                        },
                        "status": {
                            "description": "Updated status of the pet",
                            "type": "string"
                        }
                    },
                    "required": [
                        "status"
                    ]
                }
            }
        }
    },
    "responses": {
        "200": {
            "description": "Pet updated.",
            "content": {
                "application/json": {},
                "application/xml": {}
            }
        },
        "405": {
            "description": "Method Not Allowed",
            "content": {
                "application/json": {},
                "application/xml": {}
            }
        }
    },
    "security": [
        {
            "petstore_auth": [
                "write:pets",
                "read:pets"
            ]
        }
    ]
}
```

```yaml
tags:
    - pet
summary: Updates a pet in the store with form data
operationId: updatePetWithForm
parameters:
    -   name: petId
        in: path
        description: ID of pet that needs to be updated
        required: true
        schema:
            type: string
requestBody:
    content:
        'application/x-www-form-urlencoded':
            schema:
                type: object
                properties:
                    name:
                        description: Updated name of the pet
                        type: string
                    status:
                        description: Updated status of the pet
                        type: string
                required:
                    - status
responses:
    '200':
        description: Pet updated.
        content:
            'application/json': { }
            'application/xml': { }
    '405':
        description: Method Not Allowed
        content:
            'application/json': { }
            'application/xml': { }
security:
    -   petstore_auth:
            - write:pets
            - read:pets
```

#### External Documentation Object

Allows referencing an external resource for extended documentation.

##### Fixed Fields

| Field Name  | Type     | Description                                                                                                                 |
|-------------|----------|-----------------------------------------------------------------------------------------------------------------------------|
| description | `string` | A description of the target documentation. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation. |
| url         | `string` | **REQUIRED**. The URL for the target documentation. This MUST be in the form of a URL.                                      |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### External Documentation Object Example

```json
{
    "description": "Find more info here",
    "url": "https://example.com"
}
```

```yaml
description: Find more info here
url: https://example.com
```

#### Parameter Object

Describes a single action URI component parameter.

##### Fixed Fields

| Field Name      | Type      | Description                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name            | `string`  | **REQUIRED**. The name of the parameter. Parameter names are *case sensitive*. The `name` field MUST correspond to a template expression occurring within the action URI String. See [Uri templating](#WAMP-Uri-Templating) for further information.                                                                                                                                                         |
| description     | `string`  | A brief description of the parameter. This could contain examples of use. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                                                                                   |

##### Parameter Object Examples

```json
{
    "name": "catalogue-section",
    "description": "A catalog section name"
}
```

```yaml
name: catalogue-section
description: A catalog section name
```

#### Request Object

Describes a single RPC request payload.

##### Fixed Fields

| Field Name          | Type               | Description                                                                                                                                                                                                                                              |
|---------------------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| description         | `string`           | A brief description of the request payload. This could contain examples of use.  [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                        |
| args                | [any]              | The contents of WAMP Message `Arguments list`. This if present MUST be an array with any number of items (0, 1 or more). In case when `ArgumentsKw dict` is declared, but `Arguments list` is omitted, empty array `[]` will be sent as `Arguments list` |
| kwargs              | Map[`string`, any] | The contents of WAMP Message `ArgumentsKw dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).                                                                              |
| required            | `boolean`          | Determines if the request payload is required for the action. Defaults to `false`.                                                                                                                                                                       |

##### Request Examples

A request body with a referenced model definition.

*****FIXME*****: adopt Request Examples

```json
{
    "description": "user to add to the system",
    "content": {
        "application/json": {
            "schema": {
                "$ref": "#/components/schemas/User"
            },
            "examples": {
                "user": {
                    "summary": "User Example",
                    "externalValue": "https://foo.bar/examples/user-example.json"
                }
            }
        },
        "application/xml": {
            "schema": {
                "$ref": "#/components/schemas/User"
            },
            "examples": {
                "user": {
                    "summary": "User example in XML",
                    "externalValue": "https://foo.bar/examples/user-example.xml"
                }
            }
        },
        "text/plain": {
            "examples": {
                "user": {
                    "summary": "User example in Plain text",
                    "externalValue": "https://foo.bar/examples/user-example.txt"
                }
            }
        },
        "*/*": {
            "examples": {
                "user": {
                    "summary": "User example in other format",
                    "externalValue": "https://foo.bar/examples/user-example.whatever"
                }
            }
        }
    }
}
```

*****FIXME*****: adopt Request Examples

```yaml
description: user to add to the system
content:
    'application/json':
        schema:
            $ref: '#/components/schemas/User'
        examples:
            user:
                summary: User Example
                externalValue: 'https://foo.bar/examples/user-example.json'
    'application/xml':
        schema:
            $ref: '#/components/schemas/User'
        examples:
            user:
                summary: User example in XML
                externalValue: 'https://foo.bar/examples/user-example.xml'
    'text/plain':
        examples:
            user:
                summary: User example in Plain text
                externalValue: 'https://foo.bar/examples/user-example.txt'
    '*/*':
        examples:
            user:
                summary: User example in other format
                externalValue: 'https://foo.bar/examples/user-example.whatever'
```

A body parameter that is an array of string values:

*****FIXME*****: adopt Request Examples

```json
{
    "description": "user to add to the system",
    "required": true,
    "content": {
        "text/plain": {
            "schema": {
                "type": "array",
                "items": {
                    "type": "string"
                }
            }
        }
    }
}
```

```yaml
description: user to add to the system
required: true
content:
    text/plain:
        schema:
            type: array
            items:
                type: string
```

#### Response Object

Describes a single RPC response payload.

##### Fixed Fields

| Field Name  | Type               | Description                                                                                                                                                                                                             |
|-------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| description | `string`           | A brief description of the response payload. This could contain examples of use.  [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                      |
| args        | [any]              | The contents of WAMP Message `Arguments list`. This if present MUST be an array. In case when `Arguments list` is omitted by the sender but `ArgumentsKw dict` is present, `Arguments list` will be an empty array `[]` |
| kwargs      | Map[`string`, any] | The contents of WAMP Message `ArgumentsKw dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).                                             |
| details     | Map[`string`, any] | The contents of WAMP Message `Details dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).                                                 |

##### Response Object Examples

Response of an array of a complex type:

*****FIXME*****: adopt Response Object Examples

```json
{
    "description": "A complex object array response",
    "content": {
        "application/json": {
            "schema": {
                "type": "array",
                "items": {
                    "$ref": "#/components/schemas/VeryComplexType"
                }
            }
        }
    }
}
```

*****FIXME*****: adopt Response Object Examples

```yaml
description: A complex object array response
content:
    application/json:
        schema:
            type: array
            items:
                $ref: '#/components/schemas/VeryComplexType'
```

Response with a string type:

*****FIXME*****: adopt Response Object Examples

```json
{
    "description": "A simple string response",
    "content": {
        "text/plain": {
            "schema": {
                "type": "string"
            }
        }
    }
}
```

```yaml
description: A simple string response
content:
    text/plain:
        schema:
            type: string
```

Response with no return value:

```json
{
    "description": "object created"
}
```

```yaml
description: object created
```

#### Event Object

Describes a single Topic event payload.

##### Fixed Fields

| Field Name  | Type               | Description                                                                                                                                                                                                             |
|-------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| description | `string`           | A brief description of the event payload. This could contain examples of use.  [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                         |
| args        | [any]              | The contents of WAMP Message `Arguments list`. This if present MUST be an array. In case when `Arguments list` is omitted by the sender but `ArgumentsKw dict` is present, `Arguments list` will be an empty array `[]` |
| kwargs      | Map[`string`, any] | The contents of WAMP Message `ArgumentsKw dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).                                             |
| details     | Map[`string`, any] | The contents of WAMP Message `Details dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).                                                 |

##### Event Object Examples

Event of an array of a complex type:

*****FIXME*****: adopt Event Object Examples

```json
{
    "description": "A complex object array event",
    "content": {
        "application/json": {
            "schema": {
                "type": "array",
                "items": {
                    "$ref": "#/components/schemas/VeryComplexType"
                }
            }
        }
    }
}
```

*****FIXME*****: adopt Event Object Examples

```yaml
description: A complex object array event
content:
    application/json:
        schema:
            type: array
            items:
                $ref: '#/components/schemas/VeryComplexType'
```

Event with a string type:

*****FIXME*****: adopt Event Object Examples

```json
{
    "description": "A simple string event",
    "content": {
        "text/plain": {
            "schema": {
                "type": "string"
            }
        }
    }
}
```

```yaml
description: A simple string event
content:
    text/plain:
        schema:
            type: string
```

Event with no return value:

```json
{
    "description": "object created"
}
```

```yaml
description: object created
```

#### Error Object

Describes a single action processing error. This can be error during RPC Call or during event publication.

##### Fixed Fields

| Field Name  | Type               | Description                                                                                                                                                                  |
|-------------|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| error       | `string`           | **REQUIRED**. An Error URI. In most cases it will be WAMP Specification predefined URI.                                                                                      |
| description | `string`           | A brief description of the error. This could contain examples of use.  [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                      |
| details     | Map[`string`, any] | The contents of Error `Details dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects).             |
| args        | [any]              | The contents of Error Message `Arguments list`. This if present MUST be an array.                                                                                            |
| kwargs      | Map[`string`, any] | The contents of Error Message `ArgumentsKw dict`. This if present MUST be an object with any number of `string` keys and any supported values (primitives, arrays, objects). |

For RPC Calls Error Response may include the original error payload as returned by the Callee to the Dealer. Tools
may automatically extend Error object with `args` and `kwargs` schemas taken from the RPC request definition.

##### Error Object Examples

*****FIXME*****: Provide Error Object Examples

#### Example Object

Example object holds some payload schema with filled values.

##### Fixed Fields

| Field Name    | Type     | Description                                                                                                                                                                                                                                                                                                  |
|---------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| summary       | `string` | Short description for the example.                                                                                                                                                                                                                                                                           |
| description   | `string` | Long description for the example. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                           |
| value         | Any      | Embedded literal example. The `value` field and `externalValue` field are mutually exclusive. To represent examples of media types that cannot naturally be represented in JSON or YAML, use a string value to contain the example, escaping where necessary.                                                |
| externalValue | `string` | A URI that points to the literal example. This provides the capability to reference examples that cannot easily be included in JSON or YAML documents.  The `value` field and `externalValue` field are mutually exclusive. See the rules for resolving [Relative References](#relative-References-in-URIs). |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

In all cases, the example value is expected to be compatible with the type schema of its associated value. Tooling
implementations MAY choose to validate compatibility automatically, and reject the example value(s) if incompatible.

##### Example Object Examples

*****FIXME*****: adopt Example Object Examples

In a request body:

```yaml
requestBody:
    content:
        'application/json':
            schema:
                $ref: '#/components/schemas/Address'
            examples:
                foo:
                    summary: A foo example
                    value: { "foo": "bar" }
                bar:
                    summary: A bar example
                    value: { "bar": "baz" }
        'application/xml':
            examples:
                xmlExample:
                    summary: This is an example in XML
                    externalValue: 'https://example.org/examples/address-example.xml'
        'text/plain':
            examples:
                textExample:
                    summary: This is a text example
                    externalValue: 'https://foo.bar/examples/address-example.txt'
```

In a parameter:

```yaml
parameters:
    -   name: 'zipCode'
        in: 'query'
        schema:
            type: 'string'
            format: 'zip-code'
        examples:
            zip-example:
                $ref: '#/components/examples/zip-example'
```

In a response:

```yaml
responses:
    '200':
        description: your car appointment has been booked
        content:
            application/json:
                schema:
                    $ref: '#/components/schemas/SuccessResponse'
                examples:
                    confirmation-success:
                        $ref: '#/components/examples/confirmation-success'
```

#### Link Object

The `Link object` represents a possible design-time link for a response. The presence of a link does not guarantee the
caller's ability to successfully invoke it, rather it provides a known relationship and traversal mechanism between
responses and other actions.

For computing links, and providing instructions to execute them, a [runtime expression](#runtime-Expressions) is used for
accessing values in an action and using them as parameters while invoking the linked operation.

##### Fixed Fields

| Field Name   | Type                                                       | Description                                                                                                                                                                                                                             |
|--------------|------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| operationUri | `string`                                                   | An URI of existing WAS action, MUST point to an [RPC Action Object](#URI-RPC-Action-Object) or [Topic Action Object](#URI-Topic-Action-Object).                                                                                         |
| parameters   | Map[`string`, Any or [{expression}](#runtime-Expressions)] | A map representing parameters to pass to an action as specified with `operationUri`. The key is the parameter name to be used, whereas the value can be a constant or an expression to be evaluated and passed to the linked operation. |
| payload      | Any or [{expression}](#runtime-Expressions)                | A literal value or [{expression}](#runtime-Expressions) to use as a payload when calling the target action.                                                                                                                             |
| description  | `string`                                                   | A description of the link. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                             |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Examples

*****FIXME*****: adopt Link examples

Computing a link from an RPC Action where the `$request.path.id` is used to pass a request parameter to the linked
operation.

```yaml
paths:
    /users/{id}:
        parameters:
            -   name: id
                in: path
                required: true
                description: the user identifier, as userId
                schema:
                    type: string
        get:
            responses:
                '200':
                    description: the user being returned
                    content:
                        application/json:
                            schema:
                                type: object
                                properties:
                                    uuid: # the unique user id
                                        type: string
                                        format: uuid
                    links:
                        address:
                            # the target link operationId
                            operationId: getUserAddress
                            parameters:
                                # get the `id` field from the request uri parameter named `id`
                                userId: $request.path.id
    # the path item of the linked operation
    /users/{userid}/address:
        parameters:
            -   name: userid
                in: path
                required: true
                description: the user identifier, as userId
                schema:
                    type: string
        # linked operation
        get:
            operationId: getUserAddress
            responses:
                '200':
                    description: the user's address
```

When a runtime expression fails to evaluate, no parameter value is passed to the target operation.

Values from the response payload can be used to drive a linked operation.

```yaml
links:
    address:
        operationId: getUserAddressByUUID
        parameters:
            # get the `uuid` field from the `uuid` field in the response body
            userUuid: $response.body#/uuid
```

Clients follow all links at their discretion.
Neither permissions, nor the capability to make a successful call to that link, are guaranteed
solely by the existence of a relationship.

##### Runtime Expressions

Runtime expressions allow defining values based on information that will only be available within the HTTP message in an
actual API call. This mechanism is used by [Link Objects](#link-Object).

The runtime expression is defined by the following [ABNF][rfc5234] syntax

```abnf
      expression = ( "$url" / "$request." payload / "$response." payload )
      payload = "payload" ["#" json-pointer ]
      json-pointer    = *( "/" reference-token )
      reference-token = *( unescaped / escaped )
      unescaped       = %x00-2E / %x30-7D / %x7F-10FFFF
         ; %x2F ('/') and %x7E ('~') are excluded from 'unescaped'
      escaped         = "~" ( "0" / "1" )
        ; representing '~' and '/', respectively
      name = *( CHAR )
      token = 1*tchar
      tchar = "!" / "#" / "$" / "%" / "&" / "'" / "*" / "+" / "-" / "." /
        "^" / "_" / "`" / "|" / "~" / DIGIT / ALPHA
```

Here, `json-pointer` is taken from [RFC6901][RFC6901], `char` from [RFC7159][RFC7159-sec7] and
`token` from [RFC7230][RFC7230-sec3.2].

The `name` identifier is case-sensitive, whereas `token` is not.

The table below provides examples of runtime expressions and examples of their use in a value:

##### Examples

| Source Location          | example expression            | notes                                                                                                         |
|--------------------------|-------------------------------|---------------------------------------------------------------------------------------------------------------|
| URI parameter            | `uri.id`                      | URI parameters MUST be declared in the `parameters` section of the parent action or they cannot be evaluated. |
| Request payload property | `$request.payload#/user/uuid` | In actions which accept payloads, references may be made to the entire or portions of the payload.            |
| Response payload value   | `$response.payload#/status`   | In actions which return payloads, references may be made to the entire or portions of the payload.            |

Runtime expressions preserve the type of the referenced value.
Expressions can be embedded into string values by surrounding the expression with `{}` curly braces.

#### Tag Object

Adds metadata to a single tag that is used by the [WAMP URI RPC Action](#uri-rpc-action-object) or
[WAMP URI Topic Action](#uri-topic-action-object).
It is not mandatory to have a Tag Object per tag defined in the Action Object instances.

##### Fixed Fields

| Field Name   | Type                                                            | Description                                                                                                 |
|--------------|-----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| name         | `string`                                                        | **REQUIRED**. The name of the tag.                                                                          |
| description  | `string`                                                        | A description for the tag. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation. |
| externalDocs | [External Documentation Object](#external-Documentation-Object) | Additional external documentation for this tag.                                                             |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Tag Object Example

```json
{
    "name": "pet",
    "description": "Pets operations"
}
```

```yaml
name: pet
description: Pets operations
```

#### Reference Object

A simple object to allow referencing other components in the WampAPI document, internally and externally.

The `$ref` string value contains a URI [RFC3986][RFC3986], which identifies the location of
the value being referenced.

See the rules for resolving [Relative References](#relative-references-in-uris).

##### Fixed Fields

| Field Name  | Type     | Description                                                                                                                                                                                                                                                           |
|-------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| $ref        | `string` | **REQUIRED**. The reference identifier. This MUST be in the form of a URI.                                                                                                                                                                                            |
| summary     | `string` | A short summary which by default SHOULD override that of the referenced component. If the referenced object-type does not allow a `summary` field, then this field has no effect.                                                                                     |
| description | `string` | A description which by default SHOULD override that of the referenced component. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation. If the referenced object-type does not allow a `description` field, then this field has no effect.  |

This object cannot be extended with additional properties and any properties added SHALL be ignored.

Note that this restriction on additional properties is a difference between Reference Objects
and [Schema Objects](#schema-Object) that contain a `$ref` keyword.

##### Reference Object Example

```json
{
    "$ref": "#/components/schemas/Pet"
}
```

```yaml
$ref: '#/components/schemas/Pet'
```

##### Relative Schema Document Example

```json
{
    "$ref": "Pet.json"
}
```

```yaml
$ref: Pet.yaml
```

##### Relative Documents With Embedded Schema Example

```json
{
    "$ref": "definitions.json#/Pet"
}
```

```yaml
$ref: definitions.yaml#/Pet
```

#### Schema Object

The Schema Object allows the definition of input and output data types.
These types can be objects, but also primitives and arrays. This object is a superset of
the [JSON Schema Specification Draft 2020-12][JSON Schema Specification Draft 2020-12].

For more information about the properties,
see [JSON Schema Core][JSON Schema Specification Draft 2020-12]
and [JSON Schema Validation][JSON Schema Validation].

Unless stated otherwise, the property definitions follow those of JSON Schema and do not add any additional semantics.
Where JSON Schema indicates that behavior is defined by the application (e.g. for annotations), WAS also defers the
definition of semantics to the application consuming the WampAPI document.

##### Properties

The WampAPI Schema Object [dialect][JSON Schema dialect] is defined as vocabulary as specified in the JSON Schema
draft 2020-12 [general purpose meta-schema][JSON Schema meta-schema].

The following properties are taken from the JSON Schema specification but their definitions have been extended by the
WAS:

- description - [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.
- format - See [Data Type Formats](#data-types) for further details. While relying on JSON Schema's defined formats,
  the WAS offers a few additional predefined formats.

In addition to the JSON Schema properties comprising the WAS dialect, the Schema Object supports keywords from any other
vocabularies, or entirely arbitrary properties.

The WampAPI Specification's base vocabulary comprises the following keywords:

##### Fixed Fields

| Field Name    | Type                                                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|---------------|-----------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| discriminator | [Discriminator Object](#discriminator-Object)                   | Adds support for polymorphism. The discriminator is an object name that is used to differentiate between other schemas which may satisfy the payload description. See `Composition and Inheritance` section below for more details.                                                                                                                                                                                                                 |
| externalDocs  | [External Documentation Object](#external-Documentation-Object) | Additional external documentation for this schema.                                                                                                                                                                                                                                                                                                                                                                                                  |

This object MAY be extended with [Specification Extensions](#specification-Extensions), though as noted, additional
properties MAY omit the `x-` prefix within this object.

##### Composition and Inheritance (Polymorphism)

The WampAPI Specification allows combining and extending model definitions using the `allOf` property of JSON Schema, in
effect offering model composition. `allOf` takes an array of object definitions that are validated *independently* but
together compose a single object.

While composition offers model extensibility, it does not imply a hierarchy between the models. To support polymorphism,
the WampAPI Specification adds the `discriminator` field. When used, the `discriminator` will be the name of the
property that decides which schema definition validates the structure of the model. As such, the `discriminator` field
MUST be a required field. There are two ways to define the value of a discriminator for an inheriting instance.

- Use the schema name.
- Override the schema name by overriding the property with a new value. If a new value exists, this takes precedence
  over the schema name.
  As such, inline schema definitions, which do not have a given id, *cannot* be used in polymorphism.

##### Specifying Schema Dialects

It is important for tooling to be able to determine which dialect or meta-schema any given resource wishes to be
processed with: JSON Schema Core, JSON Schema Validation, WampAPI Schema dialect, or some custom meta-schema.

The `$schema` keyword MAY be present in any root Schema Object, and if present MUST be used to determine which dialect
should be used when processing the schema. This allows use of Schema Objects which comply with other drafts of JSON
Schema than the default Draft 2020-12 support. Tooling MUST support the WAS dialect schema id, and MAY support
additional values of `$schema`.

To allow use of a different default `$schema` value for all Schema Objects contained within a WAS document,
a `jsonSchemaDialect` value may be set within the [WampAPI Object](#WampAPI-Object). If this default is not set,
then the WAS dialect schema id MUST be used for these Schema Objects. The value of `$schema` within a Schema Object
always overrides any default.

When a Schema Object is referenced from an external resource which is not a WAS document (e.g. a bare JSON Schema
resource), then the value of the `$schema` keyword for schemas within that resource MUST
follow [JSON Schema rules][JSON Schema rules].

##### Schema Object Examples

###### Primitive Sample

```json
{
    "type": "string",
    "format": "email"
}
```

```yaml
type: string
format: email
```

###### Simple Model

```json
{
    "type": "object",
    "required": [
        "name"
    ],
    "properties": {
        "name": {
            "type": "string"
        },
        "address": {
            "$ref": "#/components/schemas/Address"
        },
        "age": {
            "type": "integer",
            "format": "int32",
            "minimum": 0
        }
    }
}
```

```yaml
type: object
required:
    - name
properties:
    name:
        type: string
    address:
        $ref: '#/components/schemas/Address'
    age:
        type: integer
        format: int32
        minimum: 0
```

###### Model with Map/Dictionary Properties

For a simple string to string mapping:

```json
{
    "type": "object",
    "additionalProperties": {
        "type": "string"
    }
}
```

```yaml
type: object
additionalProperties:
    type: string
```

For a string to model mapping:

```json
{
    "type": "object",
    "additionalProperties": {
        "$ref": "#/components/schemas/ComplexModel"
    }
}
```

```yaml
type: object
additionalProperties:
    $ref: '#/components/schemas/ComplexModel'
```

###### Models with Composition

```json
{
    "components": {
        "schemas": {
            "ErrorModel": {
                "type": "object",
                "required": [
                    "message",
                    "code"
                ],
                "properties": {
                    "message": {
                        "type": "string"
                    },
                    "code": {
                        "type": "integer",
                        "minimum": 100,
                        "maximum": 600
                    }
                }
            },
            "ExtendedErrorModel": {
                "allOf": [
                    {
                        "$ref": "#/components/schemas/ErrorModel"
                    },
                    {
                        "type": "object",
                        "required": [
                            "rootCause"
                        ],
                        "properties": {
                            "rootCause": {
                                "type": "string"
                            }
                        }
                    }
                ]
            }
        }
    }
}
```

```yaml
components:
    schemas:
        ErrorModel:
            type: object
            required:
                - message
                - code
            properties:
                message:
                    type: string
                code:
                    type: integer
                    minimum: 100
                    maximum: 600
        ExtendedErrorModel:
            allOf:
                -   $ref: '#/components/schemas/ErrorModel'
                -   type: object
                    required:
                        - rootCause
                    properties:
                        rootCause:
                            type: string
```

###### Models with Polymorphism Support

```json
{
    "components": {
        "schemas": {
            "Pet": {
                "type": "object",
                "discriminator": {
                    "propertyName": "petType"
                },
                "properties": {
                    "name": {
                        "type": "string"
                    },
                    "petType": {
                        "type": "string"
                    }
                },
                "required": [
                    "name",
                    "petType"
                ]
            },
            "Cat": {
                "description": "A representation of a cat. Note that `Cat` will be used as the discriminator value.",
                "allOf": [
                    {
                        "$ref": "#/components/schemas/Pet"
                    },
                    {
                        "type": "object",
                        "properties": {
                            "huntingSkill": {
                                "type": "string",
                                "description": "The measured skill for hunting",
                                "default": "lazy",
                                "enum": [
                                    "clueless",
                                    "lazy",
                                    "adventurous",
                                    "aggressive"
                                ]
                            }
                        },
                        "required": [
                            "huntingSkill"
                        ]
                    }
                ]
            },
            "Dog": {
                "description": "A representation of a dog. Note that `Dog` will be used as the discriminator value.",
                "allOf": [
                    {
                        "$ref": "#/components/schemas/Pet"
                    },
                    {
                        "type": "object",
                        "properties": {
                            "packSize": {
                                "type": "integer",
                                "format": "int32",
                                "description": "the size of the pack the dog is from",
                                "default": 0,
                                "minimum": 0
                            }
                        },
                        "required": [
                            "packSize"
                        ]
                    }
                ]
            }
        }
    }
}
```

```yaml
components:
    schemas:
        Pet:
            type: object
            discriminator:
                propertyName: petType
            properties:
                name:
                    type: string
                petType:
                    type: string
            required:
                - name
                - petType
        Cat: ## "Cat" will be used as the discriminator value
            description: A representation of a cat
            allOf:
                -   $ref: '#/components/schemas/Pet'
                -   type: object
                    properties:
                        huntingSkill:
                            type: string
                            description: The measured skill for hunting
                            enum:
                                - clueless
                                - lazy
                                - adventurous
                                - aggressive
                    required:
                        - huntingSkill
        Dog: ## "Dog" will be used as the discriminator value
            description: A representation of a dog
            allOf:
                -   $ref: '#/components/schemas/Pet'
                -   type: object
                    properties:
                        packSize:
                            type: integer
                            format: int32
                            description: the size of the pack the dog is from
                            default: 0
                            minimum: 0
                    required:
                        - packSize
```

#### Discriminator Object

When request or response payloads may be one of a number of different schemas, a `discriminator` object can be
used to aid in serialization, deserialization, and validation. The discriminator is a specific object in a schema which
is used to inform the consumer of the document of an alternative schema based on the value associated with it.

When using the discriminator, _inline_ schemas will not be considered.

##### Fixed Fields

| Field Name   | Type                    | Description                                                                                   |
|--------------|-------------------------|-----------------------------------------------------------------------------------------------|
| propertyName | `string`                | **REQUIRED**. The name of the property in the payload that will hold the discriminator value. |
| mapping      | Map[`string`, `string`] | An object to hold mappings between payload values and schema names or references.             |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

The discriminator object is legal only when using one of the composite keywords `oneOf`, `anyOf`, `allOf`.

In WAS 1.0, a response payload MAY be described to be exactly one of any number of types:

```yaml
MyResponseType:
    oneOf:
        -   $ref: '#/components/schemas/Cat'
        -   $ref: '#/components/schemas/Dog'
        -   $ref: '#/components/schemas/Lizard'
```

which means the payload _MUST_, by validation, match exactly one of the schemas described by `Cat`, `Dog`, or `Lizard`.
In this case, a discriminator MAY act as a "hint" to shortcut validation and selection of the matching schema which may
be a costly operation, depending on the complexity of the schema. We can then describe exactly which field tells us
which schema to use:

```yaml
MyResponseType:
    oneOf:
        -   $ref: '#/components/schemas/Cat'
        -   $ref: '#/components/schemas/Dog'
        -   $ref: '#/components/schemas/Lizard'
    discriminator:
        propertyName: petType
```

The expectation now is that a property with name `petType` _MUST_ be present in the response payload, and the value will
correspond to the name of a schema defined in the WAS document. Thus, the response payload:

```json
{
    "id": 12345,
    "petType": "Cat"
}
```

Will indicate that the `Cat` schema be used in conjunction with this payload.

In scenarios where the value of the discriminator field does not match the schema name or implicit mapping is not
possible, an optional `mapping` definition MAY be used:

```yaml
MyResponseType:
    oneOf:
        -   $ref: '#/components/schemas/Cat'
        -   $ref: '#/components/schemas/Dog'
        -   $ref: '#/components/schemas/Lizard'
        -   $ref: 'https://gigantic-server.com/schemas/Monster/schema.json'
    discriminator:
        propertyName: petType
        mapping:
            dog: '#/components/schemas/Dog'
            monster: 'https://gigantic-server.com/schemas/Monster/schema.json'
```

Here the discriminator _value_ of `dog` will map to the schema `#/components/schemas/Dog`, rather than the default (
implicit) value of `Dog`. If the discriminator _value_ does not match an implicit or explicit mapping, no schema can be
determined and validation SHOULD fail. Mapping keys MUST be string values, but tooling MAY convert response values to
strings for comparison.

When used in conjunction with the `anyOf` construct, the use of the discriminator can avoid ambiguity where multiple
schemas may satisfy a single payload.

In both the `oneOf` and `anyOf` use cases, all possible schemas MUST be listed explicitly. To avoid redundancy, the
discriminator MAY be added to a parent schema definition, and all schemas comprising the parent schema in an `allOf`
construct may be used as an alternate schema.

For example:

```yaml
components:
    schemas:
        Pet:
            type: object
            required:
                - petType
            properties:
                petType:
                    type: string
            discriminator:
                propertyName: petType
                mapping:
                    dog: Dog
        Cat:
            allOf:
                -   $ref: '#/components/schemas/Pet'
                -   type: object
                    # all other properties specific to a `Cat`
                    properties:
                        name:
                            type: string
        Dog:
            allOf:
                -   $ref: '#/components/schemas/Pet'
                -   type: object
                    # all other properties specific to a `Dog`
                    properties:
                        bark:
                            type: string
        Lizard:
            allOf:
                -   $ref: '#/components/schemas/Pet'
                -   type: object
                    # all other properties specific to a `Lizard`
                    properties:
                        lovesRocks:
                            type: boolean
```

a payload like this:

```json
{
    "petType": "Cat",
    "name": "misty"
}
```

will indicate that the `Cat` schema be used. Likewise, this schema:

```json
{
    "petType": "dog",
    "bark": "soft"
}
```

will map to `Dog` because of the definition in the `mapping` element.

#### Security Scheme Object

Defines a security scheme that can be used by the operations.

Supported schemes are HTTP authentication, an API key (either as a header, a cookie parameter or as a query parameter),
mutual TLS (use of a client certificate), OAuth2's common flows (implicit, password, client credentials and
authorization code) as defined in [RFC6749](https://tools.ietf.org/html/rfc6749),
and [OpenID Connect Discovery](https://tools.ietf.org/html/draft-ietf-oauth-discovery-06).
Please note that as of 2020, the implicit flow is about to be deprecated
by [OAuth 2.0 Security Best Current Practice](https://tools.ietf.org/html/draft-ietf-oauth-security-topics). Recommended
for most use case is Authorization Code Grant flow with PKCE.

##### Fixed Fields

| Field Name       | Type                                    | Applies To          | Description                                                                                                                                                                                                                                                                                                                            |
|------------------|-----------------------------------------|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type             | `string`                                | Any                 | **REQUIRED**. The type of the security scheme. Valid values are `"ticket"`, `"wamp-cra"`, `"wamp-cryptosign"`.                                                                                                                                                                                                                         |
| description      | `string`                                | Any                 | A description for security scheme. [CommonMark syntax][CommonMark syntax] MAY be used for rich text representation.                                                                                                                                                                                                                    |

This object MAY be extended with [Specification Extensions](#specification-Extensions).

##### Security Scheme Object Example

*****FIXME*****: adopt Security Scheme Object Example

```json
{
    "type": "http",
    "scheme": "basic"
}
```

```yaml
type: http
scheme: basic
```

#### Security Requirement Object

Lists the required security schemes to execute this action.
The name used for each property MUST correspond to a security scheme declared in
the [Security Schemes](#Security-Scheme-Object) under the [Components Object](#components-Object).

When a list of Security Requirement Objects is defined on the [WampAPI Object](#WampAPI-Object)
or [WAMP URI RPC Action Object](#uri-rpc-action-object) or [WAMP URI Topic Action Object](#uri-topic-action-object),
only one of the Security Requirement Objects in the list needs to be satisfied to authorize the request.

##### Patterned Fields

| Field Pattern | Type     | Description                                                                                                                                                                                                                                                                                                 |
|---------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| {name}        | [string] | Each name MUST correspond to a security scheme which is declared in the [Security Schemes](#Security-Scheme-Object) under the [Components Object](#components-Object). The array MAY contain a list of role names which are required for the execution, but are not otherwise defined or exchanged in-band. |

##### Security Requirement Object Examples

*****FIXME*****: adopt Security Requirement Object Examples

```json
{
    "api_key": []
}
```

```yaml
api_key: [ ]
```

### Specification Extensions

While the WampAPI Specification tries to accommodate most use cases, additional data can be added to extend the
specification at certain points.

The extensions properties are implemented as patterned fields that are always prefixed by `"x-"`.

| Field Pattern | Type | Description                                                                                                                                                                                                                                                             |
|---------------|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ^x-           | Any  | Allows extensions to the WampAPI Schema. The field name MUST begin with `x-`, for example, `x-internal-id`. Field names beginning `x-WAS-` are reserved for specification internal usage and prototyping. The value can be `null`, a primitive, an array or an object.  |

The extensions may or may not be supported by the available tooling, but those may be extended as well to add requested
support (if tools are internal or open-sourced).

### Security Filtering

Some objects in the WampAPI Specification MAY be declared and remain empty, or be completely removed, even though they
are inherently the core of the API documentation.

The reasoning is to allow an additional layer of access control over the documentation. While not part of the
specification itself, certain libraries MAY choose to allow access to parts of the documentation based on some form of
authentication/authorization.

Three examples of this:

1. The [URIs Object](#uris-Object) MAY be present but empty. It may be counterintuitive, but this may tell the viewer
   that they got to the right place, but can't access any documentation. They would still have access to at least
   the [Info Object](#info-Object) which may contain additional information regarding authentication.
2. The [WAMP URI RPC Object](#uri-rpc-action-object) MAY be empty. In this case, the viewer will be aware that the uri exists,
   but will not be able to see any description or parameters. This is different from hiding the uri itself from
   the [URIs Object](#uris-Object), because the user will be aware of its existence. This allows the documentation
   provider to finely control what the viewer can see.
3. The [WAMP URI Topic Object](#uri-topic-action-object) MAY be empty. Just like an RPC described above.

[BCP-14]: https://tools.ietf.org/html/bcp14
[RFC2119]: https://tools.ietf.org/html/rfc2119
[RFC8174]: https://tools.ietf.org/html/rfc8174
[RFC3986]: https://tools.ietf.org/html/rfc3986
[RFC3986-sec3]: https://tools.ietf.org/html/rfc3986#section-3
[RFC3986-sec4.2]: https://tools.ietf.org/html/rfc3986#section-4.2
[RFC3986-sec5.1]: https://tools.ietf.org/html/rfc3986#section-5.1
[RFC3986-sec5.2]: https://tools.ietf.org/html/rfc3986#section-5.2
[RFC6901]: https://tools.ietf.org/html/rfc6901
[rfc5234]: https://tools.ietf.org/html/rfc5234
[RFC7159-sec7]: https://tools.ietf.org/html/rfc7159#section-7
[RFC7230-sec3.2]: https://tools.ietf.org/html/rfc7230#section-3.2.6
[Yaml-1.2]: https://yaml.org/spec/1.2/spec.html
[JSON Schema ruleset]: https://yaml.org/spec/1.2/spec.html#id2803231
[YAML Failsafe schema ruleset]: https://yaml.org/spec/1.2/spec.html#id2802346
[JSON Schema Spec 2022-06]: https://www.ietf.org/archive/id/draft-bhutton-json-schema-01.html#section-4.2.1
[JSON Schema Spec 2022-06 sec8.2]: https://www.ietf.org/archive/id/draft-bhutton-json-schema-01.html#section-8.2
[JSON Schema Validation vocabulary]: https://tools.ietf.org/html/draft-bhutton-json-schema-validation-00#section-7.3
[JSON Schema Validation]: https://tools.ietf.org/html/draft-bhutton-json-schema-validation-00
[JSON Schema Specification Draft 2020-12]: https://tools.ietf.org/html/draft-bhutton-json-schema-00
[JSON Schema dialect]: https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-4.3.3
[JSON Schema meta-schema]: https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-8
[JSON Schema rules]: https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-8.1.1
[CommonMark 0.27]: https://spec.commonmark.org/0.27/
[CommonMark syntax]: https://spec.commonmark.org/
[SPDX]: https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60
