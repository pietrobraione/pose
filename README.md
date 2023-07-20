# POSE: Path-optimal symbolic execution prototype

This repository contains the prototype implementation of path-optimal symbolic execution. To compile and run it you need Coq, OCaml (that usually comes with Coq), make and Z3. Build the Coq files and extract the OCaml implementation by running

    $ make

You can find some examples in the Examples.v file. Process it row by row in your favorite Coq environment (I use Proof General since CoqIDE does not play well with mathematical symbols). To compile the OCaml program just issue the commands:

    $ ocamlopt pose.mli
    $ ocamlopt -o run pose.ml run.ml

You will find a run executable, with a sketchy help that you can read by issuing the command

    $ ./run -h

A possible execution of the tool is:

    $ ./run -p -z3 /usr/bin/z3 dll.txt 100

It will run the dll.txt example up to depth 100, by pruning the infeasible states using Z3 (option -p), where the Z3 executable is installed at /usr/bin. The run will print the configurations (current program, heap, path condition and expression under evaluation) reached after 100 execution steps.

Note that this prototype is underoptimized, and consequently slow and memory-consuming. If its execution crashes with a stack overflow error, crank up the stack memory with the Unix command ulimit -s. Expect in any case execution times to suddenly explode when states start to become big, which usually happens at high depths. I optimized to have decent time with the experiments in this repo, but on other programs your mileage may vary.

Some notes on the small programming language that you must use to write the programs you want to symbolically execute with this prototype. It comes with some limitations, that you can work around as follows:

* All fields must have different names even if they belong to different classes. Do not reuse field names across classes or you will confuse the tool.
* There is no sequential composition: Use let-binding, perhaps by assigning to dummy variables.
* There are ifs, but there are no loops (for, while): Use method recursion instead.
* All methods have exactly one parameter. Use a suitable object if you want more (or less) than one parameter.

The grammar of the language is reported in Parse.v, that implements a LL parser based on parser combinators. Alas, due to the need of keeping the grammar LL there are lots of silly parentheses, and you are not even free to add parentheses where you want. Refer the grammar or the examples (the treemap.txt example is quite comprehensive and also shows all the above techniques). Another limitation of the parser is that it does not do any semantic error checking, especially of typing errors. The language itself is terribly dynamic: If you do a typing error this is detected at runtime, and the typical effect is that the state that has the error is killed. When a state has not a successor it is either of two: Either the state is final, or the state was killed because of some semantic error. So if your symbolic execution has less traces than you think, maybe it is because there is some bug in the program you are analyzing. My advice is: Run your program with some concrete inputs (a.k.a., tests) and see if you obtain exactly one end state, and this end state is the one you expect. When you are pretty sure that your program is correct execute it symbolically.

For what concerns the examples included in the repo here are some stats (with pruning on):

* dll: maximum depth 143, leaves 3;
* treemap: maximum depth 336, leaves 48.

Note that for the run tool to produce the exact count of the number of leaves you must provide it a depth that is at least one more than the maximum depth.

