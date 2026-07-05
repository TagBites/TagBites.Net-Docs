# FAQ / Troubleshooting

## "Address already in use" / server won't start listening

Another process is already bound to the chosen port, or a previous `Server` instance on the same port wasn't properly disposed. Pick a different port or ensure the previous instance is stopped before starting a new one.

## Client can't connect from another machine

- Check that the server is bound to an address reachable from the client (e.g. `0.0.0.0` rather than `127.0.0.1` if you want connections from other machines).
- Check firewall rules on the server machine for the chosen TCP port.
- If running in a container/cloud environment, make sure the port is published/exposed.

## `ReceivedError` fires instead of `Received`

Usually caused by a deserialization mismatch — the two sides don't agree on the message type, or one side changed its `INetworkSerializer`/message contract without updating the other. Make sure both client and server:

- Use compatible serializers (see [Configuration](configuration.md)).
- Reference the same message/DTO types (or contract-compatible versions).

## `ClientAuthenticationException` on connect

The server's `ClientAuthenticate` handler did not set `e.Authenticated = true` for the given credentials. See [Authentication](authentication.md) — check your `MyValidateMethod` logic and that credentials are actually being sent by the client.

## Is it safe to enable dynamic/polymorphic deserialization (`TypeNameHandling.Auto`)?

Only between trusted, authenticated peers. See the security note in [Writing a custom serializer](custom-serializer.md).

## Does TagBites.Net support TLS/SSL?

Yes, natively — no need to wrap the socket stream yourself:

```csharp
// Server: pass a certificate to enable SSL
var server = new Server("127.0.0.1", 82, myX509Certificate);

// Client: connect over SSL
var client = new Client("127.0.0.1", 82);
await client.ConnectSslAsync();          // or ConnectSslAsync("server-name") for cert name validation
```

Internally this wraps the socket in an `SslStream` and negotiates TLS 1.2 (and TLS 1.3 on .NET 7+). By default, the client rejects any certificate that has policy errors (e.g. self-signed, untrusted root). To accept such certificates (e.g. in development), subclass `Client` and override the `protected virtual bool OnValidateServerCertificate(...)` method.

## Can I send binary data (files, byte arrays)?

Yes — `byte[]` is a special-cased message type: it's sent as raw bytes without going through the configured `INetworkSerializer` at all, so it's cheap compared to serializing an equivalent wrapper object.

There's no hard message-size limit enforced by the library beyond what fits in a 32-bit length prefix, but there's also no streaming/chunked-transfer API: the entire message is serialized/buffered in memory on both ends before being sent/after being received. For large files, chunk the data yourself (e.g. send fixed-size `byte[]` pieces) rather than sending one very large message.
