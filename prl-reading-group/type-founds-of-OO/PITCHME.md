### Type-theoretic foundations of object-oriented languages

<p style="color:#2266aa; text-align: center">Everything old is new again...</p>

<br/>

<p style="text-align: center">
PRL Reading Group<br/>
Tue Jan/30/2018<br/>
<br/>
Julia Belyakova
</p>

---
@title[Part 1. Type Systems]

## Part 1
# Type Systems

---

### References

1. [On Understanding Types, Data Abstraction, and Polymorphism](http://lucacardelli.name/Papers/OnUnderstanding.A4.pdf).
   _Luca Cardelli, Peter Wegner_.  
   ACM Computing Surveys. 1985.

1. [Type Systems](http://lucacardelli.name/Papers/TypeSystems.pdf). _Luca Cardelli_.  
   ACM Computing Surveys. 1996.

---

### Language F1. Simple types, records, and references
<!-- first-order language -->

Types

```less
A, B = Unit | Bool | ..      // basic types
   | A → B                   // function type
   | {l1:A1, .., ln:An}      // record type
   | Ref[A]                  // reference type
```

Terms

```less
M, N = x | 0, ..             // variable    | literals
   | λx:A.M                  // abstraction (function)
   | M N                     // application (call)
   | let x=M in N | M ; N    // let-binding | composition
   | {l1=M1, .., ln=Mn}      // record
   | ref(M)                  // reference 
   | !M | M := N             // dereference | assignment
```

---

### Private Local State

```less
let counter = 
    let // private data
        count = ref(0)
    in  // object with interface {inc, tot}
        { inc = λx:Int. count := !count + x,
          tot = λx:Unit. !count }
in
    count.inc 1 ;
    count.inc 5 ;
    count.tot ()         // yields 6
```

Typing:

```less
       ⊢ 0 : Int         count:Ref[Int] ⊢ inc : Int→Unit  ...
 ---------------------  --------------------------------------
  ⊢ ref(0) : Ref[Int]    count:Ref[Int] ⊢ {inc=..} : {inc:..}
--------------------------------------------------------------
   ⊢ (let count=ref(0) in ..) : {inc:Int→Unit, tot:Unit→Int}
```

---

### Language F2. Universal types

Types

```less
A, B = ..                // types of F1
   | X                   // type variable
   | ∀X.A                // universally quantified type
```

Terms

```less
M, N = ..                // terms of F1
   | λX.M                // type abstraction
   | tm [A]              // type application (instantiation)
```

Typing

```less
(T-TyAbs)            (T-TyApp)
    Γ, X ⊢ M : A         Γ ⊢ M : ∀X.A     Γ ⊢ B
  ----------------     --------------------------
   Γ ⊢ λX.M: ∀X.A          Γ ⊢ M [B] : [B/X]A
```

---

### Parametric Polymorhism (Generics)

Generic identity function:

```less
let id = λX.λx:X.x      // id : ∀X→X
in  (id [Int] 4)        // yields 4
```

Type of generic stack:

```less
∀Item.{push : Item×List[Item] → List[Item],
       pop  : List[Item] → List[Item], 
       empty: List[Item],     top: List[Item] → Item}
```

Implementation of generic stack:

```less
λItem.{push = λx:Item.λs:List[Item]. cons[Item] x s,
       pop  = λs:List[Item]. tail [Item] s, ..}
```

---

@title[Part 2. Object-Oriented Languages]

## Part 2
# Object-Oriented Languages

---

blaaa

```
Γ ⊢ λx:A.M: A→B
```
