**Formal specification of eval :**
1. eval (N) = N 
2. eval ({+ E1 E2}) = eval (E1) + eval (E2)
3. eval ({- E1 E2}) = eval (E1) - eval (E2)
4. eval ({\* E1 E2}) = eval (E1) * eval (E2)
5. eval ({ E1 E2}) = eval (E1) / eval (E2)
6. eval(id) = error!
7. eval({with {x E1} E2}) = E2 (\[eval (E1) \x])  
8. eval ( { fun {x} E } ) = { fun {x} E }
9. eval ( { call E1 E2 } ) = if {fun {x} E} <--  eval (E1)
							eval ( E1 \[ eval (E2) \x ] )
						else : error!

>Remider : the match statement in racket can be use only on the defined object in racket , when cases can be used on the object we create .


We started by defining the operation of our language (define-type)
"With"  is setting the variable id to be something
"id" represent  the variable 
"fun" represent the function 

**parsing** it's the part of receiving the code and to cast it to FLANG
**eval** take the abstraction of the FLANG by the parsing and compute the operation that we wrote .

For a nested With like this 
```racket
{with {..{...}}}
	{with {..{...}}}
		{with {..{...}}}
			{with {..{...}}}

;We lost a lot of performance because of all the replacement 
;Then we want to improve our With 
```

To do so we are gonna add for each time we meet a new variable and is value , the value of it like : 
```racket

{with {..{...}}} []
	{with {..{...}}} [(x value1)]
		{with {..{...}}} [(x value1)(y value2)]
			{with {..{...}}}  [(x value1)(y value2)(z value3)]
```

i add the every new variable that i met like in a stack LIFO

![[Pasted image 20230109210205.png]]

In this exemple at the last line we will get : 
`- y=9 (* z=10 (/ x=8 7)) ` but the return sentence that i get is the first ( `{ x {+ 3 2}}`) because the rest is the body of the all sentence.

Now to adapt the code to this changement we will modify only the eval function .

we will set :
1. Empty to be an empty list 
2. Extend to add pair to the list

```racket
(define-type SubstCache = (Listof (List Symbol FLANG)))

; define the const empty
(: empty-subst : SubstCache )
(define empty-subst null) ; null = empty arr

(: extend : Symbol FLANG SubstCache -> SubstCache)
(define (extend id expr sc)
	(cons (list id expr) sc))
(: lookup : Symbol SubstCache -> FLANG)
(define (lookup name sc)
	(cond [(null? sc)(error 'lookup "free instance ~s" name)]
		  [(eq? (first (first sc)) name)(second (first sc))]
		  [else (lookup name (rest sc))]))

(test (lookup 'x
	(extend 'y
		(Num 5)
			(extend 'x
				(Num 33)
					(extend 'foo
						(Fun 'x (Id 'x))
							(extend 'w (Num 5)
								empty-subst ))))))
=> (Num 33))
```

