---
description: >-
  A target is a self-contained unit that receives a test case from Fuzzbuzz,
  runs the test, and reports any bugs. This page describes how to build a target
---

# Targets

{% hint style="info" %}
This page describes how to write new targets for Fuzzbuzz. If you have existing targets for AFL, Libfuzzer, Gofuzz or other open-source fuzzing frameworks, they already work on Fuzzbuzz! [Click Here](porting-targets-to-fuzzbuzz/) to learn how.
{% endhint %}

Each target consists of two parts:

1. a method within your code, that Fuzzbuzz provides test data to - similar to a unit testing framework
2. an entry in the `targets` list of the project's `fuzz.yaml`, which describes how to run the target, and defines the type of errors to look for

## Test Method

{% tabs %}
{% tab title="C" %}
To fuzz code written in C, create a new file with the following method:

{% code-tabs %}
{% code-tabs-item title="target.c" %}
```c
int FuzzerEntrypoint(const uint8_t *Data, size_t Size) {
  // Step 1: read Data into desired format
  // Step 2: run your methods/code with the test data
  // Step 3: check for errors, and abort if errors are found
  // return -1 if the Data is irrelevant or invalid
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can use [assert](https://en.cppreference.com/w/cpp/error/assert) to check for errors as you would in a unit test - Fuzzbuzz will pick up failed assertions as bugs.
{% endtab %}

{% tab title="C++" %}
To fuzz code written in C++, create a new file with the following method:

{% code-tabs %}
{% code-tabs-item title="target.cpp" %}
```cpp

