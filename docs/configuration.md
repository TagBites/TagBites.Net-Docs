# Network Configuration

`NetworkConfig` class allows to define basic network configuration like encoding or serializer. Both `Encoding` and `Serializer` are **read-only** properties — they can only be set via a constructor, not through an object initializer.

`Server` or `Client` instance can be created with custom configuration by passing `NetworkConfig` instance to their constructor.

To change default network configuration:

```csharp
NetworkConfig.Default = new NetworkConfig(...);
```

## Constructors

| Constructor | Notes |
|---|---|
| `NetworkConfig()` | `Encoding = UTF8`, `Serializer = ` built-in Json.NET serializer. |
| `NetworkConfig(Encoding encoding)` | Custom encoding, default serializer. |
| `NetworkConfig(INetworkSerializer serializer)` | Default encoding (`UTF8`), custom serializer. |
| `NetworkConfig(Encoding encoding, INetworkSerializer serializer)` | Custom encoding and serializer. |
| `NetworkConfig(Action<Stream, object> serializeDelegate, Func<Stream, Type, object> deserializeDelegate)` | Default encoding, serializer defined inline as delegates (no need to implement `INetworkSerializer`). |
| `NetworkConfig(Encoding encoding, Action<Stream, object> serializeDelegate, Func<Stream, Type, object> deserializeDelegate)` | Custom encoding, serializer defined inline as delegates. |

All arguments are validated as non-null (constructors throw `ArgumentNullException` otherwise).

## Encoding

By default all text messages are encoded using `UTF8`.

## Serializer

By default [Json.NET](https://www.newtonsoft.com/json) is used for serialization, with following implementation:

```csharp
public class NewtonsoftJsonSerializer : INetworkSerializer
{
    private readonly JsonSerializer _serializer;

    public NewtonsoftJsonSerializer()
    {
        var settings = new JsonSerializerSettings 
        { 
            // Required for internal dynamic type handling  
            TypeNameHandling = TypeNameHandling.Auto
        };
        _serializer = JsonSerializer.CreateDefault(settings);
    }


    public void Serialize(Stream stream, object value)
    {
        using var writer = new StreamWriter(stream);
        using var jsonWriter = new JsonTextWriter(writer);

        _serializer.Serialize(jsonWriter, value);
    }
    public object Deserialize(Stream stream, Type type)
    {
        using var reader = new StreamReader(stream);
        using var jsonReader = new JsonTextReader(reader);

        var value = _serializer.Deserialize(jsonReader, type);
        return value;
    }
}
```
