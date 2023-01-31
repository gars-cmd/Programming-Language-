###  Comments 

comment in racket is `;;` 
exemple : `;; this is a commment`

comment one multiples lines : 

```rkt
#|
first comment line
second comment  line
|#
```

### Arrays

- if we want to declare an array :

`list( 1 -4 7 8 9)`

or 

`cons 7 (list -9 8 7 10 2)` that give us the list : [7, -9, 8, 7, 10, 2]
in the same way we add an element at the beginning of the list.

- if we want to create an array of one element we create a pair with only one element .
`(cons 1 null)`

another exemple :
`(cons 'a (cons 2 null)` we first create the pair (2, null) that equal to the list [2] then we concatenate it with the second pair (a,(2, null)) -> [a, 2]

>To declare a list with char we need to declare them before (as explained in [[lesson 1]] ) as follow:
`(list 'a 'b 'c 5 6 7'`

- To append en element  to an array  we use the word `append` as follow .

```rkt
(append (list 3 4 5) (list 6 7 8))
```
we get from the command above the list : [3, 4, 5, 6, 7, 8,]

- To get a specific element of an array :
```rkt
(first (list 2 3 6 'a)    ;return the first element 
(second (list 2 3 6 'a)   ;return the second element 
(third (list 2 3 6 'a)    ;return the third element 
...
(thenth (list 1 2 3 ))    ;return the tenth element 

; other alternative

(list-ref '(1 2 3) 2)     ; return the element in position 2 (3)
(list-ref '(1 2 3) 1)     ; return the element in position 1 (2)





```

- To get all the element  except the first :
`(rest (list 2 3 4))` return [3, 4]


- To get the length of a list ;
`(length '(1 2 3 4 5 6)` we get from this command 6

- To define a value to a indicator we can do as follow :
`(define PI 3.14` 
once we define it we can do operations on it (on the indicator ).

- **To write a function** we also use the keyword define but with another pair of brackets .
```rkt
(define (double x))
	(list x x)
```
**all the function must return a value** 

another exemple of function:
```rkt
; : function_name : type_input : type_output 
(: f : Number -> Number)
(define (f x)
	(* x(+ x 1)))
```
In the line 1 we define the type of input and output (no must but protect us from unforeseen cases).


another exemple of function:
```rkt
(: circle_area : Number : Number)
(define (circle_area r)
	(* PI r r))
(define PI 3.14)
```


- **if statement**
the if statement  follow the scheme : (if condition yes-statement no-statement):
```rkt
(if (> 5 4) "greater" "smaller")
```

other options to write a condition better for multiple cases of condition :

```racket
(: digits : Number-> Number)
(define (digits-num x)
(cond [(<= x 9) 1])
	  [(<= x 99) 2])
	  [(<= x 999) 3])
      [(<= x 9999) 4])
	  [else 99999]))
```

```rkt
factorial function :
( : factorial :Number -> Number)
(define factorial  x)
	(if (=0 x)
	1
	(* x (factorial (- x 1)))
```

---
# Tirgul

***Empty list :*** () or null

#### use of cons (recursive pair)
```racket
;(cons <first elem> <second elem>)
(cons 1 ( cons 2 null))
(cons 1 ( cons 2'()))

```

#### use of list
```racket
(list <first elem> <second elem>... <n elem>)
(list 1 2) ; "12"
```

#### Check if an object is a list
```racket
(list? <object>)
(list? '(1 2)) ; => #t
(list? (cons 1 (cons 2 '()))) ;v => #t 
(list? (cons 1 null)) ; => #t
```

#### Check if an object is a pair
```racket
(pair? 1) ;=> #f
(pair? (cons 1 (cons 2 '() ))) ;=> #t (cons 1'(2))
(pair? (cons 1 (cons 2 (cons 3 '() )))) ;=> #t (cons 1'(2 3))
(pair? ()) ;=> #f 
(pair? '(1)) ;=> #t (cons 1 '())
(pair? '('())) ;=> #t (cons '() null)
```

#### create a list 
```racket
; with cons 
(cons 1 (cons 2 (cons 3 null))) ;=> '(1 2 3)

;with list 
(list 1 2 3) ;=> '(1 2 3)
(list 1 2 3 null) ;=> '(1 2 3 ())

```

#### first & rest 
```
;first return the first element of the list
(first '(1 2 3)) ; => 1
(first '((1 2) 3)) ; => (1 2)

;rest return all elements of a list or pair except the first 

(rest '(1 2 3)) ; => (2 3)
(rest '((1 2) 3)) ; => (3)
```

#### Get the length of a list 
```racket
(: length : (Listof Any ) -> Natural)
(define (length lst)
  (if (null? lst)
      0
      (+ 1 (length (rest lst)))))

;a built-in option keyword "length"
(length '(1 2 3 4 5)) 

```