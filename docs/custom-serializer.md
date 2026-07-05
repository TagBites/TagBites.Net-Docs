# Writing a Custom Serializer

`NetworkConfig` accepts any implementation of `INetworkSerializer`. This page shows a full custom implementation, complementing the built-in Json.NET one documented in [Configuration](configuration.md).

## The interface

```csharp
public interface INetworkSerializer
{
    void Serialize(Stream stream, object value);
    object Deserialize(Stream stream, Type type);
}
```

## Example: System.Text.Json serializer

```csharp
using System.Text.Json;

public class SystemTextJsonSerializer : INetworkSerializer
{
    private readonly JsonSerializerOptions _options;

    public SystemTextJsonSerializer()
    {
        _options = new JsonSerializerOptions
        {
            // Mirrors the intent of TypeNameHandling.Auto in the built-in
            // Json.NET serializer — only enable polymorphic/dynamic type
            // resolution if you fully trust the remote side (see the
            // security note below).
        };
    }

    public void Serialize(Stream stream, object value)
    {
        JsonSerializer.Serialize(stream, value, value.GetType(), _options);
    }

    public object Deserialize(Stream stream, Type type)
    {
        return JsonSerializer.Deserialize(stream, type, _options);
    }
}
```

## Registering it

`NetworkConfig.Encoding` and `NetworkConfig.Serializer` are both **read-only** — they can only be set through a constructor, not via an object initializer. Pass the serializer directly:

```csharp
NetworkConfig.Default = new NetworkConfig(new SystemTextJsonSerializer());

var client = new Client("127.0.0.1", 82); // uses NetworkConfig.Default
// or, per-instance:
var client2 = new Client("127.0.0.1", 82, new NetworkConfig(new SystemTextJsonSerializer()));
```

If you also want a non-default `Encoding`, use the `NetworkConfig(Encoding, INetworkSerializer)` overload instead. See [Configuration](configuration.md#constructors) for the full list of `NetworkConfig` constructors.

## ⚠️ Security note

Both the built-in serializer example and the pattern above involve deserializing types received over the network. If you enable polymorphic/dynamic type resolution (e.g. `TypeNameHandling.Auto` in Json.NET, or a custom `$type`-like mechanism), a malicious or compromised peer could potentially instruct your process to instantiate arbitrary types. Mitigate by:

- Only enabling this on connections where both ends are trusted and authenticated (see [Authentication](authentication.md)).
- Restricting deserialization to an allow-list of known types where possible.
- Keeping the transport itself authenticated/encrypted (see the open item in [Architecture](architecture.md#wire-protocol) about TLS support).
