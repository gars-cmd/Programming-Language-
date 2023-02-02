we will add from what we have done in [Lesson 8](Lesson%208.md) the use of function , and change from now WAE with FLANG ( what we made before name WAE is now FLANG)

we want to implement this kind of operation :
	`{call {fun {x} {* x x}} 5 }*` that mean we wan to **call** the function **fun** with the parameter 5

or this kind of operation :
`with {sqrt {fun {x} {* x x}}} {+ {call sqr 5} {call sqr 6}}}*`

```racket
; Redefine the type

(define-type FLANG
	[Num Number]
	[Add FLANG FLANG ]
	[Sub FLANG FLANG ]
	[Mul FLANG FLANG ]
	[Div FLANG FLANG ]
	[Id Symbol ]
	[With Symbol FLANG FLANG ]
	[Fun Symbol FLANG ]
	[Call FLANG FLANG ]
)


```

we took what we did from [Lesson 6](Lesson%206.md) to implement call and fun 

```racket
(: parse-sexpr : Sexpr -> WAE)
(define (parse-sexpr sxp)
	(match sxp
		[(number: n) (Num n)]
		[(Symbol: name) (Id name)]
		[(cons 'with more)
		(match sexpr ;we check if the With input is sexpr
		(list 'with (list (symbol: name) named-expr) body )
			(With name ( parse-sexpr named-expr) (parse-sexpr body))])
		[else (error 'parse-sexpr "bad syntax in ~s" sxp)]]
		(match sexpr 
		[(list 'fun (list (symbol: name)) body)
		(Fun name body)]
		[else (error 'parse-sexpr "bad 'fun in ~s" sexpr )])]
		[(list 'call fun-expr arg-expr)
			(Call (parse-sexpr fun-expr) (parse-sexpr  arg-expr))]
		[(list '+ l r) (Add (parse-sexpr l) (parse-sexpr r))]
		[(list '- l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[(list '* l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[(list '/ l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[else (error 'parse-sexpr "bad syntax in ~s" sxp)]
))

(test (parse "{fun {x} x}") => (Fun 'x( Id 'x ))) ;1
(test (parse "{fun {x} {/ x 5}}") => (Fun 'x( (Div (Id 'x) (Num 5)))) ;2
(test (parse "{call {fun {x} {/ x 5}} 8}") => (Fun 'x( (Div (Id 'x) (Num 5))) (Num 8))) ;3
(test (parse "{with {sqr {fun {x} {* x x*}}} {+ {call sqr 5} {call sqr 6}}}") => (With 'sqr
	(Fun 'x (Mul (Id 'x) (Id 'x)))
	(Add (call (Id 'sqr) (Num 5))
		 (Call (Id 'sqr) (Num 6))))) ;4

```

