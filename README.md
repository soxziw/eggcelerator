# Eggcelerator: cracking cryptographic speed with e graphs

Given a cost model, we create a Domain Specific Language (DSL) and rewrite rules to find the cheapest logically equivalent form of an expression via algebraic rewrites 

---
### Prerequisites

1. Download [Rust](https://www.rust-lang.org/tools/install)
2. Build all dependencies
```bash
apt install llvm clang libclang-dev
cd src
cargo build
```

### Tutorial
TL;DR Quick Command for B1:
```bash
echo -e "(+ (* a0 a0) (* (* a1 a1) x1))\n(* (* a0 2) a1)\ndone" | cargo run ../rules/rules_fp2.txt ../costs/costs_fp2.txt
```
1. Run the program 
```bash
cargo run LocationOfRules* LocationOfCostModel*
```
*These fields are optional and will be assigned to an fp2 default if not inputted

Output:
```
Rule 'add-zero' is valid.
Rule 'zero-add' is valid.
Rule 'mul-one' is valid.
Rule 'one-mul' is valid.
Rule 'mul-zero' is valid.
Rule 'zero-mul' is valid.
Rule 'sub-zero' is valid.
Rule 'sub-self' is valid.
Rule 'sub-to-add' is valid.
Rule 'comm-add' is valid.
Rule 'comm-mul' is valid.
Rule 'assoc-add' is valid.
Rule 'assoc-mul' is valid.
Rule 'distribute' is valid.
Rule 'factor' is valid.
Rule 'square-def' is valid.
Rule 'square-mul' is valid.
Rule '2ab-forward' is valid.
Rule '2ab-backward' is valid.
Rule 'karatsuba' is valid.
Finite Field Expression Optimizer
=================================
Enter an expression to optimize (in S-expression format):
formatted:
```
2. Write your S-expressions and when finished type "done"
```
(+ (* a0 a0) (* (* a1 a1) x1))
```

```
(* (* a0 2) a1)
```
```
done
```
3. You will now see the optimized expressions as well as their costs
```
Optimization Results:
---------------------
Original expression 0:
(+
  (* a0 a0)
  (*
    (* a1 a1)
    x1))
Optimized expression 0:
(+
  (^2 a0)
  (*
    (^2 a1)
    x1))
Original expression 1:
(*
  (* a0 2)
  a1)
Optimized expression 1:
(*
  a1
  (+ a0 a0))
Original total cost: 45
Optimized cost: 34
Improvement: 24.44%

E-Graph Statistics:
Iterations: 30
Total e-graph nodes: 9264
E-classes: 1719
```


### Branch-wise details
Eirik:
TL;DR
`git checkout master`

`echo -e "(+ (* ?a0 ?b0) (* ?a1 ?b1))\n(- (* (+ ?a0 ?a1) (+ ?b0 ?b1)) (+ (* ?a0 ?b0) (* ?a1 ?b1)))\ndone" | cargo run rules_fp0.txt costs_fp0.txt`
This example shows how eggcelerator works on Benchmark 1. It demonstrates the Karatsuba multiplication rule, but in this case, no improvement was found.

`echo -e "(+ (* ?a0 ?a0) (* (* ?a1 ?a1) xi))\n(* 2 (* ?a0 ?a1))\ndone" | cargo run rules_fp4.txt costs_fp2.txt`
This example shows how eggcelerator works on Benchmark 2. It demonstrates optimization of elliptic curve addition, reducing the cost of operations by using squaring instead of multiplication and utilizing more efficient constant multiplication patterns. According to the costs defined in costs_fp2.txt (add=1, sub=1, const_mul=4, square=6, mul=10, inv=80), the optimization achieves a 24.44% improvement, reducing the total cost from 45 to 34.

`echo -e "(+ (* x0 y0) (* xi (+ (* x1 y2) (* x2 y1))))\n(+ (+ (* x0 y1) (* x1 y0)) (* xi (* x2 y2)))\n(+ (+ (* x0 y2) (* x1 y1)) (* x2 y0))\ndone" | cargo run rules_fp6.txt costs_fp2.txt`

This example shows how eggcelerator works on Benchmark 3, which involves optimization of Fp6 multiplication. It demonstrates the application of Karatsuba multiplication technique to complex number arithmetic in finite field extensions. The input expressions represent the components of Fp6 multiplication, but in this case, no cost improvement was found because the implemented rules already represent an optimal strategy for the given cost model. Furthermore, eggcelerator processes each expression independently, with no knowledge transfer between subformulas, which limits potential optimizations across the complete Fp6 multiplication algorithm. This benchmark highlights eggcelerator's ability to handle multi-variable expressions and complex algebraic structures, while also showing a limitation in optimizing compound operations.

Richard: 
TL;DR
`git checkout richard`
`echo -e "(** (Fp6 g0 g1 g2 h0 h1 h2) (+ (- (** (Fp6 g0 g1 g2 h0 h1 h2) 4) (** (Fp6 g0 g1 g2 h0 h1 h2) 2)) 1))\ndone" | cargo run ../rules/rules_fp6.txt ../costs/costs_fp6.txt`

This branch focuses on Benchmark 6 with algorithm 29, 30, and 31 and section 3.2 of the BMO paper. The rewrite rules are in "../rules/rules_fp6.txt"
It incorporates the Fp6 DSL which is represented as Fp6 g0 g1 g2 h0 h1 h2. Caveats include that any exponentiation has a non-symbolic cost (aka just an integer value) so the cost of p^100 = p^p. 
