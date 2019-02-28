# Overview

## Integrates into your DevOps Pipeline

Fuzzbuzz integrates with source-control and CI tools like GitHub, Jenkins and CircleCI, to ensure that the latest version of your code is always being tested. Whenever code is updated, Fuzzbuzz automatically checks to see if bugs have been fixed, so developers don't have to spend time closing out bug reports.

In addition, Fuzzbuzz plays nicely with existing open-source fuzzing tools: if your code already works with AFL, Gofuzz, Libfuzzer, or other popular frameworks, it takes less than 10 minutes to get set up. [Learn How.](../developer-documentation/porting-targets-to-fuzzbuzz/)

## Simple, flexible test framework

Fuzzbuzz generates the tests, so developers don't have to spend their time dreaming up edge cases. The flexible interface allows developers to write a wide variety of tests that can all take advantage of Fuzzbuzz's platform. Additionally, Fuzzbuzz uses a modified version of the [open-source fuzzer AFL](http://lcamtuf.coredump.cx/afl/)'s algorithm, which has a [proven track record](http://lcamtuf.coredump.cx/afl/#bugs) of finding hundreds of bugs and vulnerabilities.

[Click Here](../developer-documentation/targets.md) to learn how Fuzzbuzz's API makes it simple to write robust, flexible automated tests.

## Code Coverage Information

Code coverage is one of the most important metrics to track when testing code. Fuzzbuzz automatically generates code coverage reports so you can see, at a glance, the extent to which your code is being tested.

## Automated Alerting

Fuzzbuzz automatically sends out alerts when a new bug is found, and gets out of your way when there's nothing new to report. Integrations with Slack, Jira, GitHub, and other developer tools allow you to track bugs your way, and don't force you to adapt to a new workflow.

## Reporting

Fuzzbuzz provides statistical reports of how your software and developers are performing, so you can quickly view historical data of your software's performance, code coverage improvements, and lead-time on bugfixes.

## Intelligent Bug Management

The [Fuzzbuzz Command-Line Interface](../reference/cli.md) provides bug-management functionality to speed up the time taken to fix a bug, including tools to analyze bugs on a local machine, and run regression tests to ensure that fixed bugs don't return in later versions of your code.

