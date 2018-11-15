# Events

`Client` class events:
- `Connected` - Occurs after connection is established to the server and the client is authenticated.
- `Disconnected` - Occurs when the client disconnects from the server.
- `Received` - Occurs when the server sends a message.
- `ReceivedError` - Occurs when the client was unable to receive server message (eg. deserialization error).
- `ControllerResolve` - Occurs when the server requests access to the controller for the first time. It allows creating and returning an instance of controller without register it first.

`Server` class events:
- `ClientAuthenticate` - Occurs right after the tcp connection has been established, but before the `ClientConnected` event. It allows authenticating client and assigning the identity.
- `ClientConnectingError` - Occurs when an exception is thrown while accepting the tcp client or during the authentication procedure or during the `ClientConnected` event.
- `ClientConnected` - Occurs after the connection is established and the client is successfully authenticated.
- `ClientDisconnected` - Occurs when the client disconnects from the server.
- `Received` - Occurs when the client sends a message.
- `ReceivedError` - Occurs when the server was unable to receive client message (eg. deserialization error).
- `ControllerResolve` - Occurs when client requests access to the controller for the the first time. It allows creating and returning an instance of controller without register it first.

`ServerClient` class events:
- `Disconnected` - Occurs when the client disconnects from the server.
- `Received` - Occurs when the server sends a message.
- `ReceivedError` - Occurs when the client was unable to receive server message (eg. deserialization error).
- `ControllerResolve` - Occurs when the server requests access to the controller for the first time. It allows creating and returning an instance of controller without register it first.
