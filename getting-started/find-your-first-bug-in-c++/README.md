---
description: >-
  This Getting Started guide will walk you through an end-to-end demo of the
  Fuzzbuzz platform, from setting up a project, all the way to finding and
  fixing a bug.
---

# Find your first bug in C++

## Step 1: Get the code

First, clone the tutorial code to your machine:

```bash
git clone https://github.com/fuzzbuzz/tutorial-c.git
```

## Step 2: Code Review

The repository contains a `fuzz.yaml` file, which is how Fuzzbuzz is configured, along with a couple of C++ files, and a directory named `corpus`. This section will quickly walk you through all of these files.

### api.cpp

This file contains the method we want to test. It has a very basic bug that serves our purpose of demonstrating how the platform works.

{% code-tabs %}
{% code-tabs-item title="api.cpp" %}
```c
#include "api.h"

#include <vector>

// Do some computations with 'str', return the result.
// This function contains a bug. Can you spot it?
size_t BrokenMethod(const std::string &str) {
  std::vector<int> Vec({0, 1, 2, 3, 4});
  size_t Idx = 0;
  if (str.size() > 5)
    Idx++;
  if (str.find("foo") != std::string::npos)
    Idx++;
  if (str.find("bar") != std::string::npos)
    Idx++;
  if (str.find("ouch") != std::string::npos)
    Idx++;
  if (str.find("omg") != std::string::npos)
    Idx++;
  return Vec[Idx];
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### harness.cpp

This file contains `FuzzerEntrypoint`, the method that Fuzzbuzz will run repeatedly with the tests it generates. This method is very simple, as it just passes the raw test data through to `BrokenMethod`. To learn how to write more advanced tests, read our [Target Documentation ](../../developer-documentation/targets.md)page.

{% code-tabs %}
{% code-tabs-item title="harness.cpp" %}
```c
#include "api.h"

#include <string>

// Simple fuzz target for BrokenMethod().
extern "C" int FuzzerEntrypoint(const uint8_t *data, size_t size) {
  std::string str(reinterpret_cast<const char *>(data), size);
  BrokenMethod(str);
  return 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### corpus

This directory contains some initial tests \(the "test corpus"\) that can be provided as input to FuzzerEntrypoint, and don't cause it to fail. These initial cases give Fuzzbuzz an idea of what a valid test case looks like, which allows it to more efficiently learn how to generate interesting new tests.

### fuzz.yaml

This is how you provide your configuration to Fuzzbuzz. It allows you to install any dependencies, tells Fuzzbuzz what parts of your code to test, and allows you to configure more advanced constraints to make your code more efficient.

{% code-tabs %}
{% code-tabs-item title="fuzz.yaml" %}
```yaml
base: ubuntu:16.04
setup: # global setup steps here
environment: # global env vars here
targets:
- name: tutorial
  language: c++
  setup:
    - $FUZZ_CXX $CXXFLAGS -c ./api.h ./api.cpp
    - $FUZZ_CXX $CXXFLAGS ./api.o ./harness.cpp $FUZZ_ENGINE -o ./target
  harness: 
    binary: ./target
  sanitizers:
    address: detect_stack_use_after_return=1
  corpus: ./corpus
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This `fuzz.yaml` is very basic. It defines the base operating system to build and fuzz code in, and has configuration for a target named `tutorial`. Every target has a corresponding method or binary that it represents.

The target configuration defines the language and version to use, as well as the method to test, where to find and import it, and the initial test corpus. You can learn more about other configuration options by reading the [Target Documentation ](../../developer-documentation/targets.md)page.

You're all set! Head to the next page to set up the Fuzzbuzz tools.

