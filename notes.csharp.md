---
id: 5bf46f8xwbcv2oz7jj4ld89
title: Csharp
desc: ''
updated: 1775522476088
created: 1772941871103
---

#TODO: Add memory profiler screenshots  
## using statement
recently I stumbled upon a code snippet like this

```csharp
SPWeb rollbackProcess
try {
    using (SPWeb process = someSite.Openweb() ) {
        // prepare Rollback
        rollbackProcess = process
        
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
2. To know that `Dispose()` is just a convention, an agreed upon name for a function which cleans up it's unmanaged ressources. Ressources the GC does not know about. It does not remove the object itself, or trigger the GC to collect it.
3. Notice how the message says `The name 'process' does not exist`. This means exactly what it says. The name pointing to that heap object is gone. The object itself is still alive. And it is still alive. Including an




We know now that `process` doesn't exist in the catch block or rollback.
But `rollbackProcess` holds a reference to that not existing variable. So what happens?

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

But this still does not explain why `process` is still accessible via `rollbackProcess`. The answers lies in the Garbage Collector.









## using statement
recently I stumbled upon a code snippet like this

```csharp
var rollbackProcess
try {
    using (SPWeb process = someSite.Openweb() ) {
        // prepare Rollback
        rollbackProcess = process
        
        //... some code here
    }
} catch {
    // rollback
}
```
`The using declaration calls the Dispose method on the object in the correct way when it goes out of scope. The using statement causes the object itself to go out of scope as soon as Dispose is called.`

If you dispose an object manually you should(must?) remove the reference with assigning a new value to "unchain" it and make it free for the GC to collect.
Mostly this looks like this
```csharp
process.dispose()
process = null
```
but `using` is only aware of `process`, which means the object is still referenced in `rollbackProcess` and therefore, will never be disposed.

### Solution
How do we solve the problem?  
In this case, the developer just turned `rollbackProcess = process` into `rollbackProcessURL = process.URL` which is a `string` Type and it did the Trick.  
I'm not sure `rollbackProcessURL` is still a reference to the `URL` Property of `process` and I don't know how to check that and if that works in other cases.  

For better solutions it may help us to understand what's going on.
Sharplab shows us that `using`s are nothing more than a `try{} finally{}` 
```csharp
public class C {
    public void M() {
        using (Person Demo = new()) {
            Console.WriteLine(Demo.Name);
        }
    }
}
```
get's lowered to
```csharp
public class C
{
    public void M()
    {
        Person person = new Person();
        try
        {
            Console.WriteLine(person.Name);
        }
        finally
        {
            if (person != null)
            {
                ((IDisposable)person).Dispose();
            }
        }
    }
}
```
which means we can use the example above to this
```csharp
SPWeb process = null
try {   
    someSite.Openweb()
        //... some code here
} catch {
    // rollback
} finally {
    // this is the new way to do
    // if (process){ process.Dispose }
    process?.Dispose()
}
```