Proving termination by decreasing Repr
======================================

This page describes how `Repr` sets can be used to prove termination of recursive method calls.
It requires some understanding of dynamic frames, `Repr` sets, and `modifies` clauses.
Chapters 8 and 9 of [Using Dafny, an Automatic Program Verifier](http://leino.science/papers/krml221.pdf) provide a good introduction to this.

Suppose we have a simple counter interface and implementation:

```
trait Counter {
    method inc() modifies this
    method get() returns (res: nat) modifies this 
}

class BasicCounter extends Counter {
    var c: nat
    constructor() {
        c := 0;
    }
    method inc() 
        modifies this
    {
        c := c + 1;
    }
    method get() returns (res: nat) {
        res := c;
    }
}
```

This seems to work fine, but as soon as we want to add this `LogWrapCounter`, we get errors:

```
class LogWrapCounter extends Counter {
    var underlying: Counter;
    constructor(underlying: Counter) {
        this.underlying := underlying;
    }
    method inc() 
        modifies this
    {
        print "inc is called\n";
        underlying.inc(); // ERRORS
    }
    method get() returns (res: nat) {
        print "get is called\n";
        res := underlying.get(); // ERRORS
    }
}
```

Both lines marked with `ERRORS` have the following two errors:

* `call may violate context's modifies clause`
* `cannot prove termination; try supplying a decreases clause`

Dafny rightfully points out that when we call `underlying.inc`, `underlying` will be modified, but the `modifies` clause only contains `this`, not `this.underlying`. 
If we try to fix this by saying `modifies this, this.underlying`, the `modifies` clause won't match the `modifies` clause of the parent trait any more, and we can't adapt the `modifies` clause of the parent trait because the parent trait has no `underlying` field.
So we have to use the more general approach of `Repr` sets, and rewrite the code as follows:

```
trait Counter {
    ghost var Repr: set<object>
    predicate Valid() reads this, Repr
    method inc() requires Valid() modifies Repr ensures Valid() && Repr == old(Repr)
    method get() returns (res: nat) requires Valid()
}

class BasicCounter extends Counter {
    var c: nat
    predicate Valid() reads this, Repr {
        this in Repr
    }
    constructor() ensures Valid() && fresh(Repr) {
        this.c := 0;
        this.Repr := {this};
    }
    method inc() 
        requires Valid() modifies Repr ensures Valid() && Repr == old(Repr)
    {
        c := c + 1;
    }
    method get() returns (res: nat) requires Valid() {
        res := c;
    }
}

class LogWrapCounter extends Counter {
    var underlying: Counter;
    predicate Valid() reads this, Repr {
        this in Repr && underlying in Repr && underlying.Repr <= Repr && this !in underlying.Repr &&
        underlying.Valid()
    }
    constructor(underlying: Counter) 
        requires underlying.Valid()
        ensures Valid()
        ensures fresh(Repr - {underlying} - underlying.Repr)
    {
        this.underlying := underlying;
        this.Repr := underlying.Repr + {underlying, this};
    }
    method inc() 
        requires Valid() 
        modifies Repr     
        ensures Valid() && Repr == old(Repr)
    {
        print "inc is called\n";
        underlying.inc(); // ERROR
    }
    method get() returns (res: nat) requires Valid() {
        print "get is called\n";
        res := underlying.get(); // ERROR
    }
}
```

Now the first error is gone, and the two lines marked with `ERROR` only have the error

* `cannot prove termination; try supplying a decreases clause`

This error is also a valid concern: 
For instance, the following method would not terminate, because `c1.inc()` would call `c2.inc()`, which in turn would call `c1.inc()` again, and so on:

```
method infiniteLoop() {
    var b := new BasicCounter();
    var c1 := new LogWrapCounter(b);
    var c2 := new LogWrapCounter(c1);
    c1.underlying := c2;
    c1.inc();
}
```

This problem can also be solved using `Repr` sets, because each call to `inc` should be a call to an underlying counter whose `Repr` set is strictly smaller (it does not contain `this`), so we can use the `Repr` set as a termination measure:

```
trait Counter {
    ghost var Repr: set<object>
    predicate Valid() reads this, Repr
    method inc() requires Valid() modifies Repr decreases Repr ensures Valid() && Repr == old(Repr)
    method get() returns (res: nat) requires Valid() decreases Repr
}

class BasicCounter extends Counter {
    var c: nat
    predicate Valid() reads this, Repr {
        this in Repr
    }
    constructor() ensures Valid() && fresh(Repr) {
        this.c := 0;
        this.Repr := {this};
    }
    method inc() 
        requires Valid() modifies Repr decreases Repr ensures Valid() && Repr == old(Repr)
    {
        c := c + 1;
    }
    method get() returns (res: nat) requires Valid() decreases Repr {
        res := c;
    }
}

class LogWrapCounter extends Counter {
    var underlying: Counter;
    predicate Valid() reads this, Repr {
        this in Repr && underlying in Repr && underlying.Repr <= Repr && this !in underlying.Repr &&
        underlying.Valid()
    }
    constructor(underlying: Counter) 
        requires underlying.Valid()
        ensures Valid()
        ensures fresh(Repr - {underlying} - underlying.Repr)
    {
        this.underlying := underlying;
        this.Repr := underlying.Repr + {underlying, this};
    }
    method inc() 
        requires Valid() 
        modifies Repr 
        decreases Repr 
        ensures Valid() && Repr == old(Repr)
    {
        print "inc is called\n";
        underlying.inc();
    }
    method get() returns (res: nat) requires Valid() decreases Repr {
        print "get is called\n";
        res := underlying.get();
    }
}
```

This is interesting because the static call graph anaysis indicates that `LogWrapCounter.inc()` might call itself, and there's no non-ghost decreasing argument in sight -- but the `ghost var Repr` saves us.

Now if we try to verify the `infiniteLoop()` method from above, Dafny tells us that the precondition for `c1.inc()` might not hold.
We could try to fix this by also updating the `Repr` set of `c1` after assigning `c1.underlying`, but then of course `c1 !in c1.underlying.Repr` does not hold any more, but that's required by `c1.Valid()`, which is a precondition for `c1.inc()`.

```
method infiniteLoop() {
    var b := new BasicCounter();
    var c1 := new LogWrapCounter(b);
    var c2 := new LogWrapCounter(c1);
    assert c2.Valid();
    c1.underlying := c2;
    c1.Repr := c1.Repr + c2.Repr;
    assert c1 !in c1.underlying.Repr; // fails
    c1.inc();
}
```

So Dafny has correctly prevented us from writing infinite loops through object oriented virtual method calls.
But of course, it allows correct usages of our counters, eg we can log each event twice as follows:

```
method Main() {
    var b := new BasicCounter();
    var c1 := new LogWrapCounter(b);
    var c2 := new LogWrapCounter(c1);
    c2.inc();
    c2.inc();
    var v := c2.get();
    print v, "\n";
    c2.inc();
    v := c2.get();
    print v, "\n";
}
```

which prints

```
inc is called
inc is called
inc is called
inc is called
get is called
get is called
2
inc is called
inc is called
get is called
get is called
3
```


