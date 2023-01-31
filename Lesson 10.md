

```racket
(: arith-op : (Number Number -> Number) FLAMG FLANG -> FLANG)
(: Num->Number : FLANG -> Number ) ;help function
(define (Num -> Number arg)
	(cases arg 
	[(Num n) n] 
	[else (error 'Num->Number "expects a number, got: ~s" arg)]
))

(define (arith-op op arg1 arg2)
(Num (op (Num-Number arg1) (Num-Number arg2))))

(: eval : FLANG -> FLANG)
(define (eval exp)
	(cases exp
	[(Num n) exp]
	[(Add l r) (arith-op + (eval l) (eval r))]	
	[(Sub l r) (arith-op - (eval l) (eval r))]	
	[(Div l r) (arith-op / (eval l) (eval r))]
	[(Mul l r) (arith-op * (eval l) (eval r))]  
	[(With name named-expr body)
		(eval (subst body
					name
					(eval named-expr)))]
	[(Id name)(error 'eval "free instance: ~s" name) ]
	[(Fun name body) exp]
	[(Call (fun-expr  arg-expr)
	 (let ([fval (eval fun-expr)])
	 (cases fval
	 [(Fun name body) (eval (subst body
		 name
		 (eval arg-expr )))]))
	[else (error 'eval "expected a fucntion , got: ~s" )]))
 ]

(test (eval (Call  (Fun 'x (Mul (Id 'x) (Num 3))) (Num4)))
	=> (Num 12))
(test (eval (Call (With 'foo
				  (Fun 'x (Mul (Id 'x) (Num 3)))
				  (Id 'foo))
			(Num 4))) => (Num 12))


(:run : String -> Number)
(define (run code)
	(let ([res (eval (parse code))])
		(cases res
		[(Num n) n]
		[else (error 'run "evaluation returned a non-number: ~s" res)]
)))

(test (run "{with {sqr {fun {x} {* x x}}}}
			{+ {call sqr 5}
			   {call sqr 6}}}") => 61) ;5^2 +6^2 = 25 + 36 = 61
(test (run "{call {with {foo {fun {x} {* x 3*}}}
			foo}
			4}") => 12 )
(test (run "{call 4 5}") =error> "eval: expected  a function , got")

(test (run "{with {sqr {fun {x} {* x x*}}}
					sqr}") =error> "run: evaluation returned a non-number")





```


---
# Tirgul 10 

We use  `let` to have a nicer syntax in racket in place of lambda function .

exemple of code that do the same , once with `let` and the second with lambda function

```racket
;use of let 
(let ([x(+ 3 2)])(* x x))
```

```racket
;use of lambda function 
(lambda (x)(* x x))(+ 3 2)
```

In the same way we use in our language `with` and `Call` & `Fun` to make the syntax nicer to use ( like `let` and `lambda`)

**Exercise 1:** We need to implement `with` with `Fun` and `Call` in the parser .

```racket
(: parse-sexpr : Sexpr -> FLANG)
(define (parse-sexpr sexpr)
    (match sexpr
    [(number: n) (Num n)]
    [(symbol: name) (Id name)]
    [(cons 'with more)
        (match sexpr
            [(list 'with (list (symbol: name) named) body)
                (Call (Fun name (parse-sexpr body) ) (parse-sexpr named) )]
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
 We can see the change from [Lesson 9](/Lesson%209.md) , we changed the implemention of `with` by using the `Call` function.

![[Pasted image 20230131133829.png]]

**Exercise 2:** Do we need now to change actions in `eval` or `subst` function ?
 No , we can also remove the `With` constructor from `eval` function 

**Exercise 3:** we need to draw the tree of execution of the parsing on the expression : 
(run "{with {foo1 {fun {x} {* x y}}}
	{with {foo2 {fun {x} {call x 2}}}
		{with {y 4}
			{call foo2 foo1}}}}") 
			![[Pasted image 20230131144805.png]]

The answer : 
![[Pasted image 20230131144733.png]]

See **Exercise 4** in the Tirgul 10 in the slides .
