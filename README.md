
## Interpreter for a simple Lisp, written in Prolog

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

Here is a bit beyond that: [**lisprolog.pl**](lisprolog.pl).

These 160 lines of Prolog code give you an interpreter for a simple
Lisp, *including* a parser to let you write Lisp code in its
natural&nbsp;form.

Sample queries, using SWI-Prolog:


Append:

    ?- run("

        (defun append (x y)
          (if x
              (cons (car x) (append (cdr x) y))
            y))

        (append '(a b) '(3 4 5))

        ", V).
    %@ V = [append, [a, b, 3, 4, 5]].


Fibonacci, naive version:

    ?- time(run("

        (defun fib (n)
          (if (= 0 n)
              0
            (if (= 1 n)
                1
              (+ (fib (- n 1)) (fib (- n 2))))))
        (fib 24)

        ", V)).
    %@ % 12,857,193 inferences, 2.724 CPU
    %@ V = [fib, 46368].


Fibonacci, accumulating version:


    ?- time(run("

        (defun fib (n)
          (if (= 0 n) 0 (fib1 0 1 1 n)))

        (defun fib1 (f1 f2 i to)
          (if (= i to)
              f2
            (fib1 f2 (+ f1 f2) (+ i 1) to)))

        (fib 250)

        ", V)).
    %@ % 55,773 inferences, 0.018 CPU
    %@ V = [fib, fib1, 7896325826131730509282738943634332893686268675876375].


Fibonacci, iterative version:


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
    %@ % 48,749 inferences, 0.012 CPU
    %@ V = [fib, 6254449428820551641549772190170184190608177514674331726439961915653414425].


Higher-order programming and eval:

    ?- run("

        (defun map (f xs)
          (if xs
              (cons (eval (list f (car xs))) (map f (cdr xs)))
            ()))

        (defun plus1 (x) (+ 1 x))

        (map 'plus1 '(1 2 3))

        ", V).
    %@ V = [map, plus1, [2, 3, 4]].
