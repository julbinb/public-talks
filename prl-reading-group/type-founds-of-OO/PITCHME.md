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

## Language F1

<p style="text-align:center;font-size:1.25em"><b>Simple types, records, and references</b></p>

Types:

```less
ty, A, B = Unit | Bool | ..   // basic types
   | A → B                    // function
   | {l1:A1, .., ln:An}       // record
   | Ref[ty]                  // reference
```

Terms:

```less
tm, M, N = x | ...            // variable    | literals
   | λx:A.M | M N             // abstraction | application
   | let x=M in N | M ; N     // let-binding | composition
   | {l1=M1, .., ln=Mn}       // record
   | ref(tm)                  // reference 
   | !tm | M := N             // dereference | assignment
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

@title[Part 2. Object-Oriented Languages]

## Part 2
# Object-Oriented Languages

---

blaaa

```
Γ ⊢ λx:A.M: A→B
```
