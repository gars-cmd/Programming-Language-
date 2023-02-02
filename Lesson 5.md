AE -> Number 
\<AE> := \<num> 1
			| { +\<AE> \<AE>} 2 
			| { -\<AE> \<AE>} 3  
			| { \*\<AE> \<AE>} 4  
			| { \/\<AE> \<AE>} 5  

1. eval ('\<num>') := \<num>
2. eval ('E1'+'E2') -> eval ("E1") + eval("E2")
3. eval ('E1'-'E2') -> eval ("E1") - eval("E2")
4. eval ("1 -2+3") -> eval ("1") - eval("2+3") -> eval ("1") - ( eval ("2") + eval ("3") )


1.\<num> := \<digit>
			2.	| \<digit>\<num> |
\<digit> := 1 | 2 | 3 | 4.... | 10



![Image](https://github.com/gars-cmd/Programming-Language-/blob/main/Pasted%20image%20221122162908.png)
)


```rkt

(: eval-sexpr : Sexpr -> Number)
(define (eval-sexpr sxp)
	(match sxp
		[(number: n) n)]
		[(list '+ l r) (+ (eval-sexpr l) (eval-sexpr r))]
		[(list '- l r) (- (eval-sexpr l) (eval-sexpr r))]
		[else (error 'eval-sexpr "bad syntax in ~s" sxp)]
))

(: eval : AE -> Number )
(define (eval exp)
	(eval-sexpr (string -> sexpr code)))



; help function 
(: run : String -> Number)
(define (run code)
(eval (parse code)))


(: eval : AE -> Number )
(define (eval exp)
	(cases exp 
	[ (Num n) n]
	[ (Add l r) (+ (eval l) (eval r) )]
	[ (Sub l r) (- (eval l) (eval r) )]
	[ (Mul l r) (* (eval l) (eval r) )]
	[ (Div l r) (/ (eval l) (eval r) )]
	))


```

---
# Tirgul 5


The function `match` return the value defined for matching a pattern 

(match value 
	\[pattern1 result-expr1 ]
	\[pattern2 result-expr2 ]
	 ...)
ex : 

```racket
(match ‘(1 2 3)
	[(number: n) n]
	[(list x y z) (+ x y z)]
	[(list x y z) (* x y z)]
	[else -1])
=> 6
```

exemple to sum the number of in a list :

```racket
(: sum : ListOf Number) -> Number)
(define (sum lst)
	(match lst
		['() 0]
		[(cons f r) (+ f (sum r))]))
```

**exemples of match :** 

```racket
#|
In this case id is a variable name then it will match any value , then the value will always match and the result-expr will be always  evaluated.
|# 

(match value
	[ id result-expr])
``
#|
In this case the "_" represent a wildcard or catch all pattern , then the result-expr  will be always evaluated
|#

(match value
	[ _ result-expr])

;Match every number  and bind the val to n

(match value
	[(number : n) result-expr])

;Match every symbol  and bind the val to s

(match value
	[(symbol  : s) result-expr])

;match every string and bind the val to s

(match value
	[(string  : s) result-expr])

;list pattern for match 

(match ‘((1) (2) 3)
	[(list (list x) (list y) z) (+ x y z))])

#|
The pattern (list (number: n) …) matches a list that starts with a number and possibly continues with more elements. The number: syntax is used to bind the first element of the list to the variable n. The … syntax is used to match any number of additional elements in the list.
|#

(match ‘(1, 2, 3, 4)
[(list (number: n) …) n])

#|
The pattern (list (list x y) …) matches a list that consists of sublists, where each sublist contains two elements , he variables x and y are used to match the first and second elements of each sublist, respectively. The … syntax is used to match any number of additional sublists. This expression uses the append function to concatenate the lists x and y, resulting in a single list of all the first elements followed by all the second elements
|#

(match '((1 2) (3 4) (5 6) (7 8))
[list (list x y) ...) (append x y)])
```


Another exemple :  reverse a list 
```racket 
	(:revr : ListOf Any)-> (ListOf Any))
	(define  (revr ls)
		(match ls
			[(list) ls]
			[(list _) ls]
			[(list bs...a)(cons (a (revr bs))]))
```

```racket
(: calc-list : (ListOf(U Number Symbol )) -> Number )
(define (calc-list ls )
	(: help : (ListOf (U Number Symbol )) Symbol -> Number)
	 (define help ls s)
		 (match ls
			['() 0]
			[(list (number : n) x...)
				(match s
					['+ (+ (help x s) n) ]
					['- (- (help x s) n)])]
			[(list (symbol: op) x ...) (help x op)]))
	(help ls '+))
				
  
```


Use of the word `let` to define a variable locally 

```racket
(let ([id1 exp1] [id2 exp2].. [idn expn])
	body_using id1,id2 and idn )

;ex : 
(let ([x 2][y 3]) (* x y))
; =>6

(let ([x 4]) (* x x))
; =>16

(let ([x 1] [y 2] [z 3]) (+ x y z))
; => 6
```
