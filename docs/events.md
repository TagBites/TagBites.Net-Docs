# Events

`Client` class events:
- `Connected` - Occurs after connection is established to the server and client is authenticated.
- `Disconnected` - Occurs when client disconnects form the server.
- `Received` - Occurs when server sends a message.
- `ReceivedError` - Occurs when client was unable to receive server message (eg. deserialization error).
- `ControllerResolve` - Occurs when server requests access to controller for the first time. It allows to create and return controller instance without register it first.

`Server` class events:
- `ClientAuthenticate` - Occurs right after tcp connection has been established, but before <see cref="ClientConnected"/> event. It allows to authenticate client and assign identity.
- `ClientConnectingError` - Occurs when an exception is thrown while accepting tcp client, or during authentication procedure, or during <see cref="ClientConnected"/> event.
- `ClientConnected` - Occurs after the connection is established and client is successfully authenticated..
- `ClientDisconnected` - Occurs when client disconnects form the server.
- `Received` - Occurs when client sends a message.
- `ReceivedError` - Occurs when server was unable to receive client message (eg. deserialization error).
- `ControllerResolve` - Occurs when client requests access to controller for the first time. It allows to create and return controller instance without register it first.

`ServerClient` class events:
- `Disconnected` - Occurs when client disconnects form the server.
- `Received` - Occurs when server sends a message.
- `ReceivedError` - Occurs when client was unable to receive server message (eg. deserialization error).
- `ControllerResolve` - Occurs when server requests access to controller for the first time. It allows to create and return controller instance without register it first.
