# AFL Targets

To fuzz targets written for AFL, replace calls to AFL's compilers \(i.e. `afl-clang`,`afl-clang++` etc\) with `FUZZ_STANDALONE_CC` and `FUZZ_STANDALONE_CXX`. If you have a configurable build system, this may look something like:

```bash
CC=$FUZZ_STANDALONE_CC CXX=$FUZZ_STANDALONE_CXX make my_target
```

Ensure that you also use the `CFLAGS` and `CXXFLAGS` provided in the build environment. You can append your own, but be sure to include the Fuzzbuzz-provided ones. These help us build special versions of your target to track metrics like code coverage.

Then, in the harness field, specify whether the binary takes input via stdin, or from a file. If the binary takes input from a file, mark the location in the binary's command line where the input file should be placed with a @@. This will be replaced automatically when fuzzing.

AFL configuration options like memory limit, timeout threshold and dictionary can also be set in the configuration file. This is an example of a configuration file with all the features described above:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
language: c++
targets:
    - name: my-target
      environment:
        # you can specify your own CFLAGS/CXXFLAGS, just remember to
        # use the fuzzbuzz-provided defaults as well
        - CXXFLAGS="$CXXFLAGS -g"
      setup:
        - $FUZZ_STANDALONE_CXX $CXXFLAGS my_binary.cpp -o ./target
      corpus: ./my_target/corpus
      timeout: 500 # in milliseconds
      memory_limit: 1000 # in megabytes
      dictionary: ./my_target/dict # location of dictionary file
      harness:
        binary: ./target @@
        # input can be one of: stdin, file, socket
        input: file
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You're ready to go! [Push your project to Fuzzbuzz](../fuzzing-jobs.md) and it will detect your targets automatically.

{% hint style="info" %}
Check out our [Advanced Configuration](../targets.md#advanced-configuration) documentation to learn how to use sanitizers on Fuzzbuzz
{% endhint %}

