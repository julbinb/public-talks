#### Type-theoretic foundations of object-oriented languages

#### Or 

#### Everything old is new again 

<br/>

<p style="text-align: center">
PRL Reading Group<br/>
Tue Jan/30/2018<br/>
<br/>
Julia Belyakova
</p>

---
@title[Part 1. Type Systems]

### Part 1
## Type Systems

---

### References

1. [On Understanding Types, Data Abstraction, and Polymorphism](http://lucacardelli.name/Papers/OnUnderstanding.A4.pdf).
   _Luca Cardelli, Peter Wegner_.  
   ACM Computing Surveys. 1985.

1. [Type Systems](http://lucacardelli.name/Papers/TypeSystems.pdf). _Luca Cardelli_.  
   ACM Computing Surveys. 1996.

---

#### Language F1. Simple types, records, and references
<!-- first-order language -->

Types

```less
A, B = Unit | Bool | ..      // basic types
   | A → B                   // function type
   | {l1:A1, .., ln:An}      // record type
   | Ref[A]                  // reference type
```

Terms

```lasso
M, N = x | 0, ..             // variable    | literals
   | λx:A.M                  // abstraction (function)
   | M N                     // application (call)
   | let x=M in N | M ; N    // let-binding | composition
   | {l1=M1, .., ln=Mn}      // record
   | ref(M)                  // reference 
   | !M | M := N             // dereference | assignment
```

---

#### Private Local State

```less
let count =                               class Counter {
  let // private data
    cnt = ref(0)                            private Int cnt = 0
  in  // obj with interface {inc,tot}
    { inc = λx:Int.  cnt := !cnt + x,       Void inc(Int x){..} 
      tot = λx:Unit. !cnt }                 Int  tot(){ret cnt;}
in                                        }
  count.inc 1 ;                           count = new Counter();
  count.inc 5 ;                           count.inc(1);
  count.tot ()         // yields 6
```

Typing (`Γ ⊢ M : A`)

```sml
        ⊢ 0 : Int         cnt:Ref[Int] ⊢ inc : Int→Unit  ...
  ---------------------  ------------------------------------
   ⊢ ref(0) : Ref[Int]    cnt:Ref[Int] ⊢ {inc=..} : {inc:..}
 -------------------------------------------------------------
    ⊢ (let cnt=ref(0) in ..) : {inc:Int→Unit, tot:Unit→Int}
```

---

#### Language F1<sub><:</sub> with Subtyping

Subtyping

```less
   Γ ⊢ A <: A                            Γ ⊢ A <: Top

           Γ ⊢ A1 <: B1   ...   Γ ⊢ Ak <: Bk
   ---------------------------------------------------
    Γ ⊢ {l1:A1,..,lk:Ak,..,ln:An} <: {l1:B1,..,lk:Bk}

```

Typing

```dns
       Γ ⊢ M : A → B    Γ ⊢ N : A'    Γ ⊢ A' <: A
      --------------------------------------------
                       Γ ⊢ M N : B
```

Example

```sml
let distO = λp:{x:Int,y:Int}.sqrt (x*x+y*y) (*: Point → Flt *)
in  distO {x=3,y=4,w=2.5}   (* {x:Int,y:Int,w:Flt} <: Point *)
```

---

#### Language F2. Universal types

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
  ------------------     --------------------------
   Γ ⊢ λX.M : ∀X.A          Γ ⊢ M [B] : [B/X]A
```

---

#### Parametric Polymorhism (Generics)

Generic identity function

```sml
let id = λX.λx:X.x    (* id      : ∀X→X                      *)
in  (id [Int] 4)      (* id[Int] : Int→Int; (id.. ) yields 4 *)
```

Type of generic stack

```less
∀Item.{push  : Item × List[Item] → List[Item],
       pop   : List[Item]        → List[Item], 
       top   : List[Item]        → Item,
       empty : List[Item]}
```

Implementation of generic stack

```less
λItem.{push  = λx:Item.λs:List[Item]. cons [Item] x s,
       pop   = λs:List[Item].         tail [Item] s, 
       top   = λs:List[Item].         head [Item] s,
       empty = nil[Item]}
```

---

#### Language F2 with Existential types

Syntax

```clean
A, B, C = ..                // types of F2
   | ∃X.B                   // existential type

M, N, P = ..                // terms of F2
   // value M packed into pkg of type ∃X.A with hidden type B
   | pack X=B in A with M
   // use of value x:A from pkg P in term N
   | open P as X,x:A in N
```

Typing

```clean
        Γ ⊢ M : [B/X]A           Γ ⊢ P:∃X.A  Γ,X,x:A ⊢ N:C  Γ ⊢ C
-------------------------------  --------------------------------
Γ ⊢ pack X=B in A with M : ∃X.A    Γ ⊢ open P as X,x:A in N : C
```

---

#### Abstract Data Types

Abstract datatype of generic stack

```lasso
GenericStack = ∀Item.∃Stack.StackRecord // shorthand
StackRecord  = {push : Item×Stack → Stack,  pop  : Stack → Stack, 
                top  : Stack      → Stack,  empty: Stack}
```

Implementation of GenericStack abstract datatype

```sml
let listStackPackage (* : GenericStack *) = 
λItem.pack Stack=List[Item] as StackRecord with
      {push  = λx:Item.λs:List[Item]. cons[Item] x s,
       pop   = λs:List[Item].         tail[Item] s, 
       ... }
```

Use of GenericStack

```sml
in open listStackPackage[Int] as Stack, s:{push:Int×Stack→..}
in s.top (s.push 10 (s.push 1 s.empty)) (* : Int — ok *)
(* in s.empty (* : Stack — type-error *) *)
```

---

#### Language F2<sub><:</sub> with Bounded quantification

Syntax

```lasso
A, B, C = ..                  // types of F2
   | ∀X<:A.B                  // instead of ∀X.B  (~ ∀X<:Top.B)
   | ∃X<:A.B                  // instead of ∃X.B

M, N = ..                     // terms of F2
   | λX<:A.M                  // instead of λX.M  (~ λX<:Top.M)
   | pack X<:A=B in C with M  // instead of pack X=B in ...
```

Example

```sml
getx = λp:Point.       (p.x)             (* Point→Int *)
mvdx = λd:Int.λp:Point.(p.x := p.x+d; p) (* Int→Point→Point *)

gety = λP<:Point.λp:P. (p.y)             (* ∀P<:Point.P→Int *)
mvdy = λP<:Point.
       λd:Int.λp:P.    (p.y := p.y+d; p) (* ∀P<:Point.Int→P→P *)
```

---

@title[Part 2. Object-Oriented Languages]

### Part 2
## Object-Oriented Languages

---

blaaa

```
Γ ⊢ λx:A.M: A→B
```
