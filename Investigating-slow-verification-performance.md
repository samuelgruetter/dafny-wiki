## Debugging performance issues with the AxiomProfiler

*This is an advanced topic.*

We can use the [Axiom Profiler](https://bitbucket.org/viperproject/axiom-profiler/) to debug Dafny performance issues. 

### 1. Obtaining the Boogie file and performance log

Generate a Boogie (bpl) file using `/print`. We also generate a log XML file, which shows which verification tasks take a long time. The `@FILE@` and `@TIME@` placeholders are expanded by Boogie.

    Dafny.exe file.dfy /print:file.bpl /xml:profile-@FILE@-@TIME@.xml

If you want to just create a Boogie file without running verification and without producing other output you can run

    Dafny.exe file.dfy /compile:0 /noVerify /print:file.bpl
 
### 2. Running Boogie
We will now call z3 through Boogie to create the log that we then feed to the AxiomProfiler. Some flags are necessary to be passed to z3. More information at https://bitbucket.org/viperproject/axiom-profiler/src/default/ and https://github.com/microsoft/dafny/issues/229.

The parameter `/proc` limits Boogie to the slow (in verification time) method that we detected by inspecting the XML log. 

It is important to **use z3 version >= 4.8.5 for Axiom Profiler to work**. At the time of writing this requires z3 to be built from source.

    Boogie.exe /z3exe:'<path to z3>' /proc:'<name of slow Boogie task starting with 'Impl$$' or so>' /timeLimit:5 /z3opt:TRACE=true /z3opt:PROOF=true /z3opt:trace-file-name=file.log /vcsCores:1 file.bpl
 
### 3. Running the AxiomProfiler

Run the AxiomProfiler on the log file we just obtained from z3.

    AxiomProfiler /l:file.log

### 4. Interpreting AxiomProfiler output

To understand the AxiomProfiler output we recommend reading the AxiomProfiler paper at:

* http://people.inf.ethz.ch/summersa/wiki/lib/exe/fetch.php?media=papers:axiomprofiler.pdf

And to read up on matching triggers, for example at:
* https://github.com/dafny-lang/dafny/wiki/FAQ#how-does-dafny-handle-quantifiers-ive-heard-about-triggers-what-are-those
* https://www.microsoft.com/en-us/research/publication/computing-smt-solver/