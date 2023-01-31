
How to define a constant  in racket: 
```racket
(: pi : Number)
(define pi 3.14)
```

The `assq` function exist in racket and do a part of the lookup function  from [[Lesson 11]]  
```racket 
(assq 'a '((1 2) (3 4) (a b) (7 8)))
; return '(a b)
```

Then we can now modify the function lookup as follow 
```racket
(: lookup : Symbol SubstCache -> FLANG)
(define (lookup name sc)
let ([cell (assq name sc)])
(if cell 
(second cell)
(error 'lookup "free instance ~s" name))))
```

Now we will change some of the specification of eval from [[Lesson 11]] 
7.eval ({with {x E1} E2}, sc) = eval(E2, extend(x, eval(E1, sc), sc))
9.eval ({call E1 E2}, sc ) = if {fun {x} E} <-- eval(E1, sc)
												eval (E, extend(x, eval(E2, sc),sc))
												otherwise: error!


Now we will implement it : 
```racket

(define (eval exp sc)
(cases exp
	[(Num n) exp]
	[(Add l r) (arith-op + (eval l sc) (eval r sc))]	
	[(Sub l r) (arith-op - (eval l sc) (eval r sc))]	
	[(Div l r) (arith-op / (eval l sc) (eval r sc))]
	[(Mul l r) (arith-op * (eval l sc) (eval r sc))]  
	[(With name named-expr body)
		(eval body (extend name 
						(eval named-expr sc)
						sc))]
	[(Id name)(lookup  name sc)]
	[(Fun name body) exp]
	[(Call (fun-expr  arg-expr)
	 (let ([fval (eval fun-expr sc)])
		 (cases fval
			 [(Fun name body) (eval body (extend named-expr (eval arg-expr ) sc))]
	[else (error 'eval "expected a fucntion , got: ~s" fval )]))
```



