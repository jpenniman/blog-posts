---
layout: post
title: Identity Performance in Microsoft SQL Server
date: 2025-09-07
image: /blog/images/id-perf.png
author: Jason M Penniman
excerpt: Out of curiosity, I wanted to see what offered the best performance when inserting records: 
  database generated or application generated ids.
tags:
- dotnet
- C#
- SQL Server
---
A twitter post about using hi-lo in Entity Framework Core got me thinking about performance impact to inserts when
choosing different ways of generating IDs. I was surprised at the results.

The performance falls into 4 quadrants:

![Scenarios](/blog/images/id-perf-scenarios.png)

Test setup:
- BenchmarkDotNet v0.15.2, Linux Ubuntu 24.04.3 LTS (Noble Numbat)
- 13th Gen Intel Core i7-1370P 5.20GHz, 1 CPU, 20 logical and 14 physical cores
- .NET SDK 9.0.109
- Runtime: .NET 9.0.8 (9.0.825.36511), X64 RyuJIT
- Concurrent Server GC
- SQL Server 2019 for Linux running in docker on the test machine.
- Query Store enabled in SQL Server with wait stats turned on.
  ``` sql
  ALTER DATABASE IdentityPerformance SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE, WAIT_STATS_CAPTURE_MODE = ON)
  ```
- IterationCount=10

Commonly, an identity column is used to autogenerate sequential integer ids, so this was my baseline.

``` sql
id bigint not null identity(1,1) primary key`
```

Since I'm using Entity Framework Core, we'll let EF take care of it.

``` cs
entity.Property(e => e.Id).UseIdentityColumn();
```

For hi-lo, the application generates the id. Well, sort of. The implementation in Entity Framework Core uses a
sequence with an increment value of the hi-lo range. so there is a database component involved in generating ids.
However, the application assigns the id to the record and inserts the record with the id. For this, we just need an id
column in the database.

``` sql
id bigint not null primary key`
```

And of course, the sequence:

``` sql
create sequence dbo.hilo_test_seq
    start with 1
    increment by 100
go
```

In EF, we have:

``` cs
modelBuilder.HasSequence<long>("hilo_test_seq").StartsAt(1).IncrementsBy(100);
 entity.Property(e => e.Id).UseHiLo("hilo_test_seq");
```

For the database generated UUIDs, we'll use a default constraint to call the SQL Server functions:

``` cs
entity.Property(e => e.Id).ValueGeneratedOnAdd().HasDefaultValueSql("newid()");
...
entity.Property(e => e.Id).ValueGeneratedOnAdd().HasDefaultValueSql("newsequentialid()");
```

For the application set UUIDs, we'll just let EF know IDs will never be generated, though this is more to express intent:

``` cs
entity.Property(e => e.Id).ValueGeneratedNever();
```

Setting the id's was was just a simple:

``` cs
Id = Guid.NewGuid()
...
Id = Ulid.NewUlid().ToGuid()
```

Yes, we could create a custom value generator that sets these in the application so devs don't have to explicitly set
the value when creating an instance. For simplicity, I just assign it on construction for the tests.

I tried to simulate concurrent inserts into the same table by multiple client connections by utilizing a `Parallel.For`:

``` cs
    const int itemsCount = 20;
    const int parallelism = 10;

    [Benchmark]
    public void InsertIdentityTest()
    {
        Parallel.For(1, parallelism, t =>
        {
            var context = Setup();
            for (int i = 0; i < itemsCount; i++)
            {
                context.IdentityTests.Add(new IdentityTest
                {
                    Name = $"Test {i}",
                    Email = $"test{i}@example.com"
                });
            }
            
            context.SaveChanges();
        });
    }
```

This gives me 10 concurrent db connections/clients each inserting 20 records into the table.

## The Results

| Method                                           | Mean      | Min       | Max       | P95       | Ratio | RatioSD | Gen0    | Gen1   | Allocated | Alloc Ratio |
|------------------------------------------------- |----------:|----------:|----------:|----------:|------:|--------:|--------:|-------:|----------:|------------:|
| Identity Column, database generated.             | 23.316 ms | 17.362 ms | 29.047 ms | 28.045 ms |  1.03 |    0.23 | 31.2500 |      - |   2.03 MB |        1.00 |
| Hi-Lo, application generated using DB Sequence.  | 16.543 ms |  9.004 ms | 22.589 ms | 21.683 ms |  0.73 |    0.21 | 46.8750 |      - |   2.16 MB |        1.07 |
| **Guid (UUIDv4), application generated.**        |  4.653 ms |  2.798 ms |  5.450 ms |  5.446 ms |  0.20 |    0.05 | 39.0625 |      - |   1.99 MB |        0.98 |
| **ULID, application generated.**                 |  5.017 ms |  3.260 ms |  6.899 ms |  6.551 ms |  0.22 |    0.06 | 39.0625 |      - |   1.99 MB |        0.98 |
| **Guid (UUIDv4), database generated (NEWID()).** |  4.801 ms |  4.502 ms |  5.237 ms |  5.143 ms |  0.21 |    0.04 | 42.9688 | 3.9063 |   2.06 MB |        1.02 |
| ULID, database generated (NEWSEQUENTIALID()).    | 29.130 ms | 27.700 ms | 29.865 ms | 29.864 ms |  1.28 |    0.21 | 46.8750 |      - |   2.06 MB |        1.02 |

I suspect this has to do with the how SQL Server deals with identity columns and unique values internally. Both identity
and NEWSEQUENTIALID() need to perform checks to ensure the generated value is unique. This results in very high wait
times, specifically on BUFFER LATCH. Total wait time for each scenario:

| Scenario                                        | total wait time ms |
| :---------------------------------------------- | -----------------: |
| Identity Column, database generated.            |             129833 |
| SELECT NEXT VALUE FOR \[hilo\_test\_seq\]       |                 81 |
| Hi-Lo, application generated using DB Sequence. |              67169 |
| Guid (UUIDv4), application generated.           |               1765 |
| ULID, application generated.                    |               1039 |
| Guid (UUIDv4), database generated (NEWID()).    |               1908 |
| ULID, database generated (NEWSEQUENTIALID()).   |             157061 |

## Conclusion

For insert-heavy applications, consider UUID. If read performance is also critical, application generated ULIDs can
provide a happy medium. As with anything else performance related, test the options in your own product environment.
Every system has different data and access patterns. Just because a raw insert with no business context is faster in 
this test, that doesn't mean it is the optimal solution for you application.
