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

```less
                    typed lambda calculus
                        as a model of
         statically typed object-oriented language
```

---

### References

1. [On Understanding Types, Data Abstraction, and Polymorphism](http://lucacardelli.name/Papers/OnUnderstanding.A4.pdf).
   _Luca Cardelli, Peter Wegner_.  
   ACM Computing Surveys. 1985.

1. [Type Systems](http://lucacardelli.name/Papers/TypeSystems.pdf). _Luca Cardelli_.  
   ACM Computing Surveys. 1996.

<span style="font-size: 90%">Alternative approach (late 90-s/early 2000-s) —
Little&nbsp;Java, Featherweight&nbsp;Java.</span>

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
let count =                              class Counter {
  let // private data
    cnt = ref(0)                           private Int cnt = 0
  in  // obj with interface {inc,tot}
    { inc = λx:Int.  cnt := !cnt + x,      Void inc(Int x){..} 
      tot = λx:Unit. !cnt }                Int  tot(){ret cnt;}
in                                       }
  count.inc 1 ;                          count = new Counter();
  count.inc 5 ;                          count.inc(6); count.tot()
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
A, B = .. | List[A]           // types of F1 + built-in list
   | X                        // type variable
   | ∀X.A                     // universally quantified type
```

Terms

```less
M, N = .. | cons[A](M,N) | .. // terms of F1 + list primitives
   | λX.M                     // type abstraction
   | tm [A]                   // type application (instantiation)
```

Typing

```less
(T-TyAbs)            (T-TyApp)
     Γ, X ⊢ M : A         Γ ⊢ N : ∀X.A     Γ ⊢ B
  ------------------     --------------------------
   Γ ⊢ λX.M : ∀X.A          Γ ⊢ N [B] : [B/X]A
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
∀Item.{push  : Item       → List[Item] → List[Item],
       pop   : List[Item] → List[Item], 
       top   : List[Item] → Item,
       empty : List[Item]}
```

Implementation of generic stack

```less
λItem.{push  = λx:Item.λs:List[Item]. cons[Item](x, s),
       pop   = λs:List[Item].         tail[Item](s), 
       top   = λs:List[Item].         head[Item](s),
       empty = nil[Item]}
```

---

#### Language F2 with Existential types

Syntax

```clean
A, B, C = ..               // types of F2
 | ∃X.B                    // existential type

M, N, P = ..               // terms of F2
 // value M packed into pkg of type ∃X.A with hidden type B
 | pack X=B in A with M
 // pkg = pack X=Int in {v:X, f:X→Int} with {v=0, f=λx:Int.x+1}
 
 // use of value x:A from pkg P in term N
 | open P as X,x:A in N
 // open pkg as X, x:{v:X, f:X→Int} in x.f (x.f x.v) // yields 2
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
StackRecord  = {push: Item → Stack → Stack, pop  : Stack → Stack, 
                top : Stack→ Stack,         empty: Stack}
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
in open listStackPackage [Int] as Stack, s:{push:Int×Stack→..}
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

```less
                          data abstraction
                                 +
    object-oriented   =     object types     (Luca Cardelli)
                                 +
                          type inheritance
```

</br>

#### SIMULA 67


```less
                    object types ~ classes
               type inheritance  ~ class inheritance
```
---

#### Structural Subtyping

<div style="text-align: center; font-size: 60%; margin-top:-20px">
(Emerald, PolyTOIL in late 80-s/90-s)</div>

```sml
type Meter{ v: Float }    type ColMeter{ v: Float, c: Color }
type Feet { v: Float }

fun calcFuel(dist : Meter, ...) : Float ... end

val fuel = calcFuel(Meter{v=1000}, ...)           (* ok *)
val fuel = calcFuel(ColMeter{v=1000}, ...)        (* ok *)
val fuel = calcFuel(Feet {v=1000}, ...)           (* ok :( *)
```

#### Nominal Subtyping (Inheritance)

```java
class Meter{ Float v }    class ColMeter extends Meter{ Color c }
class Feet { Float v }

Float calcFuel(Meter dist, ...) { ... }

var fuel = calcFuel(new Meter(1000, ...))         // ok
var fuel = calcFuel(new ColoredMeter(1000, ...))  // ok
var fuel = calcFuel(new Feet (1000), ...)         // type error :)
```

---

#### Generic Programming </br> (Bounded Parametric Polymorphism)

```dart
class Showable { String show(){ return "Showable" } }
// λT<:Showable. λxs:List[T]. ..    // xs is homogeneous
<T extends Showable> String showAll(List<T> xs){.. x.show() ..}

class Person{ .. String show(){..} }
// ??? showAll<Person>

// ShowablePerson cannot extend both Showable and Person
class ShowablePerson extends Showable {.. String show(){..} }
showAll<ShowablePerson>(..); // ok

class Weighed { .. }

// ??? class WeighedShowableStudent

```

---

* [Subtypes vs. Where Clauses: Constraining Parametric Polymorphism](http://www.pmg.lcs.mit.edu/papers/where-clauses.pdf).
  _Mark Day, Robert Gruber, Barbara Liskov, Andrew C. Myers_. OOPSLA 1995.

Constraints on type parameters via _where clauses_
instead of subtype bounds (language Theta).

```csharp
showAll[T](xs: List[T]): String 
    where T has show() returns(String)
```

---

#### Interfaces (Java, late 90-s)

```dart
interface Showable { String show(); }

<T implements Showable> String showAll(List<T> xs){.. x.show() ..}

class Person implements Showable { .. String toString(){..} }
showAll<Person>(..);      // ok

interface Weighed { .. }
<T implements Weighed> Float totalWeight(List<T> xs){ .. }

class Student extends Person implements Weighed { .. }
totalWeight<Student>(..); // ok

// ??? totalWeight<Person>(..);
```

<div style="text-align:center"> Next step — retroactive modeling.</div>

---

#### Protocols (Swift, 2014)

```swift
protocol Showable { func show() -> String; }
func showAll<T: Showable>(xs: List<T>) {.. x.show() ..}

class Person: Showable { .. }
showAll<Person>(..);      // ok

protocol Weighed { .. }
fun totalWeight<T: Weighed>(xs: List<T>){ .. }

class Student: Person, Weighed { .. }
totalWeight<Student>(..); // ok

extension Person: Weighed { .. }
totalWeight<Person>(..);  // ok
```

---

#### Multi-type constraints

Theta

```csharp
member[E, C](..): Bool
    where E has equal(E)   returns(Bool)
          C has elements() returns(List[E])
```

Swift

```swift
protocol Equatable  { func equal(x: Self) -> Bool }
protocol Collection { 
    associatedtype Elem;
    func elements() -> List<E>;
}

member<E: Equatable, C: Collection>(..) -> Bool
    where C.Elem == E

```

---

## What's next?

