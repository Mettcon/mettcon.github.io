---
id: suyvdgeyipdebxr61uoo1xd
title: csharp
desc: ''
updated: 1662229120041
created: 1661764973049
tags: #using #dispose #garbage collector
---


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
    process?.Dispose()
}
```
