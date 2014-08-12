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
foo  : Int
bar  : String
baz  : [Int]
quux : a
```

--

* Functions
```
mod : Int -> Int -> Int
id  : a -> a
any : (a -> Bool) -> [a] -> Bool
map : (a -> b) -> [a] -> [b]
fst : (a, b) -> a
```

--

* Compound types
```
ints : [Int]
name : Optional String
sth  : Optional a
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

What types to they have?

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

```
infix operator <*> { associativity left }

func <*> <A,B>(fm: Maybe<A -> B>, m: Maybe<A>) -> Maybe<B> {
    switch fm {
    case let .Just(f):
        return f <^> m
    case .Nothing:
        return .Nothing
    }
}
```

---
class: center, middle

# Questions?
