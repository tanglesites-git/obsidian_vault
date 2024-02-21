```csharp
internal static class WeatherEndpoint  
{  
    public const string SectionName = "GetWeatherForecast";  
    private static readonly string[] summaries = new[]  
    {  
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"  
    };  
  
    /// <summary>  
    /// This method returns a list of weather forecasts randomly generated based on the `id` parameter.    /// </summary>    /// <param name="id">This value determines the number of forecasts to be returned.</param>    /// <returns>This returns a <see cref="WeatherForecast"/></returns>    /// <exception cref="ValidationException">Throws if the request is invalid</exception>    /// <exception cref="ArgumentOutOfRangeException">Throws if the request is valid but equal to zero.</exception>    public static async Task<IResult> Handle(IValidator<WeatherForecastRequest> _validator, [AsParameters] WeatherForecastRequest request)  
    {  
        var result = await _validator.ValidateAsync(request);  
  
        if (!result.IsValid) return Results.ValidationProblem(result.ToDictionary());  
  
        var forecast = Enumerable.Range(0, request.Id).Select(index =>  
           new WeatherForecast  
           (  
               DateOnly.FromDateTime(DateTime.Now.AddDays(index)),  
               Random.Shared.Next(-20, 55),  
               summaries[Random.Shared.Next(summaries.Length)]  
           ))  
           .ToArray();  
  
        if (forecast.Length == 0) throw new ArgumentOutOfRangeException("The forecast array is empty.");  
  
        return TypedResults.Ok(forecast);  
    }  
}
```

```csharp
internal class WeatherForecastRequest  
{  
    public int Id { get; set; }  
}
```

```csharp
internal class WeatherForecastValidator : AbstractValidator<WeatherForecastRequest>  
{  
    public WeatherForecastValidator()  
    {  
        RuleFor(x => x.Id).GreaterThan(-1);  
    }  
}
```

[[Global Request Validation]]
[[Global Exception Middleware]]