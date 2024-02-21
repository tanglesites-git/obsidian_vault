```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped
<IValidator<WeatherForecastRequest>, WeatherForecastValidator>();  
builder.Services.AddTransient<GlobalExceptionHandlerMiddleware>();

			// ... Rest of Configuration

app.UseMiddleware<GlobalExceptionHandlerMiddleware>();
```

```csharp
app.MapGet("/weatherforecast", WeatherEndpoint.Handle)  
.WithName(WeatherEndpoint.SectionName)  
.WithOpenApi()  
.AddEndpointFilter<MinimalApiValidationFilter<WeatherForecastRequest>>();
```

```csharp
using FluentValidation;  
  
using LearningDotnetCore.ResultsTypeDemo;  
  
var builder = WebApplication.CreateBuilder(args);  
  
builder.Services.AddScoped<IValidator<WeatherForecastRequest>, WeatherForecastValidator>();  
builder.Services.AddTransient<GlobalExceptionHandlerMiddleware>();  
  
  
builder.Services.AddEndpointsApiExplorer();  
builder.Services.AddSwaggerGen();  
  
  
var app = builder.Build();  
  
if (app.Environment.IsDevelopment())  
{  
    app.UseSwagger();  
    app.UseSwaggerUI();  
}  
  
app.MapGet("/weatherforecast", WeatherEndpoint.Handle)  
.WithName(WeatherEndpoint.SectionName)  
.WithOpenApi()  
.AddEndpointFilter<MinimalApiValidationFilter<WeatherForecastRequest>>();  
  
app.UseMiddleware<GlobalExceptionHandlerMiddleware>();  
  
app.Run();  
internal record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)  
{  
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);  
}
```