class: center, middle

# Optionals in Swift

---

## Swift

* Programming language announced in 2014 

* Mix of object oriented and **functional programming**

* Inspired by Objective-C, Rust, **Haskell**, Ruby, Python...

* During this hangout main focus will be `Optional` type

---

## Notation for types

* Values
```
foo : Int
bar : String
bar : a
```

--

* Functions
```
max : Int -> Int -> Int
id  : a -> a
fst : (a, b) -> a
any : (a -> Bool) -> [a] -> Bool
map : (a -> b) -> [a] -> [b]
```

--

* Compound types
```
ints : [Int]
name : Optional String
sth  : Optional a
```
---
## Swift syntax 101

* Values
```
let foo: Int = 10
let bar: String = "Swift"
let baz: [Int] = [1, 2, 3, 4]
```
--

* Functions
```
func max(a: Int)(b: Int) -> Int {
        if (a > b) { return a }
        else       { return b }
}
```
```
func id<A>(a: A) -> A { return a }
func fst<A,B>(a: A, b: B) -> A { return a }
let increment: Int -> Int = { $0 + 1 }
```
--

* Compound types
```
let ints: [Int] = [1, 2, 3]
let name: Optional<String> = .None
```

---

## Curried functions

Every function can only take one parameter.

There are two ways multiparameter functions can be simulated:

--

* Use a tuple as the single parameter
```
min : (Int, Int) -> Int
```

--

* Currying
```
min : Int -> (Int -> Int)
min : Int ->  Int -> Int
```

Curried function takes a value and returns a function that takes another value
and then returns the result.

--

**Both approaches yield the same result, sometimes one of them is more
convenient than the other.**
---

## Swift syntax 102

Swift allows to write both "regular" and curried functions.

The difference may be hard to notice.

* Regular function
```
func min(a: Int, b: Int) -> Int {
        if (a < b) { return a }
        else       { return b }
}
```

* Curried function
```
func min(a: Int)(b: Int) -> Int {
        if (a < b) { return a }
        else       { return b }
}
```

---


## `Optional` type

It can be interpreted as a:

* container type that can hold `Some` value or `None` at all

--

* computation that can fail (imagine: looking up a word in a
  dictionary)

--

Example usage in Swift

```
func safeDivision(a: Int, b: Int) -> Optional<Int> {
   if (b == 0) {
       return .None
   } else {
       return .Some(a/b)
   }
}
```

--

A neat trick, but **we have lost something**.

--

It's not possible to write `safeDivision(2, 3) + 1` now, because `+` won't work
with `Optional<Int>`.
---

## `Optional` chaining

To recover from that loss Swift provides two operators `?` and `!`.

```
let movies = [
    "Pulp Fiction":   ["rating": 4, "year": 1994],
    "Reservoir Dogs": ["rating": 5, "year": 1992]
]

let maybeYear: Optional<Int> = movies["Pulp Fiction"]?["year"]
```

`?` allows to chain functions on optionals.

--

```
let year: Int = maybeYear!
```

`!` extracts the value from optional if there is one.

--

What types do they have?

--
```
! : Optional a -> a

? : Optional a -> (a -> Optional b) -> Optional b
```
---

## Rolling our own `Optional`


First define `Maybe` data type.

--

```
enum Maybe<A> {
    case Just(A)
    case Nothing
}
```
Next, write the chaining function `>>=`.

--
```
infix operator >>= { associativity left }

func >>= <A,B>(m: Maybe<A>, f: A -> Maybe<B>) -> Maybe<B> {
    switch m {
    case let .Just(a):
        return f(a)
    case .Nothing:
        return .Nothing
    }
}
```

Let's try it out!

---

## `Maybe` test drive

We only need to adjust the types in `safeDivision` from previous slide a little
bit...
```
let a = safeDivision(90, 3) >>=
        { .Just(2 * $0) }   >>=
        { .Just(3 + $0) }
```
After running that code `a` is now equal to `.Just(63)`.

--

```
let b = a >>= { .Nothing } >>= { .Just($0 - 1) }
```

Now `b` is `.Nothing` and the last block won't be run at all.

--

The only thing we lack is the ability to unwrap the value.

--

Let's define `&` operator that:

* returns the value when we have it (`.Just` case)

* terminates the program when we don't (`.Nothing` case)

---

## Unwrapping

This should be simple, right?
--
 **Nope**.

```
postfix operator & {}

postfix func & <A>(m: Maybe<A>) -> A {
    switch m {
    case let .Just(a):
        return a
    case .Nothing:
        // Oh, bugger...
        // We don't have an A, but we need to return one
    }
}
```
--

One thing we can do is to use **bottom** value.

Bottom represents a non-terminating computation which either loops forever or
fails due to an error.

Bottom value has **any** type.

---

## Making the type checker happy

First we should invent a way to get an `A` out of nowhere.
--

```
func bottom<A>() -> A { return bottom() }
```

Next thing is to make the program crash with an appropriate message.
--

```
func error<A>(msg: String) -> A {
    NSException(name: NSInternalInconsistencyException,
        reason: msg, userInfo: nil).raise()
    return bottom()
}
```

--
Complete `&` operator:
```
postfix func & <A>(m: Maybe<A>) -> A {
    switch m {
    case let .Just(a):
        return a
    case .Nothing:
        return error("Expected something, but found Nothing")
    }
}
```
---
## Lifting

There's one more thing that `Optional` has and our implementation doesn't.

Ability to take a single argument function like `increment` and make it work on
optional values.
```
func increment(a: Int) -> Int { return a + 1 }

let ten: Optional<Int> = 10
let eleven = ten.map(increment)
```

This concept is known as **lifting**. Let's define such function for `Maybe`.

--

```
infix operator <^> { associativity left }

func <^> <A,B>(f: A -> B, m: Maybe<A>) -> Maybe<B> {
    switch m {
    case let .Just(a):
        return .Just(f(a))
    case .Nothing:
        return .Nothing
    }
}
```
---
## Multiparameter lifting

Now we can write something like this:
```
let ten: Maybe<Int> = .Just(10)

let eleven = increment <^> ten
```

--
When we squint, it looks somewhat like code below, but it does a bit more
```
let eleven = increment(ten)
```

--
There's one drawback. It only works with functions that take a single
parameter.

However there is a way to extend it with currying.
```
<^> : (a -> b) -> Maybe a -> Maybe b
```
--

`b` is an arbitrary type, so it may as well be `c -> d`
```
<^> : (a -> c -> d) -> Maybe a -> Maybe (c -> d)
```
--
Now we need a function with type: `Maybe (c -> d) -> Maybe c -> Maybe d`

---
## Multiparameter lifting

That function can be implemented in terms of `<^>`

```
infix operator <*> { associativity left }

func <*> <C,D>(fm: Maybe<C -> D>, m: Maybe<C>) -> Maybe<D> {
    switch fm {
    case let .Just(f):
        return f <^> m
    case .Nothing:
        return .Nothing
    }
}
```
--

We can use it to lift a function that takes **any number** of arguments as long it
is curried.
```
func diff(a: Int)(b: Int) -> Int { return abs(a - b) }

let result = diff <$> safeDivision(x, y) <*> someValue
```

--
Which looks oddly similar to code below
```
let result = diff(safeDivision(x, y))(someValue)
```

---
class: center, middle

# Questions?
