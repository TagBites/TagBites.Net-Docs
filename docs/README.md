# TagBites.Net

Lightweight and simple TCP client-server .NET library with RMI support.

NuGet Package: https://www.nuget.org/packages/TagBites.Net/

## Chat example

### Client code.
```csharp
var client = new Client("127.0.0.1", 82);
client.Received += (s, e) => Console.WriteLine(e.Message.ToString());
await client.ConnectAsync();

while (true)
    await client.SendAsync(Console.ReadLine());
```

### Server code.
```csharp
var server = new Server("127.0.0.1", 82);
server.Received += (s, e) => server.SendToAllAsync($"{e.Client}: {e.Message}", e.Client);
server.ClientConnected += (s, e) => server.SendToAllAsync($"{e.Client} connected", e.Client);
server.ClientDisconnected += (s, e) => server.SendToAllAsync($"{e.Client} disconnected", e.Client);
server.Listening = true; // starts new thread

Console.ReadLine(); // for console application to prevent app from closing
```

In this example a `string` type is used for communication, but any serializable objects can be send/received. By default `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` is used for serialization, but it can be replaced with custom implementation.

Full example on github: [TagBites.Net-Sample-Chat](https://github.com/TagBites/TagBites.Net-Sample-Chat).

## Chat example using RMI (Remote Method Invocation)
    
### Client code.
```csharp
var client = new Client("127.0.0.1", 82);
client.Use<IChatClient, ChatClient>();
await client.ConnectAsync();

while (true)
{
    var message = Console.ReadLine();
    client.GetController<IChatServer>().Send(message);
}
```

Method `Use<TControllerInterface, TController>()` registers controller that can be used by server. Server site can use `IChatClient` interface to execute methods implemented by `ChatClient` on client site. Client/Server can registers many controllers. Controller instance will be created on first usage.

`GetController<T>()` returns proxy interface to class register on remote site. Calling method on this instance will invoke method on remote site. Controller can invoke methods with primitive or serializable parameters types and returns void/Task or any primitive or serializable type.

### Server code.
```csharp
var server = new Server("127.0.0.1", 82);
server.Use<IChatServer, ChatServer>(client => new ChatServer(client, server));
server.ClientConnected += (s, e) =>
{
    var controller = client.GetController<IChatClient>();

    foreach (var client in server.GetClients())
        controller.MessageReceive(null, $"Client {e.Client} connected.");

    Console.WriteLine($"Client {e.Client.Identity} connected.");
};
server.ClientDisconnected += (s, e) =>
{
    var controller = client.GetController<IChatClient>();

    foreach (var client in server.GetClients())
        controller.MessageReceive(null, $"Client {e.Client} disconnected.");

    Console.WriteLine($"Client {e.Client} disconnected.");
};
server.Listening = true;

Console.ReadLine();
```

### Classes.
```csharp
public interface IChatClient
{
    void MessageReceive(string userName, string message);
}
public interface IChatServer
{
    void Send(string message);
}
public class ChatClient : IChatClient { ... }
public class ChatServer : IChatServer { ... }
```

Full example on github: [TagBites.Net-Sample-ChatWithControllers](https://github.com/TagBites/TagBites.Net-Sample-ChatWithControllers).