# 6 Working with Disconnected Graphs of Objects

## New Use Case: Registering a Student

Problem Adding Objects With Child Objects

Entity States:
*   Detached
*   Unchanged
*   Deleted
*   Modified
*   Added


**Existing Child Entity which is not Attached is automatic signed as Added,
So it must be Attached !** 


## Update and Attach Methods in DbSet

First Attach then Update !

Attach (Prefer)
* new => Added
* Existing => Unchanged

Update (Avoid)
* new => Added
* Existing => Modified

## Assigning a Disconnected Entity to a Connected One

Problem of combining a attached and detached entity
The detached Entity will automatically marked as modified

```C#
public override int SaveChanges()
    {
        foreach (EntityEntry<Course> course in ChangeTracker.Entries<Course>())
        {
            course.State = EntityState.Unchanged;
        }

        return base.SaveChanges();
    }
```
     