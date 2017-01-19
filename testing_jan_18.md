# brief introduction to testing

## modeling the general problem
Definition: *Testing* is the practice of writing programs that aim to disprove the assertion that some software under test meets its specification. The specification reflects an idea that is implemented as the program.

- Stressing the difference between a program and its specification is critical. The specification may define the expected behavior of a program, but not its precise implementation (e.g., SBCL and CLISP vs ANSI Common Lisp).

- Therefore, tests must be designed in a way that supports this goal. We'll talk about how programmatic behavior may be tested at different levels of granularity and certain domain specific approaches used to achieve this end.

## example brief specification
- L-system renderer: Sierpinsky triangle program renders a graphic based on a context-free grammar specification and set of drawing rules (e.g., turtle graphics)
- The specification of such a program might indicate the following sequence:

1. Some user-defined grammar is read from the command-line interface.

    ```
    ... please specify your grammar:
    start? AB                   <enter>
    expansion rule 1? A -> B    <enter>
    expansion rule 1? B -> BA   <enter>
    rendering rule? turtle      <enter>
    ```

2. User is prompted for a level of iterations (bound not specified)

2. A graphic interface displays the result (show)

## imagining an implementation
- What doesn't the specification mandate?
- One possible implementation: Functional-paradigm implementation in Clojure, using the Quil library as a front-end to Processing to render to images in the browser
- Suppose we have an implementation: let's progress up a hierarchy of confidence that this implementation meets the requirements of the specification.

## unit test
- For object oriented programs, the smallest level of granuarity is the object; for functional programs, it is the function.
- As an aside, one might argue that this difference supports the argument that functional programs have the benefit of being easier to reason about: Functional programming aims to untie data from function, so the behavior covered by unit tests is less complex.
- For some function, unit testing involves assertions that some elements of a functions domain map to the expected elements in the codomain.

``` clojure
(defn my-reverse
  "given some sequence, returns the elements of the sequence read backward"
  [xs]
  (letfn [(aux [xs acc]
            (cond
              (empty? xs)
              acc
              :else
              (aux (rest xs) (cons (first xs) acc))))]
    (aux xs '())))
```
- Here, we can think of the docstring is a mini-specification. A unit test is necessary to confirm that my-reverse meets this requirement.

``` clojure
(deftest my-reverse-test
  (let [observed (my-reverse '(a b c d e))
        expected '(e d c b a)]
    (testing "my-reverse returns the reverse of a seq"
    (is (= observed expected)))))
```

- Using the case of the L-system rendering program, a unit test might involve validating some function

``` clojure
(defn expand-term
  "
  outputs a vector specifying the lsystem after n iterations
    g -> grammar
    n -> number of iterations
  "
  [g n]
  {:pre [(map? g) (integer? n) (> n 0)]}
    (letfn [(aux [g n]
              (cond
                (zero? n) (:start g)
                :else (postwalk-replace (:rules g) (aux g (dec n)))))]
      (flatten (aux g n))))

[...]

(deftest test-expand-term
  (testing "expand-term expands from starting point recursively given a grammar and number of iterations")
    (let [grammar {:variables [:A :B]
                   :constants [:+]
                   :start :A
                   :rules {:A [:A :+ :B]
                           :B [:A]}}
          iterations 3 ;; arbitrarily chosen
          expected [:A :+ :B :+ :A :+ :A :+ :B]
          observed (expand-term grammar iterations)]
    (is (= expected observed))))
```

- tests running successfully confirm that for the probed elements of the domain, the function yields the expected result.
- However, we're left unsure if these results may be extrapolated to the entire domain of the function. For this reason, we default to edge cases (as commonly done in Physics)

## higher levels of granularity
- integration testing: do the different modules of the program work together as expected? Integrations account for unexpected emergent properties and failure to put parts together properly.
- end-to-end testing: does the sum of the parts of the software under test perform in a way that meets the specification?

## Post-hoc approaches
- Inevitably, bugs are found that must be addressed; To this end, *regression tests* provide a mechanism for ensuring that emerging problems don't recur

## Towards a more robust testing
- *Fuzzing* is a set of techniques concerned with drawing infererences about a programs behavior in reponse to random input.


- *Property-based testing*: generate instances of the domain element that are fed to the function. Rather than confirming a mapping to some element of the domain, some relation between the domain and codomain is tested. In other words, the program searches for anomolies to the assertion that some relation holds up between the domain and codomain.

- An simple example:

``` clojure
(def sort-idempotent-prop
  (prop/for-all [v (gen/vector gen/int)]
    (= (sort v) (sort (sort v)))))

(tc/quick-check 100 sort-idempotent-prop)
```
- critical point: sample entire domain of function; best done in conjunction with unit tests, since the probability of hitting all branches might be extremely low.

## resources: TODO
