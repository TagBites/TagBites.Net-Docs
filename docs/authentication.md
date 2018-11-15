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
* {
    if (MyValidateMethod(e.Credentials))
    {
        e.Identity = e.UserName;
        e.Authenticated = true;
    }
};

bool MyValidateMethod(ClientCredentials credentials) { ... }
```

To enable authentication, subscribe to `ClientAuthenticate` event. To authenticate the user, the `Authenticated` property must be set to `true`, otherwise `ClientAuthenticationException` exception will be throw on client's site.
