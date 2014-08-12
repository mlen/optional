class: center, middle, inverse

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

