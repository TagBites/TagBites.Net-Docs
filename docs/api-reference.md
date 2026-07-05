# API Reference

This page summarizes the public API surface as documented across this site and the published examples. It is **not** a substitute for generated XML-doc/IntelliSense reference — treat it as a map of what exists, and cross-check exact signatures (parameter names, overloads, nullability) against the installed package before relying on details not shown elsewhere in this documentation.

## `NetworkClient` (base class)

Abstract base class shared by `Client` and `ServerClient`. Members below are available on both.

| Member | Description |
|---|---|
| `bool IsConnected` | Whether the underlying connection is currently active. |
| `bool IsDisposed` | Whether the instance has been disposed. |
| `Task SendAsync(object message)` | Sends a serializable message to the other side. |
| `T GetController<T>()` | Returns a proxy for a controller registered on the remote side. |
| `void Close()` | Closes the connection, waiting for in-flight data to finish sending. |
| `void Dispose()` | Disposes the instance, closing the connection without waiting for in-flight data. |
| `Disconnected` (event) | See [Events](events.md). |
| `Received` (event) | See [Events](events.md). |
| `ReceivedError` (event) | See [Events](events.md). |
| `ControllerResolve` (event) | See [Events](events.md) and [RMI advanced](rmi-advanced.md). |

## `Client` (extends `NetworkClient`)

