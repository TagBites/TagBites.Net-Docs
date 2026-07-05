# Architecture

## Overview

TagBites.Net is a TCP client-server library built around three main types:

- **`Client`** — connects to a single `Server` and exchanges messages with it.
- **`Server`** — listens for incoming TCP connections and communicates with any number of connected clients.
- **`ServerClient`** — the server-side representation of a single connected client (returned by `Server.GetClients()` and passed in server events, e.g. `e.Client`).

Messages can be plain serializable objects (`SendAsync`/`Received`) or, when using RMI, method calls made through generated proxy interfaces (see [RMI advanced](rmi-advanced.md)).

## Threading model

- Setting `server.Listening = true` starts a background thread that accepts incoming connections; it does not block the calling thread.
- Each connected client is handled independently, so `Received`, `ClientConnected`, and `ClientDisconnected` handlers can be invoked concurrently for different clients. Handlers should be written to be thread-safe if they touch shared state.
- The public site describes the library as thread-safe by design, but if your handlers mutate shared collections (e.g. a list of clients or a chat history), you are still responsible for synchronizing access to *your own* state.
- All event handlers (`Received`, `ReceivedError`, `ClientConnected`, `ClientDisconnected`, etc.) are invoked directly on the background thread-pool thread (`Task.Run`) that is reading from the socket — there is **no** marshaling back to a captured `SynchronizationContext`. If you update UI (WPF/WinForms) from these handlers, you must dispatch back to the UI thread yourself (e.g. `Dispatcher.Invoke`/`Control.Invoke`).

## Message flow (plain messages)

1. `Client.ConnectAsync()` opens a TCP connection and (if configured) performs authentication — see [Authentication](authentication.md).
2. `Client.SendAsync(message)` serializes the message using the configured `INetworkSerializer` (see [Configuration](configuration.md)) and writes it to the socket.
3. On the receiving side, the `Received` event fires with the deserialized message.
4. If deserialization or transport fails, `ReceivedError` fires instead of `Received`.

## Message flow (RMI)

1. Both sides register controllers with `Use<TInterface, TImplementation>()`.
2. When one side calls `GetController<TInterface>()`, it receives a dynamic proxy implementing `TInterface`.
3. Calling a method on the proxy serializes the method name and arguments, sends them to the remote side, invokes the matching method on the registered implementation, and (for non-`void` methods) sends the return value back.
4. If no controller was registered ahead of time, `ControllerResolve` fires, letting you create and return an instance on demand.

See [RMI advanced](rmi-advanced.md) for exception propagation, async methods, and supported parameter/return types.

## Wire protocol

Each message is framed as a small binary header followed by content:

1. `messageId` (int32) and `inResponseToId` (int32) — used internally to correlate RMI calls/responses; both `0` for plain `SendAsync`/`Received` messages.
2. A one-byte `TypeCode` (from `System.TypeCode`) identifying the payload kind (`Empty`/`DBNull` for `null`, `String`, `DateTime`, other primitives, or `Object`).
3. For non-null payloads: the `Encoding` code page (int32), then the content itself:
   - `string`/primitives/`DateTime` are encoded directly with the configured `Encoding` (no serializer involved).
   - `byte[]` is sent as raw bytes — also bypassing the configured `INetworkSerializer` entirely.
   - Any other object: the fully-qualified type name (length-prefixed) followed by the bytes produced by `NetworkConfig.Serializer.Serialize(...)`.
4. A 4-byte content length prefix, then the content bytes.

There is no application-level maximum message size beyond the `int32` length prefix, and no built-in chunking/streaming — an entire message is buffered in memory on both ends.

TLS/SSL **is** supported natively: `Client.ConnectSslAsync()`/`ConnectSslAsync(serverName)` on the client side, and passing an `X509Certificate` to a `Server` constructor on the server side. Internally this wraps the socket in an `SslStream` negotiating TLS 1.2 (plus TLS 1.3 on .NET 7+). See [FAQ: Does TagBites.Net support TLS/SSL?](faq.md#does-tagbitesnet-support-tlsssl) for a usage example.