Specification of subst :
N\[v/x] = N (i want to substitute all the occurrence of x by v in N , from  [Lesson 7](Lesson%207.md) 
(+ E1 E2) \[v/x] = (+E1 \[v/x] E2\[v/x])

N\[v/x] = N
		(+ E1 E2) \[v/x] = (+ E1\[v/x] E2\[v/x])
		(- E1 E2) \[v/x] = (- E1\[v/x] E2\[v/x])
		(* E1 E2) \[v/x] = (* E1\[v/x] E2\[v/x])
		(/ E1 E2) \[v/x] = (/ E1\[v/x] E2\[v/x])
		y\[v/x] = y 
		x\[v/x] = v
		{ with {y E1} E2} \[v/x] = {with {y E1 \[v/x]} E2\[v/x]}
		{ with {x E1} E2} \[v/x] = {with {x E1 \[v/x]} E2}
		{call E1 E2}\[v/x] = {call E1\[v/x] E2\[v/x]} 
		{fun {y} E}\[v/x] = {fun {y} E\[v/x]}
		{fun {x} E}\[v/x] = {fun {x} E}


The implementation goes like  :
```racket 
[(Call fun-expr arg-expr ) (Call (subst fun-expr from to)
								 (subst arg-expr from to))]

(test (subst (Fun 'x (Add (Id 'x) (Id 'y)))
'x (Num 4)) => (Fun 'x (Add (Id 'x) (Id 'y))) )

(test (subst (Fun 'x (Add (Id 'x) (Id 'y)))
'y (Num 4)) => (Fun 'x (Add (Id 'x) (Num 4))) ) ;because here y is free instance 

(test (subst (Call (Fun 'x (Div (Id 'x) (Id 'y))) (Add (Id 'x (Num 5)))
'x (Num 3)) => (Call Fun 'x (Div (Id 'x) (Id 'y)))) (Add (Num 3) (Num 5) ))))
```


---
# Tirgul 9

We define a new language FLANG , with the BNF  ,with two new operations `fun` and `call`: 

\<FLANG> ::= \<num>
		| { + \<FLANG> \<FLANG> }
		| { - \<FLANG> \<FLANG> }
		| { * \<FLANG> \<FLANG> }
		| { / \<FLANG> \<FLANG> }
		| { with { \<id> \<FLANG> } \<FLANG> }
		| \<id>
		| { fun { \<id> } \<FLANG> }
		| { call \<FLANG> \<FLANG> }

Then we add them to our type in racket : 
```racket
(define-type FLANG
	[Num Number]
	[Add FLANG FLANG]
	[Sub FLANG FLANG]
	[Mul FLANG FLANG]
	[Div FLANG FLANG]
	[Id Symbol]
	[With Symbol FLANG FLANG]
	[Fun Symbol FLANG]
	[Call FLANG FLANG])
```

Then we add `fun` and `call` to our parser 
```racket
(: parse-sexpr : Sexpr -> FLANG)
(define (parse-sexpr sexpr)
	(match sexpr
		[(number: n) (Num n)]
		[(symbol: name) (Id name)]
		[(cons 'with more)
			(match sexpr
				[(list 'with (list (symbol: name) named) body)
					(With name (parse-sexpr named) (parse-sexpr body))]
				[else (error 'parse-sexpr "bad `with' syntax in ~s" sexpr)])]
			[(cons 'fun more)
		(match sexpr
			[(list 'fun (list (symbol: name)) body)
				(Fun name (parse-sexpr body))]
			[else (error 'parse-sexpr "bad `fun' syntax in ~s" sexpr)])]
			[(list '+ lhs rhs) (Add (parse-sexpr lhs) (parse-sexpr rhs))]
			[(list '- lhs rhs) (Sub (parse-sexpr lhs) (parse-sexpr rhs))]
			[(list '* lhs rhs) (Mul (parse-sexpr lhs) (parse-sexpr rhs))]
			[(list '/ lhs rhs) (Div (parse-sexpr lhs) (parse-sexpr rhs))]
		    [(list 'call fun arg) (Call (parse-sexpr fun) (parse-sexpr arg))]
			[else (error 'parse-sexpr "bad syntax in ~s" sexpr)]))
   
```

```racket 
(: parse : String -> FLANG)
(define (parse str)
	(parse-sexpr (string->sexpr str)))
```


Exemple of decomposition of the parsing : 
![[Pasted image 20230130191400.png]]

Add new rules for subst function : 

N\[v/x]= N
{+ E1 E2}\[v/x]= {+ E1\[v/x] E2\[v/x]}
{- E1 E2}\[v/x]= {- E1\[v/x] E2\[v/x]}
{\* E1 E2}\[v/x]= {* E1\[v/x] E2\[v/x]}
{/ E1 E2}\[v/x]= {/ E1\[v/x] E2\[v/x]}
y\[v/x]= y
x\[v/x]= v
{with {y E1} E2}\[v/x]= {with {y E1\[v/x]} E2\[v/x]} ; if y =/= x
{with {x E1} E2}\[v/x]= {with {x E1\[v/x]} E2}
{call E1 E2}\[v/x]= {call E1\[v/x] E2\[v/x]}
{fun {y} E}\[v/x]= {fun {y} E\[v/x]} ; if y =/= x
{fun {x} E}\[v/x]= {fun {x} E}

**racket implementation** :

```racket
(: subst : FLANG Symbol FLANG -> FLANG)
(define (subst expr from to)
	(cases expr
		[(Num n) expr]
		[(Add l r) (Add (subst l from to) (subst r from to))]
		[(Sub l r) (Sub (subst l from to) (subst r from to))]
		[(Mul l r) (Mul (subst l from to) (subst r from to))]
		[(Div l r) (Div (subst l from to) (subst r from to))]
		[(Id name) (if (eq? name from) to expr)]
		[(With bound-id named-expr bound-body)​
			(With bound-id​
				(subst named-expr from to)​
				(if (eq? bound-id from) bound-body ​
					(subst bound-body from to)))]))​
		[(Call l r) (Call (subst l from to) (subst r from to))]
		[(Fun bound-id bound-body)
			(if (eq? bound-id from)
				expr
				(Fun bound-id (subst bound-body from to)))]))
```

Explanation from ChatGPT : 
> subst is a function that substitutes a symbol with another expression in a given language (FLANG). It does this by pattern matching on the input expression and handling each type of expression differently:
**If the input expression is (Num n)**, then it returns expr, which is the same expression and is not changed.
**If the input expression is (Add l r), (Sub l r), (Mul l r), or (Div l r)**, then it performs the substitution on the l and r sub-expressions and returns a new expression with the sub-expressions.
**If the input expression is (Id name)**, then it checks if name is equal to from. If so, it returns to, otherwise it returns expr.
**If the input expression is (With bound-id named-expr bound-body)** it bind a value named-expr to the identifier bound-id and then evaluate the expression bound-body in an environment where bound-id is associated with the value of named-expr. The subst function substitutes the expression to for the identifier from in the bound-body expression, but only if bound-id is not equal to from. If bound-id is equal to from, then the expression remains unchanged.
more details : The `With` case checks if the `bound-id` is equal to the `from` argument.
If it is, it returns the `bound-body` as is.
If not, it substitutes the `bound-body` by calling `subst` recursively with the same arguments (`bound-body`, `from`, `to`).
Finally, it returns a new `With` expression with the same `bound-id` and the `named-expr` that has been substituted with the `to` argument.
**If the input expression is (Fun bound-id bound-body)**, then it checks if bound-id is equal to from. If so, it returns expr, which is the same expression, as the symbol is bound in this context and cannot be substituted. Otherwise, it performs the substitution on the bound-body sub-expression and returns a new expression with the substituted sub-expression.


we improve now the eval function with another function  to evaluate operation:

```racket
(: arith-op : (Number Number -> Number) FLANG FLANG -> FLANG)
(define (arith-op op expr1 expr2)
	(: Num->number : FLANG -> Number)
	(define (Num->number e)
		(cases e
			[(Num n) n]
			[else (error 'arith-op "expects a number, got: ~s" e)]))
	(Num (op (Num->number expr1) (Num->number expr2))))
```


**eval rules with the new operations :** 
eval(N) = N
eval({+ E1 E2})= eval(E1) + eval(E2)
eval({- E1 E2})= eval(E1) - eval(E2)
eval({* E1 E2})= eval(E1) * eval(E2)
eval({/ E1 E2})= eval(E1) / eval(E2)
eval(id)= error!
eval({with {x E1} E2})= eval(E2\[eval(E1)/x])
eval(FUN)= FUN ; assuming FUN is a function expression
eval({call E1 E2})= eval(E1\[eval(E2)/x]) if eval(E1)={fun {x} E1}= error!
otherwise


implementation  in racket :
```racket
(: eval : FLANG -> FLANG)
(define (eval expr)
	(cases expr
		[(Num n) expr]
		[(Add l r) (arith-op + (eval l) (eval r))]
		[(Sub l r) (arith-op - (eval l) (eval r))]
		[(Mul l r) (arith-op * (eval l) (eval r))]
		[(Div l r) (arith-op / (eval l) (eval r))]
		[(With bound-id named-expr bound-body)
			(eval (subst bound-body bound-id (eval named-expr)))]
		[(Id name) (error 'eval "free identifier: ~s" name)]
		[(Fun bound-id bound-body) expr]
		[(Call fun-expr arg-expr)
			(let ([fval (eval fun-expr)])
				(cases fval
					[(Fun bound-id bound-body) (eval (subst bound-body bound-id (eval arg-expr)))]
					[else (error 'eval "`call' expects a function, got: ~s" fval)]))]))
```

We also add ta run function to execute it : 
```racket
(: run : String -> Number)
(define (run str)
	(let ([result (eval (parse str))])
		(cases result
			[(Num n) n]
			[else (error 'run "evaluation returned a non-number: ~s" result)])))

```

see in Tirgul 9 , nice exercice to practice decomposition exercice 2 . 
