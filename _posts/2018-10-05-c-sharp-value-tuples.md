---
layout: post
title: C# value tuples and destructuring
---

### Named tuples have full-fledged support in C#  
methods returning value tuples:
```c#
//verbose
//IEnumerable<ValueTuple<double, double>> coordinates()

//idiomatic
IEnumerable<(double x, double y)> coordinates()
{
  //verbose
  //yield return ValueTuple.Create(1.1, 2.1);
  //yield return (x: 1.1, y: 2.1);
  
  //idiomatic
  yield return (1.1, 2.1);
  yield return (1.2, 2.2);
}

(double x, double y) coordinate() => coordinates().First();
```
---------------------------------------
### Consuming returned value tuples  
instantiating variables:
```c#
var point = coordinate();  //point.x and point.y
//or
var (x, y) = coordinate();

foreach (var point in coordinates()) { /* ... */ }
//or
foreach (var (x, y) in coordinates()) { /* ... */ }
```
and although the value names are given, you can destructure them into context specific names  
```c#
foreach (var (enemyX, enemyY) in enemy.coordinates()) { /* ... */ }
```
---------------------------------------
modifying variables:  
```c#
// given a function like
(double x, double y) applyPhysics((double x, double y) point, params object[] otherObjects)
{
  return (point.x + 1, point.y + 1); //high tech physics formulas go here
}

// destructuring can be used to update xPos and yPos like this
var (xPos, yPos) = coordinate();
(xPos, yPos) = applyPhysics((xPos, yPos), allEnemies);
```
