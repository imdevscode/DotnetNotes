# DotnetNotes
Q. Difference between action filter and middleware in web api
Ans: They both are used to process the request but are executed at different point in request pipeline. 
Action Filter
- They are designed to run before (OnActionExecuting) & after (OnActionExecuted) the action method of controller.
- they are useful for task that are specific to individual action or controller, like Validation, Caching a particular action, Authorization etc.
- We can apply Action filters directly to actions or controllers using attributes.
Ex:-
```csharp
public class MyActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(HttpActionContext actionContext)
    {
        // Logic before action executes
    }

    public override void OnActionExecuted(HttpActionExecutedContext actionExecutedContext)
    {
        // Logic after action executes
    }
}
```
This will render as a nicely formatted C# code block with syntax highlighting on GitHub.
This will render as a nicely formatted C# code block with syntax highlighting on GitHub.
