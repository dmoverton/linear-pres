build-lists: true
slidenumbers: true
autoscale: true

# Linear Types in Haskell

### David Overton
### Melbourne Haskell Users Group - July 2020

---

## Motivation

---

* File handles
* Arrays
* `IO` world state
* Session types

---

## Proposal

---

### Linearity

A function `f` is _linear_ when
- `f u` is consumed exactly once
- implies that `u` is consumed exactly once:

- Consuming a value of a data type exactly once means evaluating it to head normal form exactly once, discriminating on its tag any number of times, then consuming its fields exactly once.
- Consuming a function exactly once means applying it and conuming its result exactly once.

---

### Basic syntax

```haskell
A #-> B
```
is the type of linear functions from `A` to `B`.

---

### Algebraic data types

Most "normal" data types will have linear constructors. E.g.

```haskell
data Maybe a
    = Nothing
    | Just a
```
is equivalent to

```haskell
data Maybe a where
    Nothing :: Maybe a
    Just :: a #-> Maybe a
```

---

### Unrestricted

Explicitly _unrestricted_ or non-linear constructor using GADT syntax:

```haskell
data Unrestricted a where
  Unrestricted :: a -> Unrestricted a
```

---

### Polymorphism

To avoid code duplication functions can have _multipicity polymorphism_.

```haskell
map :: (a #p-> b) -> [a] #p-> [b]
```

where `p` is a multiplicity parameter.

- Linear functions have multiplicity $$1$$ or `One`
- Unrestricted functions have multiplicity $$\omega$$ or `Many`

---

### Multiple multiplicities

```haskell
(.) :: (b #p-> c) -> (a #q-> b) -> a #(p ':* q)-> c
```
would require 4 different types without polymorphism:

```haskell
(.) :: (b ->  c) -> (a ->  b) -> a ->  c
(.) :: (b #-> c) -> (a ->  b) -> a ->  c
(.) :: (b ->  c) -> (a #-> b) -> a ->  c
(.) :: (b #-> c) -> (a #-> b) -> a #-> c
```

---

### Details

[.code-highlight: 1]
[.code-highlight: 3-5]
[.code-highlight: 7-10]
[.code-highlight: 12-13]
```haskell
data Multiplicity = One | Many

-- Type families
type family (:+) (p :: Multiplicity) (q :: Multiplicity) :: Multiplicity
type family (:*) (p :: Multiplicity) (q :: Multiplicity) :: Multiplicity

-- New type constructor
FUN :: Multiplicity -> forall (r1 r2 :: RuntimeRep). TYPE r1 -> TYPE r2

-- FUN p a b ~ a #p-> b

type (->) = FUN 'Many
type (#->) = FUN 'One
```

---

### Multiplicity annotations

Multiplicity and type:

```haskell
\x :: A # 'One -> x
```

Multiplicity without type:

```haskell
\x # 'One -> x
```

Records:

```haskell
data R = R { unrestrictedField # 'Many :: A, linearField # 'One :: B }
```

---

## Linear base library

---

## I/O

```haskell
-- Ensures I/O state is linearly threaded, meaning it is safe to
-- expose the IO constructor
newtype IO a = IO (State# RealWorld #-> (# State# RealWorld, a #))

-- hClose consumes the handle so it can't be used again after it's closed.
hClose :: Handle #-> IO ()

-- Other I/O operations must also be linear with respect to handle
-- meaning each operation needs to return a new handle.
hPutChar :: Handle #-> Char -> RIO Handle
```

---

## Examples

---

### I/O

```haskell
linearGetFirstLine :: FilePath -> RIO (Unrestricted Text)
linearGetFirstLine fp = do
  handle <- Linear.openFile fp System.ReadMode
  (t, handle') <- Linear.hGetLine handle
  Linear.hClose handle'
  return t
  where
    Control.Builder {..} = Control.monadBuilder

linearPrintFirstLine :: FilePath -> System.IO ()
linearPrintFirstLine fp = do
  text <- Linear.run (linearGetFirstLine fp)
  System.putStrLn (unpack text)
```

---

## Exceptions

---

## Related Work

---

- *Linear Types Can Change the World!*, Philip Wadler, 1990
- Concurrent Clean
- Mercury
- Rust

---

## The Linearity Design Space[^1]

|  | Linearity on the arrows | Linearity on the kinds |
| --- | --- | --- |
| Linear types | Linear Haskell | Rust (borrowing) |
| Uniqueness types | | Rust (Ownership), Concurrent Clean |

[^1]: Taken from [SPJ's talk](https://youtu.be/t0mhvd3-60Y)

---

## References

[.build-lists: false]
[.autoscale: true]
- *[Linear Haskell](https://arxiv.org/pdf/1710.09756.pdf) - Practical Linearity in a Higher-Order Polymorphic Langauge*, Bernardy et al, Nov 2017.
- [GHC Proposal](https://github.com/ghc-proposals/ghc-proposals/blob/master/proposals/0111-linear-types.rst)
- [Linear Base Library](https://github.com/tweag/linear-base)
- [Examples](https://gitlab.haskell.org/ghc/ghc/-/wikis/linear-types/examples)
- [Simon Peyton Jones - Linear Haskell: practical linearity in a higher-order polymorphic language](https://youtu.be/t0mhvd3-60Y)
- *[Linear types can change the world!](https://pdfs.semanticscholar.org/24c8/50390fba27fc6f3241cb34ce7bc6f3765627.pdf)* - Philp Wadler, 1990