| Member | Description |
|---|---|
| `Client(string host, int port)` | Creates a client targeting the given host/port. |
| `Client(string host, int port, ClientCredentials credentials)` | Creates a client that will authenticate with the given credentials. See [Authentication](authentication.md). |
| `Client(string host, int port, NetworkConfig config)` | Creates a client with custom configuration. See [Configuration](configuration.md). |
| `Client(string host, int port, ClientCredentials credentials, NetworkConfig config)` | Combines credentials and custom configuration. |
| `Client(IPEndPoint address)` / `(address, credentials)` / `(address, config)` / `(address, credentials, config)` | Same as above, but targeting an `IPEndPoint` instead of separate host/port. |
| `EndPoint RemoteEndPoint` | The server's endpoint (updated to the resolved endpoint once connected). |
| `Task ConnectAsync()` | Connects to the server (and authenticates, if configured). Throws `ClientAuthenticationException` on auth failure, `NetworkConnectionOpenException` if the TCP connection itself can't be opened. Can be called again on the same instance after a `Disconnected` event to reconnect. |
| `Task ConnectSslAsync()` | Same as `ConnectAsync()`, but negotiates TLS over the connection. See [FAQ](faq.md#does-tagbitesnet-support-tlsssl). |
| `Task ConnectSslAsync(string serverName)` | Same as `ConnectSslAsync()`, additionally validating the server certificate against `serverName`. |
| `void Use<TControllerInterface, TController>()` | Registers an RMI controller implementation (instantiated via its parameterless constructor). |
| `void Use<TControllerInterface, TController>(TController controller)` | Registers an already-created RMI controller instance. |
| `protected virtual bool OnValidateServerCertificate(...)` | Override to customize SSL certificate validation (default: reject any certificate with policy errors). |
| `Connected` (event) | See [Events](events.md). |

## `Server` (implements `IDisposable`)

| Member | Description |
|---|---|
| `Server(string host, int port)` | Creates a server bound to the given host/port. |
| `Server(string host, int port, NetworkConfig config)` | Creates a server with custom configuration. |
| `Server(string host, int port, X509Certificate certificate)` | Creates a server that requires clients to connect over SSL. See [FAQ](faq.md#does-tagbitesnet-support-tlsssl). |
| `Server(string host, int port, X509Certificate certificate, NetworkConfig config)` | Combines SSL and custom configuration. |
| `Server(IPEndPoint address)` / `(address, config)` / `(address, certificate)` / `(address, certificate, config)` | Same as above, but targeting an `IPEndPoint` instead of separate host/port. |
| `bool Listening` | Set to `true` to start accepting connections on a background thread; set to `false` to stop. |
| `Task ListenAsync()` | Starts listening and returns a `Task` that completes when the listen loop ends (e.g. `Listening` set to `false` or a fatal accept error) — an awaitable alternative to `Listening = true`. |
| `IPEndPoint LocalEndpoint` | The address the server is actually bound to. |
| `bool DisconnectClientsOnDispose` | Whether connected clients are disposed when the server is disposed (default `true`). |
| `bool IsDisposed` | Whether the server has been disposed. |
| `Task SendToAllAsync(object message, ServerClient exceptClient = null)` | Broadcasts a message to all connected clients, optionally excluding one. Throws `AggregateException` if sending fails for one or more clients. |
| `ServerClient[] GetClients()` | Returns currently connected clients. |
| `ServerClient GetClient(object identity)` | Returns the connected client with the given identity, or `null`. |
| `void Use<TControllerInterface, TController>()` | Registers an RMI controller implementation shared by all clients. |
| `void Use<TControllerInterface, TController>(Func<ServerClient, TController> controllerProvider)` | Registers an RMI controller factory, invoked once per client connection (see [RMI advanced](rmi-advanced.md#lifecycle-of-controller-instances)). |
| `void Dispose()` | Stops listening and disconnects all clients (if `DisconnectClientsOnDispose`). |
| `ClientAuthenticate` (event) | See [Authentication](authentication.md). |
| `ClientConnectingError` (event) | See [Error handling](error-handling.md). |
| `ClientConnected` (event) | See [Events](events.md). |
| `ClientDisconnected` (event) | See [Events](events.md). |
| `Received` (event) | See [Events](events.md). |
| `ReceivedError` (event) | See [Events](events.md). |
| `ControllerResolve` (event) | See [Events](events.md) and [RMI advanced](rmi-advanced.md). |

## `ServerClient` (extends `NetworkClient`)

Represents a single connected client from the server's perspective (obtained from `Server.GetClients()`/`Server.GetClient(identity)` or from `e.Client` in server events).

| Member | Description |
|---|---|
| `object Identity` | The identity assigned during authentication (see [Authentication](authentication.md)). |
| `Server Server` | The `Server` that accepted this client; `null` once the server has been disposed. |
| `EndPoint RemoteEndPoint` | The client's remote endpoint. |
| `void Use<TControllerInterface, TController>()` / `(TController controller)` | Registers an RMI controller for this specific client connection. |

`GetController<T>()`, `SendAsync`, `Close`, `Dispose`, `IsConnected`, `IsDisposed`, and the `Disconnected`/`Received`/`ReceivedError`/`ControllerResolve` events are inherited from `NetworkClient` (see above).

## `NetworkConfig`

`Encoding` and `Serializer` are both **read-only** — set only via constructor, never via object initializer. See [Configuration](configuration.md#constructors) for the full constructor list.

| Member | Description |
|---|---|
| `static NetworkConfig Default` | The configuration used when none is passed explicitly to a `Client`/`Server` constructor. |
| `Encoding Encoding` | Encoding used for text/primitive payloads. Default: `UTF8`. |
| `INetworkSerializer Serializer` | Serializer used for object payloads. Default: built-in Json.NET-based serializer. |

## `ClientCredentials`

| Member | Description |
|---|---|
| `UserName` | Basic auth username. |
| `Password` | Basic auth password. |
| `Token` | Alternative to username/password. |

`[Serializable]`. Can be subclassed to pass additional/custom authentication data. See [Authentication](authentication.md).

## `INetworkSerializer`

```csharp
public interface INetworkSerializer
{
    void Serialize(Stream stream, object value);
    object Deserialize(Stream stream, Type type);
}
```

See [Writing a custom serializer](custom-serializer.md).

## Exceptions

| Type | Thrown when |
|---|---|
| `ClientAuthenticationException` | `ConnectAsync()`/`ConnectSslAsync()` fails authentication (server did not set `e.Authenticated = true`, or the auth handshake itself errored). Derives from `System.Security.Authentication.AuthenticationException`. |
| `NetworkConnectionOpenException` | The client failed to open the underlying TCP/SSL connection (before authentication is attempted). |
| `NetworkConnectionBreakException` | The connection was unexpectedly closed while reading/writing (e.g. remote side disconnected mid-message). |
| `NetworkObjectProtocolViolationException` | An unexpected error occurred while reading/writing the message framing itself (not a serialization error). Derives from `System.Net.ProtocolViolationException`. |
| `NetworkSerializationException` | `INetworkSerializer.Serialize`/`Deserialize` threw while processing a specific message. Exposes `TypeName` and `SerializationException` (the original error). Derives from `System.Net.ProtocolViolationException`. |
| `NetworkSerializationTypeNotFoundException` | A message referenced a type name that couldn't be resolved via `Type.GetType(...)` on the receiving side (e.g. type doesn't exist/isn't referenced there). Derives from `NetworkSerializationException`. |
| `NetworkControllerInvocationException` | An RMI call failed — see [RMI advanced](rmi-advanced.md#exception-propagation) for the full breakdown via its `Type`/`RemoteMessage`/`RemoteException` properties. |