extern "C" int FuzzerEntrypoint(const uint8_t *Data, size_t Size) {
  // Step 1: read Data into desired format
  // Step 2: run your methods/code with the test data
  // Step 3: check for errors, and abort if errors are found
  // return -1 if the Data is irrelevant or invalid 
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can use [assert](https://en.cppreference.com/w/cpp/error/assert) to check for errors as you would in a unit test - Fuzzbuzz will pick up failed assertions as bugs.
{% endtab %}

{% tab title="Go" %}
To fuzz code written in Golang, include the following method in your package:

{% code-tabs %}
{% code-tabs-item title="target.go" %}
```go
func FuzzerEntrypoint(Data []byte) int {
  // Step 1: read Data into desired format
  // Step 2: run your methods/code with the test data
  // Step 3: check for errors, and abort if errors are found 
  // return -1 if the Data is irrelevant or invalid
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Use panics to alert on bugs - Fuzzbuzz will pick up these failures.
{% endtab %}

{% tab title="Python" %}
To fuzz code written in Python, include the following method in your package:

{% code-tabs %}
{% code-tabs-item title="target.go" %}
```python
def FuzzerEntrypoint(Data): # Data is a buffer of bytes
  # Step 1: read Data into desired format
  # Step 2: run your methods/code with the test data
  # Step 3: check for errors, and abort if errors are found 
  # return -1 if the Data is irrelevant or invalid
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can use Python's [assert statement](https://wiki.python.org/moin/UsingAssertionsEffectively) to check for errors as you would in a unit test - Fuzzbuzz will pick up these failures.
{% endtab %}

{% tab title="Binaries" %}
Fuzzbuzz can fuzz entire binaries compiled with C or C++, that read data from files, stdin, or sockets. Check the [Configuration Section](targets.md#configuration) to learn how to fuzz these.
{% endtab %}
{% endtabs %}

This method should take the test case defined in the array `Data` and convert it into the desired test data - for example, if the code or methods you wish to test take 2 integers as input, parse 2 integers from the array of bytes. This interface aims to be as flexible as possible - you can parse any data type from these bytes, populate structs or objects, and generate very complex test structures. If the array of bytes isn't long enough to populate your desired structures, return -1 right away. The fuzzing engine is a quick learner, and will realize more data is required!

## Configuration

{% hint style="info" %}
This section provides a descriptive overview of `fuzz.yaml` configuration. For full, detailed documentation check out the [Configuration Reference](../reference/configuration.md).
{% endhint %}

Every target needs an entry in the `targets` list of the project's `fuzz.yaml`:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
language: c
targets:
    - name: my-target # this name should be unique within the project
      environment: # environment specific for this target
        - VAR=value
      setup: # setup steps specific for this target
        - make my-target
      corpus: my-target/corpus # a directory containing initial test cases
      harness: # special language-specific configuration

```
{% endcode-tabs-item %}
{% endcode-tabs %}

The `name` field is compulsory, and the `setup` steps should be used to install needed dependencies. For languages like C and C++, the `setup` steps should also be used to compile the target binaries. 

The `corpus` directory should contain a list of files, each with an input to the Test Method that does not produce in a bug. This field is optional, but examples of tests that pass gives Fuzzbuzz an idea of what a good input looks like, and can make finding new code paths more efficient.

Finally, the `harness` field tells Fuzzbuzz how to run the Test Method. This entry requires certain fields depending on the specific language of the target, as described below:

{% tabs %}
{% tab title="C" %}
To fuzz C code, compile the file containing `FuzzerEntrypoint` along with all other required files and libraries, using the `FUZZ_CC` and `CFLAGS` environment variables. 

For example, if `FuzzerEntrypoint` is implemented in `my_target.c` and calls a method in `my_api.h`:

```bash
$FUZZ_CC $CFLAGS my_api.h my_target.c -c
```

Finally, to compile into a full binary, link with `FUZZ_ENGINE`, which provides a `main` method as well as the necessary feedback for Fuzzbuzz, using the C++ compiler:

```bash
$FUZZ_CXX $CXXFLAGS my_target.o $FUZZ_ENGINE -o ./target
```

{% hint style="warning" %}
Even for code written in C, remember to link with `FUZZ_ENGINE` using the C++ compiler `FUZZ_CXX`
{% endhint %}

This is an example of all the above steps combined to fully configure a target written in C:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
language: c
targets:
    - name: my-target
      environment:
        # you can specify your own CFLAGS/CXXFLAGS, just remember to
        # use the fuzzbuzz-provided defaults as well
        - CFLAGS="$CFLAGS -g"
      setup:
        - $FUZZ_CC $CFLAGS my_api.h my_target.c -c
        - $FUZZ_CXX $CXXFLAGS my_target.o $FUZZ_ENGINE -o ./target
      corpus: ./my_target/corpus
      harness:
        binary: ./target
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="C++" %}
To fuzz C++ code, compile the file containing `FuzzerEntrypoint` along with all other required files and libraries, using the `FUZZ_CXX` and `CXXFLAGS` environment variables. 

For example, if `FuzzerEntrypoint` is implemented in `my_target.cpp` and calls a method in `my_api.h`:

```bash
$FUZZ_CXX $CXXFLAGS my_api.h my_target.cpp -c
```

Finally, to compile into a full binary, link with `FUZZ_ENGINE`, which provides a `main` method as well as the necessary feedback for Fuzzbuzz:

```bash
$FUZZ_CXX $CXXFLAGS my_target.o $FUZZ_ENGINE -o ./target
```

This is an example of all the above steps combined to fully configure a target written in C++:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
language: c++
targets:
    - name: my-target
      environment:
        # you can specify your own CFLAGS/CXXFLAGS, just remember to
        # use the fuzzbuzz-provided defaults as well
        - CXXFLAGS="$CXXFLAGS -g"
      setup:
        - $FUZZ_CXX $CXXFLAGS my_api.h my_target.cpp -c
        - $FUZZ_CXX $CXXFLAGS my_target.o $FUZZ_ENGINE -o ./target
      corpus: ./my_target/corpus
      harness:
        binary: ./target

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Go" %}
To fuzz Go code, use the `setup` steps to install all the needed dependencies. Use the `checkout` field to specify where in the Gopath your code should be placed. This can be set at the root of the `fuzz.yaml` and then overridden for a specific target if needed. Then, specify the package and method to fuzz in the `harness` field.

This is an example of the configuration of a target written in Go:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
language: go
version: "1.11"
# checkout specifies where to check your code out
# this repository will be placed in the directory:
# ~/go/src/github.com/x/y
checkout: github.com/x/y
targets:
    - name: my-target
      setup:
        - dep ensure # make sure all dependencies are installed
      corpus: ./my_target/corpus
      harness:
        # Go targets are flexible - your method doesn't need to
        # be named FuzzerEntrypoint
        function: FuzzerEntrypoint
        # build tags are optional
        build_tags: tag1 tag2 tag3
        # package specifies the package to import the
        # desired function from 
        package: github.com/x/y/z/a/b/c

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Python" %}
To fuzz Python code, use the `setup` steps to install all the needed dependencies. Use the `file` field to specify the Python file that contains the test harness. Then, specify the function to fuzz in the `function` field.

This is an example of the configuration of a target written in Go:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
language: python
version: "3.6"
setup:
  - pip3 install -r requirements.txt
targets:
    - name: my-target
      harness:
        # Python targets are flexible - your method doesn't need to
        # be named FuzzerEntrypoint
        function: FuzzerEntrypoint
        file: tests/fuzz_test_file.py

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Binaries" %}
To fuzz binaries written in C or C++, compile them using the following environment variables:

* `FUZZ_STANDALONE_CC` and `CFLAGS`
* `FUZZ_STANDALONE_CXX` and `CXXFLAGS`

These compilers ensure that Fuzzbuzz gets the feedback it needs to provide information on code coverage and more complex memory corruption bugs.

Then, in the `harness` field, specify whether the binary takes input via `stdin`, a network `socket`, or from a `file`. If the binary takes input from a file, mark the location in the binary's command line where the input file should be placed with a `@@`. This will be replaced automatically when fuzzing.

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
targets:
    - name: my-target
      language: c++
      environment:
        # you can specify your own CFLAGS/CXXFLAGS, just remember to
        # use the fuzzbuzz-provided defaults as well
        - CXXFLAGS="$CXXFLAGS -g"
      setup:
        - $FUZZ_STANDALONE_CXX $CXXFLAGS my_binary.cpp -o ./target
      corpus: ./my_target/corpus
      harness:
        binary: ./target @@
        # input can be one of: stdin, file, socket
        # currently socket is experimental, and is not available
        # on the main platform
        input: file
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}
{% endtabs %}

You're ready to go! Push your project to Fuzzbuzz and it will detect your targets automatically. Check out the Fuzzing Jobs page to learn how to fuzz your targets on the platform, or keep reading below to learn how Advanced Configuration options can help you get the most out of your code.

{% page-ref page="fuzzing-jobs.md" %}

## Advanced Configuration

Fuzzbuzz provides features that allow you to monitor your target for extreme memory usage, tests that take too long to run, and memory bugs \(in certain languages\).

### Memory Limits

You can specify a memory limit \(in Megabytes\) in your target configuration, and Fuzzbuzz will report any test cases that cause your target to use more memory than the limit.

**Usage:**

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
targets:
    # ---- rest of target config omitted
    memory_limit: '1000' # in megabytes
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Timeouts

You can specify a timeout \(in milliseconds\) in your target configuration, and Fuzzbuzz will report any test cases that take longer than the timeout threshold to process.

#### **Usage:**

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
targets:
    # ---- rest of target config omitted
    timeout: '500' # in milliseconds
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Sanitizers

Fuzzbuzz exposes the use of sanitizers for C and C++ binaries - these sanitizers monitor your code for certain memory bugs, including buffer overflows and potentially dangerous memory usage.

Currently Fuzzbuzz supports [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer). More sanitizers will be provided in the future.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
# ---- base, global setup, and global environment omitted
targets:
    # ---- rest of target config omitted
    sanitizers:
        # you can specify custom sanitizer options here
        address: detect_stack_use_after_return=1:debug=1 # can be left blank

```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can learn more about configuring your targets in the [Configuration Section](../reference/configuration.md).

