From the last section [[Options Pattern (Local)]] we had the following data.

```json
// secrets.json
{
	"MyOptions": {
		"Name": "John Smithy",
		"Age": 30
	}
}
```

and the following class

```csharp
// MyOptions.cs
internal sealed class MyOptions
{
    public const string SectionName = "MyOptions";
    public string? Name { get; set; }
    public int Age { get; set; }
}
```

To be able to access the configuration value from any class in our codebase, we need to take a different approach. There are two ways to do this: One is with Reflection and the other is with Source Generation. 

#### Reflection Method

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOptions<MyOptions>()
    .Bind(builder.Configuration.GetSection(MyOptions.SectionName));

				... // Rest of the program.cs 
```

> Then the example from before will work.

```csharp
// Program.cs
app.MapGet("/options_pattern", (IOptionsSnapshot<MyOptions> options) =>
{
    return options.Value.Name;
})
.WithName("OptionsPattern")
.WithOpenApi();
```
#### Source Generation
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<MyOptions>(builder.Configuration.GetSection(MyOptions.SectionName));

builder.Services.AddSingleton<IValidateOptions<MyOptions>, MyOptionsValidator>();

				... // Rest of the program.cs 
```

```csharp
internal sealed class MyOptions
{
    public const string SectionName = "MyOptions";
    
    public string? Name { get; set; }

    public int Age { get; set; }
}
```

> Here we had to create a partial class implementing the `IIValidateOptions` interface and add the `OptionsValidator` Decorator.  Validating your options in the next section: [[Validate Options]]

#### Further Reference
[Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/core/extensions/options)
