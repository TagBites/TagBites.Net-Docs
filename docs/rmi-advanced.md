# RMI (Remote Method Invocation) — Advanced

For a basic introduction see the [RMI example in Quick start](README.md#chat-example-using-rmi-remote-method-invocation).

## Registering and resolving controllers

```csharp
// Register ahead of time
server.Use<IChatServer, ChatServer>(client => new ChatServer { Client = client, Server = server });

// Or resolve lazily on first use
server.ControllerResolve += (s, e) =>
{
    if (e.ControllerType == typeof(IChatServer))
        e.Controller = new ChatServer { Client = e.Client, Server = server };
};
```

`ControllerResolve` fires the first time a remote method call arrives targeting a controller type that hasn't been registered with `Use<TInterface, TImplementation>()`. This lets you create controller instances on demand (e.g. with dependency injection) instead of registering every controller type upfront.

The event args type differs depending on where you subscribe:

- On `Client`/`ServerClient` (which both derive from the internal `NetworkClient` base), the event is `EventHandler<NetworkConnectionControllerResolveEventArgs>` with properties:
  - `string ControllerTypeName` — assembly-qualified-style name of the requested controller interface.
  - `Type ControllerType` — the resolved `Type`, or `null` if it couldn't be loaded via `Type.GetType(...)`.
  - `object Controller` — set this to the instance you want to use.
- On `Server`, the event is `EventHandler<ServerClientControllerResolveEventArgs>`, which additionally exposes `ServerClient Client` (the specific client the call came from) via its base `ServerClientEventArgs`.

There is no `Handled`/`Cancel` flag — the first subscriber (in the order events were added) that sets a non-null `Controller` wins, and the rest are skipped.

## Supported method signatures

Based on the documented examples, controller methods can:

- Take primitive types or serializable objects as parameters.
- Return `void`, `Task`, or any primitive/serializable type (including `Task<T>`).

```csharp
public interface IMyController
{
    void FireAndForget(string message);
    Task<int> ComputeAsync(int a, int b);
}
```

Overloads are supported — the remote call is matched by method name **and** exact parameter type list. The following are explicitly **not** supported and should be avoided:

- **Generic methods** (methods with their own type parameters, e.g. `T Echo<T>(T value)`). Using closed generic types as ordinary parameters/return values (e.g. `List<string>`) works fine — it's only open type parameters on the method itself that aren't supported.
- **`ref`/`out` parameters** — arguments are captured and sent once when the call is made; any value the remote implementation writes to a `ref`/`out` parameter is not sent back and will not be visible to the caller.
- **`CancellationToken` parameters** — a token has no meaningful representation once serialized to the remote side; cancelling a token locally will not cancel the corresponding remote call.

## Exception propagation

Exceptions thrown inside a controller method on the remote side propagate back to the caller, but **always wrapped** in `NetworkControllerInvocationException` — the original exception type is never rethrown as-is:

```csharp
public class ChatServer : IChatServer
{
    public void Send(string message)
    {
        if (string.IsNullOrEmpty(message))
            throw new ArgumentException("Message cannot be empty.");
        // ...
    }
}
```

```csharp
try
{
    client.GetController<IChatServer>().Send(message);
}
catch (NetworkControllerInvocationException ex)
{
    // ex.Message is a generic "Remote controller execution exception occurred."
    Console.WriteLine(ex.Type);           // NetworkControllerInvocationExceptionType.MethodInvokeException
    Console.WriteLine(ex.RemoteMessage);  // the original exception's Message ("Message cannot be empty.")
    Console.WriteLine(ex.RemoteException);// the original exception's full ToString() (including stack trace)
}
```

`NetworkControllerInvocationException.Type` (`NetworkControllerInvocationExceptionType`) tells you *why* the call failed:

| Value | Meaning |
|---|---|
| `MethodInvokeException` | The registered controller method threw — `RemoteMessage`/`RemoteException` describe the original exception. |
| `ControllerNotFound` | No controller was registered/resolved for the requested interface on the remote side. |
| `MethodNotFound` | The remote controller exists, but no method matched the requested name + parameter types. |
| `DataReceivingError` | The response could not be read/deserialized (e.g. serializer mismatch between the two sides). |
| `OperationCancelled` | The connection was disposed while the call was still waiting for a response. |

## Lifecycle of controller instances

The controller factory delegate passed to `Use<TInterface, TImplementation>(factory)` is invoked **once**, the first time a remote call targets that controller type on a given connection. The resulting instance is then cached for the lifetime of that connection — subsequent calls reuse the same instance rather than recreating it. This is safe to rely on if your controller holds per-connection state (as `ChatServer` does with its `Client`/`Server` properties in the example above).
