# Memoization - Cache any function to improve performance
This repository is intended for managing issues / feature requests / questions for [Unity Memoization - Cache any function to improve performance](https://u3d.as/38ti) package.

## Overview
Package implements [Memoization](https://en.wikipedia.org/wiki/Memoization) technique into any Unity project (built-in, URP, HDRP, custom render pipeline). Package allows you to cache any function's result and return a cached result instead of calling same function again next time it is invoked.

Code is super lightweight (<10KB), performance orientated and [memory friendly](##%20Memory%20management) with no memory leaks.

All code has no dependencies and would run on any Unity's project, in fact, it would run on any C# project. Memoization can be applied to any `object`, not just Unity's `MonoBehaviour`.

## Installation
Follow Unity's standard [Package Manager installation instructions](https://docs.unity3d.com/Manual/upm-ui-install.html).
Link to a package on Unity Asset Store: https://u3d.as/38ti

## Basic Usage
Say you have a standard `MonoBehaviour`
```
using UnityEngine;
using System;

public class Cube : MonoBehaviour
{
    void Start()
    {
        MyCustomFunction(100);
        MyCustomFunction(100);
        MyCustomFunction(200); 
        MyCustomFunction(200);
        MyCustomFunction(100);
    }
    int MyCustomFunction(int x)
    {
        // Your complicated function with all the logic goes here
        return UnityEngine.Random.Range(1, 10) + _x;
    }
}
```
In example above `MyCustomFunction` would be executed 5 times. Using this package it could be re-written in the following way:
```
using UnityEngine;
using System;
using Memoization; // Ensure Memoization namespace is imported

public class Cube : MonoBehaviour
{
    void Start()
    {
        // "MyFunction" delegate will be called and say "105" will be returned
        MyCustomFunction(100);
        // "105" will be returned straight away
        MyCustomFunction(100);
        // "MyFunction" delegate will be called as there isn't parameter "200" cached yet. Say "205" will be returned 
        MyCustomFunction(200);
        // "205" will be returned straight away
        MyCustomFunction(200);
        // "105" will be returned straight away
        MyCustomFunction(100);
    }
    int MyCustomFunction(int x)
    {
        Func<int, int> MyFunction = (int _x) =>
        {
            // Your complicated function with all the logic goes here
            return UnityEngine.Random.Range(1, 10) + _x;
        };
        // Use "Once" extension to cache your delegate result
        return this.Once<int>(MyFunction, x);
    }
}
```
Package will ensure that `MyCustomFunction` is only executed 2 times in this example and all consequent calls with same parameters will return a cached result instead.
## Available Functions
The following functions are available when `using Memoization;` namespace is included:

 - `TOut Once<TOut>(Delegate, DelegateParameters)` - calls a `Delegate` with given `DelegateParameters`. When `Delegate` returns `TOut` it caches a result before returning it back to your code. If cache already contains a `TOut` for a given `Delegate` and a set of `DelegateParameters`, then `Delegate` is not called and cached `TOut` from previous calls is returned straight away.
 Example:
 ```
using UnityEngine;
using System;
using Memoization; // Ensure Memoization namespace is imported

public class Cube : MonoBehaviour
{
    void Start()
    {
        MyCustomFunction(100); // "MyFunction" delegate will be called and say "105" will be returned
        MyCustomFunction(100); // "105" will be returned straight away
        MyCustomFunction(200); // "MyFunction" delegate will be called as there isn't parameter "200" cached yet. Say "205" will be returned
        MyCustomFunction(200); // "205" will be returned straight away
        MyCustomFunction(100); // "105" will be returned straight away
    }
    int MyCustomFunction(int x)
    {
        Func<int, int> MyFunction = (int _x) =>
        {
            // Your complicated function with all the logic goes here
            return UnityEngine.Random.Range(1, 10) + _x;
        };
        // Use "Once" extension to cache your delegate result
        return this.Once<int>(MyFunction, x);
    }
}
 ```	
- `Memoizer.Disable()` - globally disables memoization caching for your whole project. Any already cached values are persisted and would be available if memoization caching is enabled again.
- `Memoizer.Enable()` - globally enables memoization caching for your whole project.
- `Memoizer.ClearAllCache()` - clears all cached values
- `Memoizer.ClearObjectCache(object)` or `this.ClearObjectCache()` - clears all cached values for a given object
## Advanced Usage
Memoization package uses [GetHashCode()](https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode?view=net-7.0) to compare passed parameters. By default, different objects (by reference) would generate different `GetHashCode()` , but if you want be able to achieve objects equality by value, you could override `GetHashCode()` against your class and return your own `int` that uniquely identifies your object.
```
using UnityEngine;
using System;
using Memoization;

public class Cube : MonoBehaviour
{
    public class Test
    {
        public int number = 5;

        // With this GetHashCode override "MyCustomFunction" would execute only once
        // Without this override it would be executed each time it is called
        public override int GetHashCode()
        {
            return number;
        }
    }

    void Start()
    {
        Test test1 = new Test();
        Test test2 = new Test();
        MyCustomFunction(test1);
        MyCustomFunction(test2);
    }

    int MyCustomFunction(Test _test)
    {
        Func<Test, int> MyFunction = (Test _test) =>
        {
            return _test.number + 10;
        };

        return this.Once<int>(MyFunction, _test);
    }

}
```
Any arrays or collection type variables are not iterated over when passed as parameters. `GetHashCode()` is always called directly on a parameter that has been passed. If you need an array / collection to have all its values evaluated, consider wrapping it in a class and use `GetHashCode()` to uniquely identify your object.

## Things To Keep In Mind
_A memoized function should not have side effects._  
Memoization will prevent the function from executing if the parameters match the previous ones therefore if you depend on some side effects resulting from the function execution you will lose them.

_A memoized function should be pure._  
That means that the function result is the same if the parameters are the same, if the function was to depend on other context it would be impossible to memoize correctly as only parameters can be cached. Common mistake would be to use things like `transform.position` in a delegate you are memoizing, because it's likely that `transform.position` would be different each time, but since it's not passed as a parameter, it would never be looked at and cached value would be returned.

_There should be 1 memoized function per object function_
Due to the way memoized functions are cached, you can only have one `this.Once()` call per function. If your complex function requires more than one delegate, then you should split each of them into individual functions on your object.

## Memory management
All values are cached in [ConditionalWeakTable](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.conditionalweaktable-2?view=net-7.0) by using object as a key. When object is destroyed, all memory is released and is made available for garbage collector to collect.

## Licence
Once you have purchased this package on [Unity's asset store](https://u3d.as/38ti) you are free to use it on any commercial projects of yours.

