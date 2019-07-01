We use the [LLVM integrated tester (lit)](https://llvm.org/docs/CommandGuide/lit.html) to test Dafny.

Dafny's test suite is located in the `Test` directory.

We provide a global lit configuration in the main test directory: `Test/lit.site.cfg`. We additionally provide local test configurations within some of the `Test/*` directories, such as for example `Test/comp/lit.local.cfg`.

lit is additionally instrumented in the headers of the test files. Each `*.dfy` file within the test suite provides information on how to call Dafny on itself, which is then combined with the global and local configurations. The test's output is then compared to the corresponding `*.dfy.expect` file via `diff`.

lit stores temporary outputs in an `Output` directory adjacent to the particular test file in the directory structure. If you encounter unexpected issues, you can get additional output by running lit in verbose mode (`lit --verbose`).

lit, by default, runs tests in parallel. For performance measurements, run lit with the `-j 1` option.