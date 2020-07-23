build-lists: true
slidenumbers: true

# Linear Types in Haskell

## David Overton
### Melbourne Haskell Users Group - July 2020

---

## Hello

```haskell
main :: IO ()
main =
    putStrLn "Hello, World!"
```

---

## Motivation

* File handles
* Arrays
* `IO` world state
* Session types

---

## Proposal

---

## Exceptions

---

## Related Work

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

- *[Linear Haskell](https://arxiv.org/pdf/1710.09756.pdf) - Practical Linearity in a Higher-Order Polymorphic Langauge*, Bernardy et al, Nov 2017.
- [GHC Proposal](https://github.com/ghc-proposals/ghc-proposals/blob/master/proposals/0111-linear-types.rst)
- [Examples](https://gitlab.haskell.org/ghc/ghc/-/wikis/linear-types/examples)
- [Simon Peyton Jones - Linear Haskell: practical linearity in a higher-order polymorphic language](https://youtu.be/t0mhvd3-60Y)
- *[Linear types can change the world!](https://pdfs.semanticscholar.org/24c8/50390fba27fc6f3241cb34ce7bc6f3765627.pdf)* - Philp Wadler, 1990