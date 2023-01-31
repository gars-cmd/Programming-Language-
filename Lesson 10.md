

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