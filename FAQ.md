# Frequently Asked Qustions

## My program is correct but Dafny says there is a problem! What gives?

Dafny sometimes reports an error, *even on a correct program*. (See the discussion of Dafny's incompleteness below for more about why this is.)

Of course, you should also think carefully to convince yourself that you really are running into Dafny's incompleteness. Even expert users of Dafny sometimes write programs that look correct but actually contain subtle bugs. 

Dafny is incomplete for (at least) the following reasons:

* **Missing annotations**. Dafny breaks the verification problem down using pre-/postconditions and loop invariants. A program can be globally correct, but, if it contains insufficient or incorrect annotations, Dafny may report an error. See the [Guide](https://rise4fun.com/Dafny/tutorial/Guide) section "Assertions" for further explanation.
* **Incomplete quantifiers**. Quantifiers (`forall` and `exists`) are one source of undecidability in the logic underlying Dafny. In principle, any program using quantifiers may run into this undecidability, though Dafny handles many cases automatically. See the section below on quantifiers and triggers for more on how to work with quantifiers in Dafny.
* **Nonlinear arithmetic**. Numerical formulas involving multiplication are also undecidable in general. (Multiplication by any constant value such as `2 * x` are fine though.) You can sometimes work around this by breaking your formula down into smaller pieces by introducing new functions. You can also turn off nonlinear arithmetic in the solver using `/noNLarith` command line flag. See `dafny /help` for more details.
* **Incomplete axiomatization of collection operations and extensionality**. Dafny contains several built-in types (sequences, sets, maps, etc.) and operations over those types, all of which are axiomatized to the underlying solver. These axioms are known to be incomplete. One common problem is with "extensionality", which refers to the fact that two collections are equal when they have the same elements. Dafny has trouble knowing when to invoke this principle, but you can encourage it to do so by adding an explicit equality assertion to your program.
* **Complex heap reasoning**. Under the hood, Dafny reasons about the heaps using collections and quantifiers. Although this reasoning is carefully tuned to work well in the common case, it can occasionally "leak" incompleteness due to underlying incompleteness from quantifiers or collections. 

## What guarantees does Dafny make about programs that pass the verifier? What about programs that don't pass?

In an ideal world, Dafny would report errors for exactly those programs that are incorrect. Unfortunately, checking whether a program meets its specification is undecidable in general. As a result, Dafny sometimes reports verification-time errors on programs are correct on every run-time execution. **This is not a bug in Dafny, but a fundamental design decision in response to undecidability.** In general, the phenomenon of reporting errors even on correct programs is called "incompleteness".

On the other hand, if Dafny fails to report an error on a program that is incorrect at run time, that *is* a bug. If you find an example of such a program, please [file an issue](https://github.com/Microsoft/dafny/issues). In general, the goal of always reporting an error on incorrect programs is called "soundness". 

In summary, Dafny is designed to be sound but incomplete. Thus, modulo bugs in Dafny itself, if no errors are reported, then the program meets its specification, but a reported error does not necessarily mean the program is incorrect.

## How does Dafny handle quantifiers? I've heard about "triggers", what are those?

### What do the warnings "no triggers found" or "selected trigger may loop" mean?

The warnings have to do with how Dafny (and the underlying solver Z3) handle quantifiers.

First of all, they truly are warnings. If the program has no errors, then it has passed the verifier and satisfies its specification. You don't *need* to fix the warnings.

However, in more complex programs you will often find that these warnings come along with failed or unpredictable verification outcomes. In those cases, it's worth knowing how to fix it. Often, the warnings can be eliminated by introducing a otherwise-useless helper function to serve as the trigger.

### What is a trigger?

A trigger is a syntactic pattern involving quantified variables that serves as heuristic for the solver to do something with the quantifier. With a forall quantifier, the trigger tells Dafny when to instantiate the quantified formula with other expressions. Otherwise, Dafny will never use the quantified formula.

For example, consider the formula

```
forall x {:trigger P(x)} :: P(x) && Q(x)
```

Here, the annotation `{:trigger P(x)}` turns off Dafny's automatic trigger inference and manually specifies the trigger to be `P(x)`. Otherwise, Dafny would have inferred both P(x) and Q(x) as triggers, which is better in general, but worse for explaining triggers.

This trigger means that whenever we mention an expression of the form `P(...)`, the quantifier will get instantiated, meaning that we get a copy of the body of the quantifier with `...` plugged in for `x`.

Now consider this program

```
method test()
    requires forall x {:trigger P(x)} :: P(x) && Q(x)
    ensures Q(0)
{
}
```

Dafny complains that it cannot verify the postcondition. But this is logically obvious! Just plug in `0` for `x` in the precondition to get `P(0) && Q(0)`, which implies the postcondition `Q(0)`.

Dafny cannot verify this method because of our choice of triggers. Since the postcondition mentions only Q(0), and nothing about `P`, but the quantifier is triggered only by `P`, Dafny will never instantiate the precondition.

We can fix this method by adding the seemingly-useless assertion

```
assert P(0);
```

to the body of the method. The whole method now verifies, including the postcondition. Why? Because we mentioned `P(0)`, which triggered the quantifier from the precondition, causing the solver to learn `P(0) && Q(0)`, which allows it to prove the postcondition.

Take a minute and realize what just happened. We had a logically-correct-but-failing-to-verify method and added a logically-irrelevant-but-true assertion to it, causing the verifier to suddenly succeed. In other words, Dafny's verifier can sometimes depend on logically-irrelevant influences in order to succeed, especially when there are quantifiers involved.

It is an essential part of becoming a competent Dafny user to understand when a failure can be fixed by a logically-irrelevant influence, and how to find the right trick to convince Dafny to succeed.

(As an aside, note that this example goes through without the irrelevant assertion if we let Dafny infer triggers instead of manually hobbling it.)

### What makes a good trigger?

Good triggers are usually small expressions containing the quantified variables that do not involve so-called "interpreted symbols", which, for our purposes, can be considered "arithmetic operations". Arithmetic is not allowed in triggers for the good reason that the solver cannot easily tell when a trigger has been mentioned. For example, if `x + y` was an allowed trigger and the programmer mentioned `(y + 0) * 1 + x`, the solver would have trouble immediately recognizing that this was equal to a triggering expression. Since this can cause inconsistent instantiation of quantifiers, arithmetic is disallowed in triggers.

Many other expressions are allowed as triggers, such as indexing into Dafny data structures, dereferencing fields, set membership, and function application.

Sometimes, the most natural way of writing down a formula will contain no valid triggers, as your original example did. In that case, Dafny will warn you. Sometimes, verification will succeed anyways, but in large programs you will often need to fix these warnings. A good general strategy is to introduce a new function the abstract some part of the quantified formula that can serve as a trigger.

## I get a "call may violate context's modifies clause" error even though I know I am not modifying anything.

This can happen if you allocate a new object or array in one method and then try to use it another. For example, consider the following program 

```
method MakeArray() returns (a: array<int>)
    ensures a.Length > 0
{
    return new int[10];
}

method Main()
{
    var a := MakeArray();
    a[0] := 0;  // error: may violate modifies clause
}
```

Dafny reports that the assignment in `Main` to `a[0]` may violate the context's modifies clause. Since `Main` has no modifies clause whatsoever, it is only allowed to modify things that it allocates itself. However, because Dafny works method-by-method, it is not obvious to Dafny that the array returned from `MakeArray` is newly allocated. The fix is to add a postcondition 

```ensures fresh(a)```

to `MakeArray`, which exposes the fact that `a` is newly allocated. Then the whole program verifies. 

In a post condition, the meaning of `fresh(a)` is "`a` was allocated during the execution of this method". This allows Dafny to conclude that `a` is not equal to anything in the caller's context, and so `Main` is welcome to modify the newly allocated array.

See the tutorial section on "Framing" for more detail about `modifies` clauses.




