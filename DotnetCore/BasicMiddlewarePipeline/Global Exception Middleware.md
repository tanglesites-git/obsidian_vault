```csharp
public class GlobalExceptionHandlerMiddleware : IMiddleware  
{  
    /// <summary>  
    /// This function intercepts the request and handles any exception that occurs. Converting them into a JSON response of type <see cref="CustomProblemDetails"/>    /// </summary>    /// <param name="context">The HttpContext for the Endpoint</param>    /// <param name="next">A delegate to call the next middleware in the pipeline</param>    public async Task InvokeAsync(HttpContext context, RequestDelegate next)  
    {  
        try  
        {  
            await next(context);  
        }  
        catch (Exception ex)  
        {  
            var type = ex.GetType();  
  
            var trace = ParseStackTrace(ex).ToList();  
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;  
            context.Response.ContentType = "application/json";  
            var details = CustomProblemDetails.From(context, type, ex, trace);  
            var result = JsonSerializer.Serialize(details);  
            await context.Response.WriteAsync(result);  
        }  
    }  
  
    /// <summary>  
    /// It parses the stack trace of an <see cref="Exception"/> and returns a <see cref="IEnumerable{T}"/> of <see cref="StackTraceItem"/>    /// </summary>    /// <param name="ex">The current <see cref="Exception"/> being intercepted</param>    /// <returns>An <see cref="IEnumerable{T}"/> where T is of type <see cref="StackTraceItem"/></returns>    private static IEnumerable<StackTraceItem> ParseStackTrace(Exception ex)  
    {  
        var list = ex.StackTrace?.Split('\n').ToList().Select(x =>  
        {  
            x = x.Trim();  
            var lines = x.Split(" in ").ToList().Select(x =>  
            {  
                x = x.Trim();  
  
                return x;  
            }).ToList();  
  
            if (lines.Count == 1)  
            {  
                return new StackTraceItem  
                {  
                    Method = lines.FirstOrDefault()  
                };  
            }  
            else  
            {  
                var location = lines.FirstOrDefault();  
                var file = lines.LastOrDefault()?.Split(":line").Select(x => x.Trim()).ToList();  
  
                return new StackTraceItem  
                {  
                    Location = location,  
                    File = file?.FirstOrDefault(),  
                    Line = file?.LastOrDefault(),  
                };  
            }  
        });  
  
        return list!;  
    }  
}
```

```csharp
/// <summary>  
/// A custom extension of the ProblemDetails class that adds additional properties to the response.  
/// </summary>  
internal sealed class CustomProblemDetails : ProblemDetails  
{  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(-6)]  
    public string? TraceId { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(0)]  
    public string? Source { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(1)]  
    public string? HResult { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(2)]  
    public List<StackTraceItem>? StackTrace { get; set; }  
  
    public static CustomProblemDetails From(HttpContext context, Type type, Exception ex, List<StackTraceItem> trace)  
    {  
        return new CustomProblemDetails  
        {  
            TraceId = Activity.Current?.Id ?? context.TraceIdentifier,  
            Type = type.FullName,  
            Title = "An error occurred",  
            Status = StatusCodes.Status500InternalServerError,  
            Detail = ex.Message,  
            Instance = context.Request.Path,  
            Source = ex.Source,  
            StackTrace = trace  
        };  
    }  
  
}
```

```csharp
/// <summary>  
/// A custom class that represents a single item in the stack trace.  
/// </summary>  
internal class StackTraceItem  
{  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(-6)]  
    public string? Location { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(-5)]  
    public string? File { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(-4)]  
    public string? Line { get; set; }  
  
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]  
    [JsonPropertyOrder(-3)]  
    public string? Method { get; set; }  
  
  
}
```