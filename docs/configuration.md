# Network Configuration

`NetworkConfig` class allows to define basic network configuration like encoding or serializer. 

`Server` or `Client` instance can be created with custom configuration by passing `NetworkConfig` instance to their constructor.

To change default network configuration:

```csharp
NetworkConfig.Default = new NetworkConfig(...);
```

## Encoding

By default all text messages are encoded using `UTF8`.

## Serializer

By default `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` is used for serialization.

To use [Json.NET](https://www.newtonsoft.com/json) by default:
```csharp
NetworkConfig.Default = new NetworkConfig(new NewtonsoftJsonSerializer());
```

Example implementation of `NewtonsoftJsonSerializer` class:
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