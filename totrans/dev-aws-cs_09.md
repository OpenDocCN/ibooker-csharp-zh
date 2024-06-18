# Appendix A. Benchmarking AWS

One way to get a more profound knowledge of AWS is to benchmark the performance of different machines. Part of the value of AWS is the ability to use many various services and other abstractions to solve problems in the most efficient manner possible. You can view many code examples in this [example GitHub Repository on benchmarking](https://github.com/noahgift/benchmarking-aws). Notice that there are substantial differences in sysbench benchmarks from 1 core to a 36 core to a 96 core machine as shown in [Figure A-1](#Figure-1-8).

![doac ab01](assets/doac_ab01.png)

###### Figure A-1\. Benchmarking AWS instances

As your organization determines the correct Amazon Linux 2 machine to run .NET Core on Linux, it would be a great idea to do some initial benchmarking. To run these benchmarks in a repeatable manner on Amazon Linux 2, you can refer to this [`Makefile`](https://oreil.ly/IF5rR). Notice how `sysbench` only needs two lines to install and run the `./benchmark.sh` script.

The actual benchmark script is as follows:

[PRE0]

The `makefile` command can be run as `make benchmark-sysbench-amazon`:

[PRE1]

The benchmark script uses a “one-liner” to determine how many cores are available and then passes this into a CPU benchmark:

[PRE2]

A free tool like [Cinebench](https://oreil.ly/ROql2) works to benchmark a Windows OS. It is always good to perform multiple measures, including an OS-level benchmark and application load test, on the actual environment in which an application gets deployed.