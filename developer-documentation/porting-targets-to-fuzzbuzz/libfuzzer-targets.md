# Libfuzzer Targets

Fuzzbuzz looks for the target method `FuzzerEntrypoint`, so the easiest way to prepare a Libfuzzer target for Fuzzbuzz is to add the following method to the same file that contains `LLVMFuzzerTestOneInput`:

{% tabs %}
{% tab title="C" %}
```c
int FuzzerEntrypoint(const uint8_t *Data, size_t Size) {
    return LLVMFuzzerTestOneInput(Data, Size)
}
```

{% hint style="info" %}
Fuzzbuzz also supports the `LLVMFuzzerInitialize` method, and will call it before fuzzing if it exists.
{% endhint %}
{% endtab %}

{% tab title="C++" %}
```cpp
extern "C" int FuzzerEntrypoint(const uint8_t *Data, size_t Size) {
    return LLVMFuzzerTestOneInput(Data, Size)
}
```

{% hint style="info" %}
Fuzzbuzz also supports the `LLVMFuzzerInitialize` method, and will call it before fuzzing if it exists.
{% endhint %}
{% endtab %}
{% endtabs %}

To compile your target, replace calls to clang with `FUZZ_CC` and `FUZZ_CXX`, and use the flags in `CFLAGS` and `CXXFLAGS`. You can append your own flags, but be sure to include the Fuzzbuzz-provided ones. These help us build special versions of your target to track metrics like code coverage.

Rather than linking with Libfuzzer using `-fsanitize=fuzzer` or similar methods, simply link the file containing `FuzzerEntrypoint` and all other required files with `$FUZZ_ENGINE`.

Combining all the above steps may look something like this:

```bash
$FUZZ_CXX $CXXFLAGS my_api.h my_target.cc $FUZZ_ENGINE
```

Libfuzzer configuration options like memory limit, timeout threshold and dictionary can also be set in the configuration file.This is an example of a configuration file with all the features described above:

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
        - $FUZZ_CXX $CXXFLAGS my_api.h my_target.cc $FUZZ_ENGINE
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

