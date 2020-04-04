# 3 Introducing a Sample Project

## Problem Domain Introduction

Classic CRUD

## Application Code Introduction

```C#
private readonly IList<Enrollment> _enrollments = new List<Enrollment>();
public virtual IReadOnlyList<Enrollment> Enrollments => _enrollments.ToList();
```
## Drawbacks

CRUD Based Thinking


