# WampAPI-Specification

The WampAPI Specification (WAS) defines a standard and programming language-agnostic interface description for WAMP-based
APIs, which allows both humans and computers to discover and understand the capabilities of a service without requiring
access to source code, additional documentation, or inspection of network traffic. When properly defined via WampAPI, a
consumer can understand and interact with the remote service with minimal implementation logic. Similar to what
interface descriptions do for lower-level programming, the WampAPI Specification removes the guesswork in calling a
service.

Use cases for machine-readable API definition documents include, but are not limited to: interactive documentation; code
generation for documentation, clients, and servers; and automation of test cases. WampAPI documents describe APIs
services and are represented in YAML or JSON formats. These documents may be produced and served statically or generated
dynamically from an application.

The WampAPI Specification does not require rewriting existing APIs. It does not require binding any software to a
service – the described service may not even be owned by the creator of its description. It does, however, require the
service's capabilities be described in the structure of the WampAPI Specification. The WampAPI Specification does not
mandate a specific development process such as design-first or code-first. It does facilitate either technique by
establishing clear interactions with a WAMP API.

This GitHub project is the starting point for WampAPI. Here you will find the information you need about the WampAPI
Specification, simple examples of what it looks like, and some general information regarding the project.

The specification itself can be found in [spec/specification.md]()

## See It in Action

If you just want to see it work, check out the [list of current examples](examples).

## Tools and Libraries

Looking to see how you can create your own WampAPI definition, present it, or otherwise use it? Check out the growing
[list of implementations](IMPLEMENTATIONS.md).

## Participation

The current process for developing the WampAPI Specification is described in 
[Development Guidelines](DEVELOPMENT.md).

## Licensing

See: [MIT License](LICENSE)
