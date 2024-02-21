---
tags:
 - options_pattern
 - dotnet_core
---


### Configuration Value

```json
// secrets.json
{
	"MyOptions": {
		"Name": "John Smithy",
		"Age": 30
	}
}
```

### Register Your Options POCO

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

    [Required]
    public string? Name { get; set; }

    [Range(18, 130)]
    public int Age { get; set; }
}


[OptionsValidator]
internal partial class MyOptionsValidator : IValidateOptions<MyOptions>
{

}
```

> Here we had to create a partial class implementing the `IIValidateOptions` interface and add the `OptionsValidator` Decorator. 

#### Further Reference
[Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/core/extensions/options)

![Nick Chapsas YouTube Video](https://www.youtube.com/watch?v=mO0fwvnnzbU)
