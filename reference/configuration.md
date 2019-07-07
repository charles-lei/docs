# Configuration

## Target Configuration \(fuzzbuzz.yaml\)

Every repository needs a `fuzzbuzz.yaml`, which tells Fuzzbuzz how to set up and configure your fuzz targets.

The following sections describe each field allowed in a `fuzzbuzz.yaml` configuration file and how to use them.

### base

**Type: `string`**

**Required: yes**

**Valid values: `ubuntu:16.04`**

The name of the operating system to use as the base of the Docker image the target is set up and fuzzed in. As of now, `ubuntu:16.04` is the only option.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
base: ubuntu:16.04
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### environment

**Type: `Array`**

**Required: no**

Environment variables that will be accessible to each target.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
environment:
    - ENV_VAR=value
    - ENV_VAR2=value2
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### setup

**Type: `Array`**

**Required: no**

A list of commands to run prior to each target's specific setup commands.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
setup:
    - sudo apt-get update
    - sudo apt-get install my-dependency
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### language

**Type: `string`**

**Required: yes**

**Valid values: `c, c++, go`**

The name of the language the target was written in. This must be specified either here, or in `target.language`. If it is specified in both, the value in `target.language` overrides the one here.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
language: c++
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### version

**Type: `string`**

**Required: yes \(all languages except C/C++\)**

The version of the target's language to use. This must be specified either here, or in `target.version`. If it is specified in both, the value in `target.version` overrides the one here.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
language: go
version: "1.11"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### checkout

**Type: `string`**

**Required: yes \(Golang only\)**

Specifies where in the Gopath to place your code. This must be specified either here, or in `target.harness.checkout`. If it is specified in both, the value in `target.harness.checkout` overrides the one here.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
# checkout specifies where to check your code out
# this repository will be placed in the directory:
# ~/go/src/github.com/x/y/
checkout: github.com/x/y
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### targets

**Type: `Array`**

**Required: yes**

An array of target configurations.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: target-name
    # --- Other target configuration options omitted
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.name

**Type: `string`**

**Required: yes**

The name of the target. Must be unique within the `fuzzbuzz.yaml`. Can only contain letters, numbers, dashes and underscores.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.language

**Type: `string`**

**Required: yes**

**Valid values: `c, c++, go`**

The name of the language the target was written in.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      language: c
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.version

**Type: `string`**

**Required: yes \(all languages except C/C++\)**

The version of the target's language to use.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      language: go
      version: "1.11"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.environment

**Type: `Array`**

**Required: no**

Environment variables that will be accessible to this specific target.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      environment:
        - ENV_VAR=value
        - ENV_VAR2=value2
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.setup

**Type: `Array`**

**Required: no**

A list of commands to run that set up or compile the target.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      setup:
        - sudo apt-get install my-lib
        - CC=$FUZZ_CC CXX=$FUZZ_CXX make my-target
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.corpus

**Type: `string`**

**Required: yes**

The location of the target's test corpus. This should point to a directory of files, each of which should be a separate test case that can be fed to the target and runs a test that passes. The directory can be left empty, but this will result in a more inefficient start to fuzzing.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      corpus: ./my-target/corpus
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.harness

**Type: `Map`**

**Required: yes**

Configuration that tells Fuzzbuzz how to find & run the target. Configuration differs depending on the language

{% tabs %}
{% tab title="C" %}
Harness specifies the location of the final compiled binary.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      harness:
        binary: ./target
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="C++" %}
Harness specifies the location of the final compiled binary.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      harness:
        binary: ./target
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Golang" %}
Harness specifies the method to fuzz, the package it resides in, and any build tags to use when compiling.

#### Usage:

```yaml
targets:
    - name: my-target
      harness:
        # the name of your method
        function: FuzzMe
        # build tags are optional
        build_tags: tag1 tag2 tag3
        # package specifies the package to import the
        # desired function from 
        package: github.com/x/y/z/a/b/c
        # checkout specifies where to check your code out
        # this repository will be placed in the directory:
        # ~/go/src/github.com/x/y/
        checkout: github.com/x/y
```
{% endtab %}

{% tab title="Binaries" %}
Harness specifies the location of the final compiled binary, as well as how the binary receives input. 

If the binary takes input from a file, mark the location in the binary's command line where the input file should be placed with a `@@`. This will be replaced automatically when fuzzing.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      harness:
        binary: ./target @@
        input: file # either stdin, socket or file
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="danger" %}
The `socket` input mode is coming soon, and is not currently available on the platform.
{% endhint %}
{% endtab %}
{% endtabs %}

### target.memory\_limit

**Type: `integer`**

**Required: no**

The maximum memory usage in megabytes to allow when processing a test case. Any tests that use more memory than this will be reported as bugs.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      memory_limit: 1000 # 1 GB
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.timeout

**Type: `integer`**

**Required: no**

The maximum amount of time in milliseconds the target should take to process any one test case.

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      timeout: 500 # 500 milliseconds
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### target.sanitizers

{% hint style="danger" %}
This field is only valid for targets written in or compiled from C and C++.
{% endhint %}

**Type: `Map`**

**Required: no**

**Valid values: `address`**

Sanitizers to use with this target. Sanitizers are analyzers that are compiled along with the target, and alert on specific issues.

Supported sanitizers:

* [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer): finds bugs including Use After Free, Buffer Over/Underflows, Use After Returns, Memory Leaks, etc
  * Fuzzbuzz sets the following defaults for ASAN\_OPTIONS \(these will override your settings\): `abort_on_error=1:print_scariness=1`
  * `symbolize=0` is set during runtime, and `symbolize=1` is set when reproducing crashes to gather stacktraces

#### Usage:

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
targets:
    - name: my-target
      sanitizers:
        # sanitizer options can be specified as a string
        address: detect_stack_use_after_return=1:debug=1
        # the field can also be left blank, e.g.:
        address: 
```
{% endcode-tabs-item %}
{% endcode-tabs %}

