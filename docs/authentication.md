## Authentication

### Client site
```csharp
var credentials = new ClientCredentials()
{
    UserName = "user",
    Password = "password",
    // Token = "..."
};
var client = new Client("127.0.0.1", 82, credentials);
```

A `ClientCredentials` object allows passing a basic authentication info, but derived class instance is also supported.

### Server site
```csharp
var server = new Server("127.0.0.1", 82);
server.ClientAuthenticate += (s, e) =>
{
    if (MyValidateMethod(e.Credentials))
    {
        e.Identity = e.Credentials.UserName;
        e.Authenticated = true;
    }
};

bool MyValidateMethod(ClientCredentials credentials) { ... }
```

To enable authentication, subscribe to `ClientAuthenticate` event. To authenticate the user, the `Authenticated` property must be set to `true`, otherwise `ClientAuthenticationException` exception will be throw on client's site.

### Identity fallback

If the handler sets `Authenticated = true` but does not explicitly set `Identity`, the server falls back to `Credentials.UserName`. If the client connects **without** passing a `ClientCredentials` instance at all, `Credentials` will be `null`, and relying on this fallback will throw a `NullReferenceException` inside the `ClientAuthenticate` handler. Either always set `Identity` explicitly, or make sure clients always pass a `ClientCredentials` object when authentication is enabled.

### SSL/TLS

Authentication runs over whatever transport the connection was opened with. To also encrypt the connection itself, create the client with `ConnectSslAsync()` and the server with a certificate — see [Does TagBites.Net support TLS/SSL?](faq.md#does-tagbitesnet-support-tlsssl) in the FAQ.
