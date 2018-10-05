---
layout: post
title: Debugging externally run tests
---

I was working in the fork of NEventStore again today.
I could only run the persistence unit tests via the psake scripts, externally, yet I was having trouble with a newly added test and wanted to be able to debug it.
quick-n-dirty solution: wait for debugger to attach directly in the test
```c#
while(!Debugger.IsAttached) Thread.Sleep(100);
```
of course remember to remove it before committing (!)

---------------------------------------------------------

Before running the debugger I had added a ```Console.Writeline``` statement to output the sql statement I needed to see.
Strangely enough, when it output to the console, it looked strangely chopped up
```sql
WITH [cte] AS
   ( SELECT BucketId, StreamId, StreamIdOriginal, StreamRevision, CommitId, CommitSequence, CommitStamp, CheckpointNumber, Headers, Payload
) AS [row] FROM CommitsORDER BY CheckpointNumber
 WHERE BucketId = @BucketId
   AND CheckpointNumber > @CheckpointNumber
   AND (EventType in ('DomainEventA')
             OR (AdditionalEventTypes is not null AND
                             (AdditionalEventTypes like '%;DomainEventA;%')
                   )
         )
  )

SELECT *
  FROM [cte]
 WHERE [row] BETWEEN @Skip + 1
                 AND @Limit + @Skip;
```

fortunately debugging showed the expected string

```sql
WITH [cte] AS
   ( SELECT BucketId, StreamId, StreamIdOriginal, StreamRevision, CommitId, CommitSequence, CommitStamp, CheckpointNumber, Headers, Payload
  , ROW_NUMBER() OVER (ORDER BY CheckpointNumber 
) AS [row] FROM Commits
 WHERE BucketId = @BucketId 
   AND CheckpointNumber > @CheckpointNumber
   AND (EventType in ('DomainEventA') 
	     OR (AdditionalEventTypes is not null AND 
			     (AdditionalEventTypes like '%;DomainEventA;%')
		   ) 
	 )
  )

SELECT *
  FROM [cte]
 WHERE [row] BETWEEN @Skip + 1
                 AND @Limit + @Skip;
```

I've never seen this behavior before, and I still don't know why it happened...
