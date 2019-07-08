---
description: >-
  This Getting Started guide will walk you through an end-to-end demo of the
  Fuzzbuzz platform, from setting up a project, all the way to finding and
  fixing a bug.
---

# Find your first bug in Go

## Step 1: Get the code

First, clone the tutorial code to your machine:

```bash
git clone https://github.com/fuzzbuzz/tutorial-go.git
```

## Step 2: Code Review

The repository contains a `fuzzbuzz.yaml` file, which is how Fuzzbuzz is configured, along with a couple of Go files, and a directory named `corpus`. This section will quickly walk you through all of these files.

### method.go

This file contains the method we want to test. It has a very basic bug that serves our purpose of demonstrating how the platform works.

{% code-tabs %}
{% code-tabs-item title="method.go" %}
```go
package tutorial

// BrokenMethod has a bug - it will try to read the 4th
// index of Data even when it only has a length of 3.
func BrokenMethod(Data string) bool {
    return len(Data) >= 3 &&
        Data[0] == 'F' &&
        Data[1] == 'U' &&
        Data[2] == 'Z' &&
        Data[3] == 'Z'
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### fuzz.go

This file contains `FuzzerEntrypoint`, the method that Fuzzbuzz will run repeatedly with the tests it generates. This method is very simple, as it just passes the raw test data through to `BrokenMethod`. To learn how to write more advanced tests, read our [Target Documentation ](../../developer-documentation/targets.md)page.

{% code-tabs %}
{% code-tabs-item title="fuzz.go" %}
```go
package tutorial

// FuzzerEntrypoint is the method Fuzzbuzz will repeatedly
// run with new tests to try and find bugs in BrokenMethod
func FuzzerEntrypoint(Data []byte) int {
    testData := string(Data)
    BrokenMethod(testData)
    return 0
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### corpus

This directory contains some initial tests \(the "test corpus"\) that can be provided as input to FuzzerEntrypoint, and don't cause it to fail. These initial cases give Fuzzbuzz an idea of what a valid test case looks like, which allows it to more efficiently learn how to generate interesting new tests.

### fuzzbuzz.yaml

This is how you provide your configuration to Fuzzbuzz. It allows you to install any dependencies, tells Fuzzbuzz what parts of your code to test, and allows you to configure more advanced constraints to make your code more efficient.

{% code-tabs %}
{% code-tabs-item title="fuzzbuzz.yaml" %}
```yaml
base: ubuntu:16.04
targets:
  - name: tutorial
    language: go
    version: "1.11"
    corpus: ./corpus
    harness:
      function: FuzzerEntrypoint
      # package defines where to import FuzzerEntrypoint from
      package: github.com/fuzzbuzz/tutorial
      # the repository will be cloned to
      # $GOPATH/src/github.com/fuzzbuzz/tutorial
      checkout: github.com/fuzzbuzz/tutorial
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This `fuzzbuzz.yaml` is very basic. It defines the base operating system to build and fuzz code in, and has configuration for a target named `tutorial`. Every target has a corresponding method or binary that it represents.

The target configuration defines the language and version to use, as well as the method to test, where to find and import it, and the initial test corpus. You can learn more about other configuration options by reading the [Target Documentation ](../../developer-documentation/targets.md)page.

You're all set! Head to the next page to set up the Fuzzbuzz tools.

