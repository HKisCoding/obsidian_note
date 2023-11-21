# Notation:
Distributed Application Runtime.

Dapr is a portable, event-driven, runtime for building distributed applications across the cloud and edge.

Support to build Microservices application 

# Support:
DAPR built on concept of building blocks -> expose HTTP/gRPC API 

[**Service-to-service invocation**](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/service-invocation-overview/)

`/v1.0/invoke`

Service invocation enables applications to communicate with each other through well-known endpoints in the form of http or gRPC messages. Dapr provides an endpoint that acts as a combination of a reverse proxy with built-in service discovery, while leveraging built-in distributed tracing and error handling.

[**State management**](https://docs.dapr.io/developing-applications/building-blocks/state-management/state-management-overview/)

`/v1.0/state`

Application state is anything an application wants to preserve beyond a single session. Dapr provides a key/value-based state and query APIs with pluggable state stores for persistence.

[**Publish and subscribe**](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)

`/v1.0/publish` `/v1.0/subscribe`

Pub/Sub is a loosely coupled messaging pattern where senders (or publishers) publish messages to a topic, to which subscribers subscribe. Dapr supports the pub/sub pattern between applications.

[**Bindings**](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/)

`/v1.0/bindings`

A binding provides a bi-directional connection to an external cloud/on-premise service or system. Dapr allows you to invoke the external service through the Dapr binding API, and it allows your application to be triggered by events sent by the connected service.

[**Actors**](https://docs.dapr.io/developing-applications/building-blocks/actors/actors-overview/)

`/v1.0/actors`

An actor is an isolated, independent unit of compute and state with single-threaded execution. Dapr provides an actor implementation based on the virtual actor pattern which provides a single-threaded programming model and where actors are garbage collected when not in use.

[**Secrets**](https://docs.dapr.io/developing-applications/building-blocks/secrets/secrets-overview/)

`/v1.0/secrets`

Dapr provides a secrets building block API and integrates with secret stores such as public cloud stores, local stores and Kubernetes to store the secrets. Services can call the secrets API to retrieve secrets, for example to get a connection string to a database.

[**Configuration**](https://docs.dapr.io/developing-applications/building-blocks/configuration/configuration-api-overview/)

`/v1.0/configuration`

The Configuration API enables you to retrieve and subscribe to application configuration items for supported configuration stores. This enables an application to retrieve specific configuration information, for example, at start up or when configuration changes are made in the store.

[**Distributed lock**](https://docs.dapr.io/developing-applications/building-blocks/distributed-lock/distributed-lock-api-overview/)

`/v1.0-alpha1/lock`

The distributed lock API enables you to take a lock on a resource so that multiple instances of an application can access the resource without conflicts and provide consistency guarantees.

[**Workflows**](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-overview/)

`/v1.0-alpha1/workflow`

The Workflow API enables you to define long running, persistent processes or data flows that span multiple microservices using Dapr workflows or workflow components. The Workflow API can be combined with other Dapr API building blocks. For example, a workflow can call another service with service invocation or retrieve secrets, providing flexibility and portability.

[**Cryptography**](https://docs.dapr.io/developing-applications/building-blocks/cryptography/cryptography-overview/)

`/v1.0-alpha1/crypto`

The Cryptography API enables you to perform cryptographic operations, such as encrypting and decrypting messages, without exposing keys to your application.