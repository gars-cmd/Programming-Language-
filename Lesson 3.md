
according to what we saw in [[Lesson 2]]  , If we want to create a function that return the average of 2 numbers :

```racket
(: average2 : Number Number -> Number)
(define (average2 x y)
(\ (+ x y) 2)
)
```


factorial function  :

```racket
(: factorial : Number -> Number )
(define (factorial  n)
(cond [(= 0 n) 1]
	[else (* n (factorial (- n 1) ))]
))
```

length function for array (recursive way) :

```racket
(: len : (Listof Number) -> Number )
(define (len lst)
(cond [(null?) 0]
	[else (+ 1 (len(rest lst)))]
))
```


Because our language is functional there is no loop (for , while etc..).

sum of a list of number :
```racket 
(: sum : (Listof Number ) -> Number)
(define (sum lst)
(cond [(null? lst) 0]
	[else (+ (first lst) (sum(rest lst)))]
))
```

we will use the sum function and the len function for the average function  on list of number :

```racket
(: LstAvg : (Listof Number) -> Number)
(define (LstAvg lst)
(/ (sum lst) (len lst)
))
```


remove a particular item of a list :

```racket
(: removeItem : (Listof Number) -> (Listof Number ) )
(define (removeItem lst item)
(cond [(null? lst) null]
	[(= (first lst) item) (removeItem (rest lst) item)]
	[else (cons (first lst) (removeItem (rest lst) item))]
))
```


### Define a data-type Definition:

define a new type of Data :

```racket
(define-type Animal
[Snake Symbol Number Symbol]
[Tiger Symbo Number]
)
```

return properties of the type:

```racket
(: animal-surname : Animal -> Symbol)
(define (animal-surname a)
(cases a
[(Snake sn w f) n]
[(Tiger n d ) n]
	)
)
```


condition on types :

```racket
(: animal-weight : Animal -> (U Number #f))
(define (animal-weight a)
(cases a
[(Snake n w f) w]
[else #f]
	)
)
```



### Arithmetic Expression - AE

BNF - Backus Naur Form -> DIKDOUK

To make all number operation  work in our language like :
"5, 1+4, 4-2, 2+3-5"

We first define our AE to represent numbers :
\<AE> ::= \<num>

And we define the operation to made with them :
[1] \<AE> ::= \<num>  |
[2] \<AE> + \<AE> |
[3] \<AE> - \<AE>

to get the operation 1+4-2 , we can recreate it with the definition we already made :
\<AE> => \<AE> - \<AE> ([3])
= \<AE> => (\<AE> + \<AE>) - \<AE>  ([2])
then we get also in total (3 x  [1])


With Tree : 
![[Pasted image 20221103164858.png]]


!  Warning about the order of  the operation  (Ambiguities)
![[Pasted image 20221103165617.png]]


---
# Tirgul 

### Difference between recursion and tail recursion
> From ChatGPT : "Tail recursion is a specific form of recursion where the recursive call is the last thing that happens in the function. In other words, the recursive call is in the tail position and there is nothing left to execute after the recursive call returns.
A key difference between recursion and tail recursion is that tail recursion can be optimized by the compiler. When a function is tail recursive, the compiler can convert the recursive function call into a loop, which uses less memory and is more efficient. However, not all recursive functions can be optimized as tail recursive"

***Regular recursion exemple:***
```racket
(: fact : Natural -> Natural )
(define (fact n)
	(if (zero? n)
	1
		(* n (fact (- n 1)))))
```

***Tail Recursion exemple:***
```racket
(: helper : Natural Natural -> Natural)
(define (helper n acc)
	(if (zero? n)
		acc
		(helper (- n 1) (* acc n))))

(: fact : Natural -> Natural)
(define (fact n)
	(helper n 1))

; another exemple on the length function 

(: list-length-tail : ( Listof Any ) -> Natural )
( define ( list-length-tail ls )
	(: helper-list-length-tail : Natural ( Listof Any ) -> Natural )
	( define ( helper-list-length-tail acc ls )
		(if ( null? ls )
			acc
			( helper-list-length-tail (+ 1 acc) ( rest ls ))))
		(helper-list-length-tail 0 ls))
```

There is more exemples of Recursion and Tail-recursion in Tirgul 3

#### Define Data-type 

```racket
#|
(define-type <name>
	[<var1> <type> ...]
	[<var2> <type> ...]
	[...])
|#

(define-type Animal 
	[Snake Symbol Number Symbol ]
	[Tiger Symbol Number ])
```

After the initialization of the new type we can test a variant of the type .
```racket
(Animal? (Tiger 'Tony 12)) ;=> #t
(Animal? (Snake 'Slimey 10 'rats)) ;=> #t
(Animal? (cons 'Robi 19)) ;=> #f

```

We can use the built-in function `cases` to check condition on a variant of a type .
```racket
(cases (Snake 'Slimey 10 'rats) 
	[(Snake n w f) n] 
	[(Tiger n sc) n]) 
'Slimey
```

In cases we need to wrap into the call all variant of a type , if no we will get an error .
In cases we do not need an else case .

```racket
;Error exemple 
(cases (Snake 'Slimey 10 'rats)
	[(Snake n w f) n]) 
```


**Exercice : Define a type Person** (see Tirgoul 3)
```racket
(define-type Person 
	[Teacher String Natural Integer]
	[Student String Natural Real ])

```

**Exercice given this type return a list with the number of Teacher and the Number of Student** 
```racket
(: Person-Number : (Listof Person) -> (Listof Natural ))
(define (Person-Number lst)
	(Person-Number-h lst 0 0))
(: Person-Number-h : (Listof Person) Natural Natural -> (Listof Natural))
(define (Person-Number-h lst tn sn)
	(if (null? lst)
		(list tn sn)
		(cases (first lst)
			[(Student x y z) (Person-Number-h (rest lst) tn (+ sn 1))]
			[(Teacher x y z) (Person-Number-h (rest lst) (+ tn 1) sn)])))
```

Other exercice in Tirgul 3