**Formal specification of eval :**11
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


---
# Tirgul 11 
- Static scope means that a variable's value and location is determined by its position in the code, before the program runs.
- Dynamic scope means that a variable's value and location is determined by the call stack during runtime, i.e. based on the sequence of function calls.

exemple : 
```racket
(define fact
	(lambda (n)
		(if (zero? n) 1 (* n (fact (- n 1))))))

(let ([* +])
	(fact 5))
```

If we are in a dynamic scope we will bind the `*` to be `+` then it will add the result of the factorization (5+4+3+2+1) .
If  we are in a static scope the * is not binded to + into the recursive call of fact then the result will be a regular factorization  (5\*4\*3\*2\*1)

explanation with more complex exemple in the slide of tirgul 11 . 

Like what we saw in the lesson we will extend the language to handle nested with  . 

```racket
;; a type for substitution caches
(define-type SubstCache = (Listof (List Symbol FLANG)))

(: empty-subst : SubstCache)
(define empty-subst null)

(: extend : Symbol FLANG SubstCache -> SubstCache)
(define (extend name val sc)
	(cons (list name val) sc))

(: lookup : Symbol SubstCache -> FLANG)
(define (lookup name sc)
	(let ([cell (assq name sc)])
		(if cell
			(second cell)
			(error 'lookup "no binding for ~s" name))))
```

Here the implementation of eval function with those new function :
```racket
(: eval : FLANG SubstCache -> FLANG)
(define (eval expr sc)
	(cases expr
		[(Num n) expr]
		[(Add l r) (arith-op + (eval l sc) (eval r sc))]
		[(Sub l r) (arith-op - (eval l sc) (eval r sc))]
		[(Mul l r) (arith-op * (eval l sc) (eval r sc))]
		[(Div l r) (arith-op / (eval l sc) (eval r sc))]
		[(With bound-id named-expr bound-body)
		(eval bound-body
			(extend bound-id (eval named-expr sc) sc))]
		[(Id name) (lookup name sc)]
		[(Fun bound-id bound-body) expr]
		[(Call fun-expr arg-expr)
			(let ([fval (eval fun-expr sc)])
				(cases fval
					[(Fun bound-id bound-body)
						(eval bound-body
							(extend bound-id (eval arg-expr sc) sc))]
					[else (error 'eval "`call' expects a function, got: ~s" fval)]))]))
```

Exercise : given the following expression show how the `eval` function work with the model of exchanges and with the model of SubstCache . 
```racket 
(run "{with {foo {fun {x} {* x x}}}
              {call {with {foo {fun {y} {- y 1}}}
                           {fun {x} {call foo x}}} 
				4}}”)
```
See the slide to see how to run with the model of exchange : 

For the model of SubstCache : 
1. The first step is to evaluate the expression `{with {foo {fun {x} {* x x}}} {call {with {foo {fun {y} {- y 1}}} {fun {x} {call foo x}}} 4}}`. The expression is a With expression, so the With clause is matched.
2. The With clause takes two sub-expressions, `bound-id` and `bound-body`. `bound-id` is `foo` and `bound-body` is `{call {with {foo {fun {y} {- y 1}}} {fun {x} {call foo x}}} 4}`.
3. The `named-expr` is `{fun {x} {* x x}}`. This is evaluated to a Fun expression and the substitution cache is extended with `bound-id` `foo` and value `Fun {x} {* x x}` using the extend function. The substitution cache now becomes (extend 'foo (Fun {x} {* x x}) empty-subst).
4. The `bound-body` expression `{call {with {foo {fun {y} {- y 1}}} {fun {x} {call foo x}}} 4}` is now evaluated using the updated substitution cache. This expression is a Call expression, so the Call clause is matched.
5. The `Call` clause takes two sub-expressions, `fun-expr` and `arg-expr`. `fun-expr` is `{with {foo {fun {y} {- y 1}}} {fun {x} {call foo x}}}` and `arg-expr` is 4.
6. The `fun-expr` is evaluated first, which is a `With` expression. The `With` clause is matched and the `named-expr` is `{fun {y} {- y 1}}` and the `bound-body` is `{fun {x} {call foo x}}`.
7. The `named-expr` is evaluated to a `Fun` expression and the substitution cache is extended

## need the rest 
