## Options Pattern
> When you want to bind a POCO to a configuration value in one of the common configuration files: appsettings.json, appsettings.Development.json or UserSecrets. 

> First we start with a configuration value: 

```json
// secrets.json
{
	"MyOptions": {
		"Name": "John Smithy",
		"Age": 30
	}
}
```

> Next we will need a POCO to associate with the configuration value. Here we will use the `MyOptions` class.  `MyOptions` must be a non-abstract class with a default public constructor; that means the constructor cannot have any parameters.  If you include parameters in your constructor, the program will not complain, but your values from the configuration value will be overridden. First create a class:

```csharp
// MyOptions.cs
internal sealed class MyOptions
{
    public const string SectionName = "MyOptions";
    public string? Name { get; set; }
    public int Age { get; set; }
}
```

> Now in order to use these values throughout our codebase, we will need register MyOptions inside the IConfiguration Container. We do this by hydrating the properties of the MyOptions class with the configuration value in `secrets.json` 

```csharp
// Program.cs
var root = builder.Environment.ContentRootPath;

var options = builder.Configuration.GetSection(MyOptions.SectionName)
			.Get<MyOptions>();

				... // Rest of the program.cs 

```

> Now we can use the `MyOptions` configuration POCO in our API. 

```csharp
// Program.cs
app.MapGet("/options_pattern", (IOptionsSnapshot<MyOptions> options) =>
{
    return options.Value.Name;
})
.WithName("OptionsPattern")
.WithOpenApi();
```

> Here I used `IOptionsSnapshot` but you can use any of the IOptions variants.

> Where `options` is of type `<MyOptions>` . Now this works for the enclosing scope. However, if I was to try to access the `MyOptions` object from another class, an error will occur.  Perhaps we could try something like the following: [[Options Pattern (Global)]]

> Why does this not work? Well if we look into the `Get` and `GetSection` methods, they only reference the `IConfigurationRoot` and not the `IServiceCollection` .  But in the next section [[Options Pattern (Global)]] we will see that in order to have global access, we must interact with both the `IServiceCollection` and `ICollectionRoot` . 


#### Further Reference
[Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/core/extensions/options)

