# Error Handling & Reconnecting

## Relevant events

| Side | Event | When it fires |
|---|---|---|
| `Client` | `ReceivedError` | The client could not process an incoming message (e.g. deserialization error). |
| `Client` | `Disconnected` | The connection to the server was lost. |
| `Server` | `ClientConnectingError` | An exception was thrown while accepting a TCP connection, during authentication, or inside a `ClientConnected` handler. |
| `Server` | `ReceivedError` | The server could not process a message sent by a client. |
| `Server` / `ServerClient` | `ClientDisconnected` / `Disconnected` | A client disconnected (gracefully or due to an error). |

See [Events](events.md) for the full list.

## Handling receive errors

```csharp
client.ReceivedError += (s, e) =>
{
    // Inspect e.Exception (or equivalent) to distinguish transport errors
    // from serialization/deserialization errors.
    Console.WriteLine($"Failed to receive message: {e.Exception}");
};
```

On `Client` this is `NetworkConnectionMessageErrorEventArgs` with a single `Exception` property. On `Server`/`ServerClient` it's `ServerClientMessageErrorEventArgs`, which adds `Exception` alongside the inherited `Client` (the `ServerClient` that sent the offending message). The snippet above is accurate as written.

## Handling disconnects and reconnecting

The library does not appear to provide built-in automatic reconnection — this should be implemented by the consumer:

```csharp
async Task ConnectWithRetryAsync(Client client, int maxAttempts = 5)
{
    var delay = TimeSpan.FromSeconds(1);

    for (var attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            await client.ConnectAsync();
            return;
        }
        catch (Exception ex) when (attempt < maxAttempts)
        {
            Console.WriteLine($"Connect attempt {attempt} failed: {ex.Message}");
            await Task.Delay(delay);
            delay += delay; // simple backoff
        }
    }
}

client.Disconnected += async (s, e) =>
{
    Console.WriteLine("Disconnected, attempting to reconnect...");
    await ConnectWithRetryAsync(client);
};
```

The same `Client` instance can be reused: after `Disconnected` fires, `IsConnected` becomes `false` again, so calling `ConnectAsync()` (or `ConnectSslAsync()`) on the same instance opens a fresh connection. You do not need to create a new `Client` object to reconnect, so the retry helper above is safe to call repeatedly on the same `client` reference.

## Authentication errors

If authentication fails on the server (`ClientAuthenticate` sets `e.Authenticated = false`, or is never handled), the client throws `ClientAuthenticationException` from `ConnectAsync()`. Wrap the call in a `try/catch`:

```csharp
try
{
    await client.ConnectAsync();
}
catch (ClientAuthenticationException)
{
    Console.WriteLine("Authentication failed — check credentials.");
}
```

See [Authentication](authentication.md) for the full authentication flow.
