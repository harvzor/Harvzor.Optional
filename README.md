﻿# Harvzor.Optional

## The problem

```csharp
// The JSON would normally come from some external data source:
string json = "{}";

Foo foo = JsonSerializer.Deserialize<Foo>(json);

// This will print an empty string because C# will hydrate the model with the default value.
// There's no way to check if the model used a default value.
Console.WriteLine(foo.MyProperty); // ""    

public class Foo
{
    public string MyProperty { get; set; }
}
```

In the above example, you could make the string nullable (with `string?`), but now you're explicitly saying that null is value you want to accept.
And then you might actually want to explicitly allow null values, but then run into the same issue where you're not sure if the original JSON had
the value as `null`, or if it was just undefined.

## Optional<T>

### Basic example

... write docs ...

### Use it with JSON

This example uses System.Text.Json, but it's pretty similar to Newtonsoft:

```csharp
JsonSerializerOptions jsonSerializerOptions = new JsonSerializerOptions();
jsonSerializerOptions.Converters.Add(new Harvzor.Optional.SystemTextJson.OptionalJsonConverter());

// The JSON would normally come from some external data source:
string json = "{\"DefinedProperty\":\"Bar\"}";

Foo foo = JsonSerializer.Deserialize<Foo>(json, jsonSerializerOptions);

Console.WriteLine(foo.DefinedProperty.IsDefined); // True
Console.WriteLine(foo.UndefinedProperty.IsDefined); // False

// Now I can check if a value was defined before I try using it:
if (foo.DefinedProperty.IsDefined)
    Console.WriteLine(foo.DefinedProperty.Value); // "Bar" 
    
if (foo.UndefinedProperty.IsDefined)
    Console.WriteLine(foo.UndefinedProperty.Value); // Won't print as the value wasn't defined.

public class Foo
{
    public Optional<string> DefinedProperty { get; set; }
    public Optional<string> UndefinedProperty { get; set; }
}
```

## Harvzor.Optional.SystemTextJson

### Use in an API

To use it in your controller models, simply register in your startup:

```csharp
services
    .AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.Converters.Add(new Harvzor.Optional.SystemTextJson.OptionalJsonConverter());
    });
```

#### Swagger

If you're using Swashbuckle Swagger, you'll also need to tell it how your types should look:

```csharp
services.AddSwaggerGen(options =>
{ 
    options.MapType<Optional<string>>(() => new OpenApiSchema
    {
        Type = "string"
    });
    
    options.MapType<Optional<bool>>(() => new OpenApiSchema
    {
        Type = "boolean"
    });
});
```

You can see what basic types are available here: https://swagger.io/docs/specification/data-models/data-types/
