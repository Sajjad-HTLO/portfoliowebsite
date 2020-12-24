---
title: "Hibernate Annotations"
date: 2020-09-06T17:14:35+06:00
description: "Hibernate Annotations Orphan Removal VS ON DELETE CASCADE"
author: "Sajad Hayatlou"
type: "post"
---

There is two different concepts in defining relation attributes:

The `orphanRemoval` is an entirely ORM-specific thing. It marks `child` entity (original record) to be removed when it’s no longer referenced from the `parent` entity, e.g. when you remove the child entity from the corresponding collection of the parent entity.

But in other side, the `ON DELETE CASCADE` is a database-specific thing, it deletes the `child` row in the database when the `parent` row is deleted.


See here:

http://www.objectdb.com/java/jpa/persistence/delete#Orphan_Removal_



### Summary

The `orphanRemoval` strategy simplifies the child entity state management, as we only have to remove the child entity from the child collection, and the associated child record is deleted as well.

Unlike the `orphanRemoval` strategy, the `CascadeType.REMOVE` propagates the remove operation from the parent to the child entities, as if we manually called remove on each child entity.






