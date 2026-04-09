# Notes
## C#
### using Statement

recently I stumbled upon a code snippet like this

```csharp
SPWeb rollbackProcess;
try {
    using (SPWeb process = someSite.Openweb() ) {
        // prepare Rollback
        rollbackProcess = process;
        
        //... some code here
    }
} catch {
    // rollback
}
```

SPWebs are some pretty heavy Sharepoint Objects and have to be manually disposed. This is mentioned in the [Class Doc examples](https://learn.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ms473942(v=office.14)#examples) and explained in the [Best Practices](https://learn.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ee557362(v=office.14)#why-dispose).  
That's what the [using statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using) is for. At it's end it calls `dispose()` for you.  

I assume, initially the developer wrote something like this
```csharp
    static void Main()
    {
        try
        {
            using (TrackedResource process = new(1))
            {
                process.Use();
            }
        }
        catch
        {
            //rollback here
            process.Use(); // IDE error here
        }
    }
```
and his IDE told him `CS0103  The name 'process' does not exist in the current context`
So he prepared a variable in another context for the rollback and ended up with the code from the beginning.

3 things are important to understand here.
1. To understand here what the using statement exactly does.  
We can see this in [Sharplab](https://sharplab.io/#v2:D4AQTAjAsAULIGYAE4kBECmBbA9gYQBsBDAZxKQC4BJNASxIAccSiAjAjJWAb1iX8QoALOnpMSGABQBKJNwC+sRTEGo8cvvxTIQIgLIyNMLSZQAGJJPTZ8xMkgBCRAHYuiSALxJnGAO4zZBU1+ZXkgA=), the [docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using) and the [language specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1314-the-using-statement).  
It is nothing more than a try finally which calls the `Dispose()` function from finally.
2. To know that `Dispose()` is just a convention, an agreed upon name for a function which cleans up its unmanaged resources. Resources the GC does not know about. It does not remove the object itself, or trigger the GC to collect it.
3. Notice how the message says `The name 'process' does not exist`. This means exactly what it says. The name pointing to that heap object is gone. The object itself is still alive. Including the `rollBackProcess` reference.

Usually the collector decides on its own when it runs. For Demo purposes we call it explicitly here.
```csharp
    static void Main()
    {
        TrackedResource rollbackProcess = null;
        try
        {
            using (TrackedResource process = new(1))
            {
                rollbackProcess = process;
                // assuming something  failed 
                throw new NotImplementedException();
            }
        }
        catch
        {
            GC.Collect();
            rollbackProcess.Use();
        }
    }
```
returns
```
[1] Created
[1] Disposed (unmanaged resources freed)
[1] Used
```

It still works. Why? What is happening?  
First of all - `using(){}` is already it's own `try{} finally{}` block. We can see this in [Sharplab](https://sharplab.io/#v2:D4AQTAjAsAUOAEIIHZYG9by4iA2RALPALICGAlgHYAUAlJthjNi/ACoBOpAxgNYCmAEwBK/AM4B7AK4du/eBwkAbJQCMevAAqK5YsfAC88SlJUBuBq3gAXDgE9LrJlasgADPGqcNQ0ZJly8AAOOuL6RpT8AO7UELT0zC4szkkuiirqfNoSuuHBoXoWiamsAPSl8I4lWNYAFopRxtHwAHIS1gCSALZBSvxd/JTWQgCiAB5yQdbkEjS0RdXwAL5V2CvFrNyk1ty1q1gpJelqGtm5AHQAqmL8dAsu6yzr67AI3ny+4tKy8gBc8B0ACLkMRBCRiUiqProRwgADMCn4pEEsyUdngVGsAME9yw5RC5AAbtt5KoJMp4AB9QQgsE3QSGeAAM1IShuRVhCPeAhEXwCt0xGMECWS+2xjPIOLFSAAnNQACQAIgA2mgOoIlgBdeAAYQ4SOGgkV80cLw28MI8GutxFjEc5RY5CZnmptPBQlteIqLjqDSajQA8qoAFb8bjWYGg92CcaTaazBWK7mffw/VXqrXG3GsWWJ9Ma7XWo0mjZmlgWkBESN0m2OQ7wB3YV1R+mM2xSfjZ8sQOVK/Na+DV6OeKSULqkSikADmQkRqd0zP1HqzYoA4jrzgBlKRBEJhABiVFZ5AAXrc6iD5g2KsMVPp1/9KBImrPrM+tipmUelKf+BxTbAsBmrAQA===), the [docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using) and the [language specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1314-the-using-statement)  

But this still does not explain why `process` is still accessible via `rollbackProcess`. Even if the garbage collector ran.  
The Garbage Collector only collects objects which are considered "dead". And in General objects are dead when there is no reachable code or references anymore.


In Sharepoint at this point in time. `SPWeb` should be a pretty light object again without any unmanaged resources.
However, from here on, this can lead to multiple ways and all of them are bad and I'm too lazy to test what actually happens
- you use that object in Rollback - it creates some unmanaged resources, heavy or often enough to starve your memory and the server crashes.
- you use that object in Rollback and it throws an exception or it is undefined behaviour.
- you seem to have a working rollback, but it only reinitialized some internal state and you end with a partial or corrupt data, leaked handles which accumulate without crash or exception.

In this case the Developer solved the problem with a change to `rollbackProcessURL = process.URL`.  
Which copies the string references to Identify that object and, if needed, creates a new one in rollback.

But since we know `using()` is just `try{} finally{}` in a trenchcoat. We can just use the one we already have

```csharp
SPWeb process;
try 
{
    process = someSite.Openweb();
} 
catch
{ 
    /* Rollback here */
}
finally
{
    process?.Dispose();
}
```


