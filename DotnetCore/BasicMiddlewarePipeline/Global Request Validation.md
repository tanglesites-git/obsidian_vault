```csharp
internal class MinimalApiValidationFilter<T> : IEndpointFilter  
{  
      
    /// <summary>  
    /// A middleware that intercepts the request and validates the request using the <see cref="IValidator{T}"/> service.    /// </summary>    /// <param name="context">The <see cref="EndpointFilterInvocationContext"/> that gives you access to the context of the Request Delegate, such as the Argument List.</param>    /// <param name="next">A delegate to call the next middleware in the pipeline</param>    /// <returns>A <see cref="ValueTask"/> of a nullable type of <see cref="object"/></returns>    public async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext context, EndpointFilterDelegate next)  
    {  
        var type = typeof(T);  
        foreach (var item in context.Arguments)  
        {  
            if (typeof(T).ToString() == item?.GetType().ToString())  
            {  
                var validator = context.HttpContext.RequestServices.GetService<IValidator<T>>();  
  
                if (validator is not null)  
                {  
                    var result = await validator.ValidateAsync((T)item);  
                    if (!result.IsValid)  
                    {  
                        return Results.ValidationProblem(result.ToDictionary(), statusCode: (int)HttpStatusCode.BadRequest);  
                    }  
                }  
            }  
        }  
        return await next.Invoke(context);  
    }  
}
```