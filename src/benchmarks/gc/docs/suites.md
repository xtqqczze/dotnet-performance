# Suites

Apart from individual tests, GC Benchmarking Infra can run and process results
in sets with its suite functionality. This is specified in a `suite.yaml` file,
which has the following structure:

```yml
bench_files:
- test1.yaml
- test2.yaml
- test3.yaml
- test4.yaml
command_groups:
  example_command_group:
  - command args
  - command args
  - command args
```

These sections are explained with examples in the following sections.

## Suite Run

The `suite-run` command allows you to run a set of tests by specifying the `yaml`
files in the order you want them to be run. GC Infra will read and parse each one
of these files and run the benchmark tests accordingly. If for some reason an
error occurs, that test will be ignored and infra will proceed to the next one.

At the end of the run, if any problems were encountered, a summary detailing the
exceptions, the respective messages, and the stack traces will be shown to the user.

You can write your own suite files to tailor to your specific performance benchmarking
needs. Let's take the suite generated by default as an example.

```yml
bench_files:
- normal_workstation.yaml
- normal_server.yaml
- high_memory.yaml
- low_memory_container.yaml
command_groups: {}
```

We would run this by issuing the following command:

```sh
py . suite-run bench/suite/suite.yaml
```

The result of this is the equivalent of individually running each test like this:

```sh
py . run bench/suite/normal_workstation.yaml
py . run bench/suite/normal_server.yaml
py . run bench/suite/high_memory.yaml
py . run bench/suite/low_memory_container.yaml
```

Once the last test has finished running, if everything went fine, you'll be
greeted with a message saying "*\*\*\* Suite run finished successfully! \*\*\**".

If something went wrong somewhere, you will be shown a summary with the following
format:

```text
*WARNING*: One or more tests in the suite encountered errors.

*** Here is a summary of the problems found: ***

========= <Yaml Filename> =========

======= <Executable Name> =======

===== <Coreclr Name> =====

=== <Configuration Name> ===

- <Benchmark Name> -
<Iteration Number>
<Error Message>

<Stack Trace>

<Repeat with other errors that might have occurred>
```

## Suite Run Command

Just as you can run tests in bulk, you can also run analysis commands bundled
in sets. GC Infra will read each command from the given group and issue it, one
after the other. Let's add this group to our default example.

```yml
bench_files:
- normal_workstation.yaml
- normal_server.yaml
- high_memory.yaml
- low_memory_container.yaml
command_groups:
  first-gcs:
  - analyze-single bench/suite/normal_server.yaml.out/a__only_config__0gb__0.yaml --show-first-n-gcs 10 --gc-where Generation=0 --txt bench/suite/server_firstgcs.txt
  - analyze-single bench/suite/normal_workstation.yaml.out/a__only_config__0gb__0.yaml --show-first-n-gcs 10 --gc-where Generation=0 --txt bench/suite/wks_firstgcs.txt
```

In this example, we are naming our command group *first-gcs*, and adding to *analyze-single*
calls to see the details on the first 10 GC's of Generation 0, from the *0GB* benchmarks
in the server and workstation default tests. It is highly recommended you add
the `--txt` option to your commands, as GC analysis usually yields big outputs,
and it becomes harder to read if you need to scroll up through various results to
find the one you are interested in first.

To run this set, issue the following command:

```sh
py . suite-run-command --suite-path bench/suite/suite.yaml --command-name first-gcs
```

Once this has finished running, you will find the results in the text files you
provided in the `yaml` file. It is worth mentioning that despite being in the
same file, *bench_files* and *command_groups* are not related, as in what one says
does not affect the other.

Similar to the `suite-run` command, this `suite-run-command` is equivalent to
running the following:

```sh
py . analyze-single bench/suite/normal_server.yaml.out/a__only_config__0gb__0.yaml --show-first-n-gcs 10 --gc-where Generation=0 --txt bench/suite/server_firstgcs.txt
py . analyze-single bench/suite/normal_workstation.yaml.out/a__only_config__0gb__0.yaml --show-first-n-gcs 10 --gc-where Generation=0 --txt bench/suite/wks_firstgcs.txt
```
