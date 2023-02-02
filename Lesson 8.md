
example from [Lesson 7](Lesson%207.md)  :

```racket
(eval (With 'x (Add (Num 5) (Num 3)) (* (Id 'x) (Id 'x')) ))`

#| eval ({With {x E1} E2})

1. v<- eval(E1)            => v<- eval ((Add (Num 5) (Num 3))
2. E2' <- subst (E2, x, v) => E2' <- subst ((Mul (Id 'x) (Id 'x')) 'x (Num v)
3. eval (E2')
```


let's now write the implementation in racket ;
we will use the subst function from [Lesson 7](Lesson%207.md) 

```racket
(: run : String -> Number)
(define (run code)
(eval (parse code))
))

(:eval : WAE -> Number)
(define (eval expr)
(cases expr
[(Add l r) (+ (eval l) (eval r))]
[(Sub l r) (- (eval l) (eval r))]
[(Mul l r) (* (eval l) (eval r))]
[(Div l r) (/ (eval l) (eval r))]
[(Add l r) (+ (eval l) (eval r))]
[(With name named-expr body)
	(eval (subst body name (Num (eval named-expr ))))]
[(Id name) (error 'eval "free identifier")]
))
(test (run "5") => 5)
(test (run "{+ 9 2}")=>11)
(test (run "{with {x {+ 8 7}} {*x 2}}")=>30)
(test (run "{with {x 15} {*x 2}}")=>30)
(test (run "{with {x 5} {with {y {- x 3} {* y y} }}")=>4)
(test (run "{with {x 5} {with {x {+ x 1} {* x x} }}")=>36)
(test (run "{with {x 1} y}")=> error "free identifier")
))
```

#### Anonymous function in racket :

key-word : **lambda**
exemple: 
`(( lambda (x y z) (+ x y z)) 5 4 6) `

implementation with with :
```racket
{with {f {fun {x}
{* x x}}} {+ {call f 2} {call f 3}}}
```


reasons to use lambda function :
	1. multiple code 
	2. modularity 


---
# Tirgul 8 

We will work with the language of the registers 
- Every expression of the language is of the form : {reg-len = len A}
	len = Natural 
	A = list of actions on registers 

- Every list of  0 and 1 wrapped with {} is legal for actions on registers  ex: {0 1 1}
- If A and B are actions on registers , then the expression  from the use of A and B with the operand `and` and `or` and the global expression is wrapped into {} , is legal for actions on registers  , also work with `shl` (shift left).
		ex :  	{and {1 1 0} {0 0 0}} {or {1 1 0} {0 0 0}} {shl {1 1 0}}

- We can also use `with` like in WAE , ex: {with {x {0 1 0}} {and x x}}

exemples of legal operations in the registers language :
{reg-len = 4 {1 0 0 0}}
{reg-len = 4 {shl {1 0 0 0}}}
{reg-len = 4 {and {shl {1 0 1 0}} {shl {1 0 1 0}}}}
{reg-len = 4 {or {and {shl {1 0 1 0}} {shl {1 0 1 0}}} {1 0 1 0}}}
{reg-len = 2 {with {x {and {1 0} {1 1}}} {or x x}}}


We write a parser for the language ROL :
\<ROL> ::= {reg-len = \<num> \<RegE> }
\<RegE> ::=
			{\<Bits>}       
                     |{and \<RegE> \<RegE>}        
					   |{or \<RegE> \<RegE>}        
	                 |{shl \<RegE>}        
	                 |{with {\<ID> \<RegE>} \<RegE> }          
	                 |\<ID> 
 \<Bits> ::= \<bit>         
                        |\<bit>  \<Bits>

 \<bit>::= 1 | 0  

in racket code : 
```
(define-type BIT = (U 1 0))
(define-type Bit-List = (Listof BIT))

(define-type RegEx
	[Reg Bit-List ]
	[And RegE RegE]
	[Or RegE RegE]
	[Shl RegE ]
	[With Symbol RegE RegE]
	[Id Symbol])
```

Then we will write a parser for this : 
```racket
(: list->bit-list : (Listof Any) -> Bit-List)
	(define (list->bit-list lst)
		(cond 
			[(null? lst) null]
			[(eq? (first lst) 1)
				(cons 1 (list->bit-list (rest lst)))]
			[else (cons 0 (list->bit-list (rest lst)))]))
```

Then the parse-sexpr
```
(: parse-sexpr : Sexpr -> RegE)
(define (parse-sexpr sexpr)
	(match sexpr
		[(list 'reg-len ‘=   (number: n)   args)
 			(if (> n 0)
       				(parse-sexpr-RegL args n)
    				(error 'parse-sexpr "Register length must be at least 1 ~s" sexpr) )]
		[else (error 'parse-sexpr "bad syntax in ~s" sexpr)]))
```

```racket
(: parse-sexpr-RegL : Sexpr Number -> RegE)
(define (parse-sexpr-RegL sexpr len)
	(match sexpr
	[(list (and a (or 1 0)) ... ) 
		(if (= len (length a))
			 (Reg (list->bit-list a))
		 	(error 'parse-sexpr-RegE "wrong number of bits in ~s" a)) ]
	
	[(list 'and list1 list2) (And (parse-sexpr-RegL list1 len) (parse-sexpr-RegL list2 len))]
	[(list 'or list1 list2) 	 (Or (parse-sexpr-RegL list1 len) (parse-sexpr-RegL list2 len))]
	[(list 'shl list)		 (Shl (parse-sexpr-RegL list len))]
	[(symbol: id-name) (Id id-name)]  
 
	[(cons 'with args)
	 	(match sexpr
     		[(list 'with (list (symbol: name) named) body)
     			 (With name (parse-sexpr-RegL named len) (parse-sexpr-RegL body len))]
   	        [else (error 'parse-sexpr-RegE "bad `with' syntax in ~s" sexpr)])]
[else (error 'parse-sexpr "bad syntax in ~s" sexpr)]))

```

**explanation**  `(list (and a (or 1 0)) ... )` : 
The syntax `(and a (or 1 0)) ...)` in the first case is a pattern that matches a list whose first element is either 1 or 0, followed by a variable number of elements with the same type. The `and` operator matches a value that satisfies both of its arguments: a is a pattern variable that matches any value and the `(or 1 0)` matches either 1 or 0. The ... syntax is a shorthand for a sequence of elements with the same pattern, which in this case is the pattern `(and a (or 1 0))`.
