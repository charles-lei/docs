---
description: >-
  This Getting Started guide will walk you through an end-to-end demo of the
  Fuzzbuzz platform, from setting up a project, all the way to finding and
  fixing a bug.
---

# Find your first bug in Python

## Step 1: Get the code

First, clone the tutorial code to your machine:

```bash
git clone https://github.com/fuzzbuzz/tutorial-python.git
```

## Step 2: Code Review

The repository contains a `fuzz.yaml` file, which is how Fuzzbuzz is configured, along with a couple of Python files, and a directory named `corpus`. This section will quickly walk you through all of these files.

### demo.py

This file contains the method we want to test, `BrokenMethod`. It has a very basic bug that serves our purpose of demonstrating how the platform works.

It also contains `FuzzerEntrypoint`, the method that Fuzzbuzz will run repeatedly with the tests it generates. This method is very simple, as it just converts the bytes it receives to a string, and passes them to `BrokenMethod`. To learn how to write more advanced tests, read our [Target Documentation ](../../developer-documentation/targets.md)page.

{% tabs %}
{% tab title="Python 2" %}
```python
def BrokenMethod(strInput):
    if len(strInput) >= 1:
        return strInput[0] == 'F' and strInput[1] == 'U'

def FuzzerEntrypoint(Data):
    try:
        strData = Data.read().decode("utf-8")
        BrokenMethod(strData)
    except UnicodeDecodeError, e:
        return
```
{% endtab %}

{% tab title="Python 3" %}
```python
def BrokenMethod(strInput):
    if len(strInput) >= 1:
        return strInput[0] == 'F' and strInput[1] == 'U'

def FuzzerEntrypoint(Data):
    try:
        strData = str(Data.read(), "utf-8")
        BrokenMethod(strData)
    except UnicodeDecodeError:
        return
```
{% endtab %}
{% endtabs %}

### corpus

This directory contains some initial tests \(the "test corpus"\) that can be provided as input to FuzzerEntrypoint, and don't cause it to fail. These initial cases give Fuzzbuzz an idea of what a valid test case looks like, which allows it to more efficiently learn how to generate interesting new tests.

### fuzz.yaml

This is how you provide your configuration to Fuzzbuzz. It allows you to install any dependencies, tells Fuzzbuzz what parts of your code to test, and allows you to configure more advanced constraints to make your code more efficient.

```yaml
base: ubuntu:16.04
language: python
targets:
  - name: tutorial-python2
    version: "2.7"
    harness:
      function: FuzzerEntrypoint
      file: python2/demo.py
  - name: tutorial-python3
    version: "3.6"
    harness:
      function: FuzzerEntrypoint
      file: python3/demo.py
```

This `fuzz.yaml` is very basic. It defines the base operating system to build and fuzz code in, and has configuration for a target named `tutorial`. Every target has a corresponding method or binary that it represents.

The target configuration defines the language and version to use, as well as the method to test, where to find and import it, and the initial test corpus. You can learn more about other configuration options by reading the [Target Documentation ](../../developer-documentation/targets.md)page.

You're all set! Head to the next page to set up the Fuzzbuzz tools.

