---
id: suyvdgeyipdebxr61uoo1xd
title: csharp
desc: ''
updated: 1661768750969
created: 1661764973049
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
