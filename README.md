
## Interpreter for a simple Lisp, written in Prolog.


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
