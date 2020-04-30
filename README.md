
# Interpreter for a simple Lisp, written in Prolog

Some online books show how to implement simple "Prolog" engines in
Lisp. These engines typically assume a representation of Prolog
programs that is convenient from a Lisp perspective, and can't even
parse a single proper Prolog term. Instead, they require you to
manually translate Prolog programs to Lisp forms that are no longer
valid Prolog syntax. With this approach, implementing a simple "Lisp"
in Prolog is even easier ("Lisp in Prolog in zero lines"): Manually
translate each Lisp function to a Prolog predicate with one additional
argument to hold the original function's return value. Done. This is
possible since a function is a special case of a relation, and
functional programming is a restricted form of logic programming.

Here is a bit beyond that: [**`lisprolog.pl`**](lisprolog.pl)

These 165 lines of Prolog code give you an interpreter for a simple
Lisp, *including* a parser to let you write Lisp code in its
natural&nbsp;form.

Internally, Prolog [**Definite Clause
Grammars**](https://www.metalevel.at/prolog/dcg) are used for parsing
Lisp&nbsp;code, and
[semicontext&nbsp;notation](https://www.metalevel.at/prolog/dcg#semicontext)
is used to <i>implicitly</i> thread through certain arguments. This
Prolog&nbsp;feature is very similar to Haskell's&nbsp;<i>monads</i>.

Read [**The Power of Prolog**](https://www.metalevel.at/prolog) for
more information about&nbsp;Prolog.


Sample queries, using Scryer Prolog:


Append:

<pre>
?- run("

    (defun append (x y)
      (if x
          (cons (car x) (append (cdr x) y))
        y))

    (append '(a b) '(3 4 5))

    ", V).
<b>   V = [append,[a,b,3,4,5]]</b>
</pre>

Fibonacci, naive version:


<pre>
?- time(run("

    (defun fib (n)
      (if (= 0 n)
          0
        (if (= 1 n)
            1
          (+ (fib (- n 1)) (fib (- n 2))))))
    (fib 15)

    ", V)).

<b>   % CPU time: 2.541 seconds
   V = [fib,610]</b>
</pre>

Fibonacci, accumulating version:

<pre>
?- time(run("

    (defun fib (n)
      (if (= 0 n) 0 (fib1 0 1 1 n)))

    (defun fib1 (f1 f2 i to)
      (if (= i to)
          f2
        (fib1 f2 (+ f1 f2) (+ i 1) to)))

    (fib 250)

    ", V)).

<b>   % CPU time: 0.221 seconds
   V = [fib,fib1,7896325826131730509282738943634332893686268675876375]</b>
</pre>


Fibonacci, iterative version:


<pre>
?- time(run("

    (defun fib (n)
      (setq f (cons 0 1))
      (setq i 0)
      (while (< i n)
        (setq f (cons (cdr f) (+ (car f) (cdr f))))
        (setq i (+ i 1)))
      (car f))

    (fib 350)

    ", V)).

<b>   % CPU time: 0.231 seconds
   V = [fib,6254449428820551641549772190170184190608177514674331726439961915653414425]</b>
</pre>


Higher-order programming and eval:

<pre>
?- run("

    (defun map (f xs)
      (if xs
          (cons (eval (list f (car xs))) (map f (cdr xs)))
        ()))

    (defun plus1 (x) (+ 1 x))

    (map 'plus1 '(1 2 3))

    ", V).
<b>   V = [map,plus1,[2,3,4]]</b>
</pre>

More information about this interpreter is available at:

[**https://www.metalevel.at/lisprolog/**](https://www.metalevel.at/lisprolog/)
