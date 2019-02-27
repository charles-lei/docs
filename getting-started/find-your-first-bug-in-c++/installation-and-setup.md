---
description: >-
  This Getting Started guide will walk you through an end-to-end demo of the
  Fuzzbuzz platform, from setting up a project, all the way to finding and
  fixing a bug.
---

# Installation & Setup

## Step 3: Sign up for Fuzzbuzz

Head to [https://app.fuzzbuzz.io/](https://app.fuzzbuzz.io) and follow the instructions to sign up. Once you see this page, you're all set!

![](../../.gitbook/assets/screen-shot-2019-02-06-at-2.11.35-pm.png)

## Step 4: Download the CLI

Fuzzbuzz provides a CLI that makes it easy to build & test your targets locally before pushing code to the platform. It also allows you to reproduce bugs found on the platform, so you can test your fixes immediately.

{% tabs %}
{% tab title="Mac OSX" %}
Use the following command to download the CLI, and then give it the right permissions and save it somewhere in your path.

```bash
curl -O https://app.fuzzbuzz.io/releases/cli/0.0.1/osx/fuzzbuzz
```
{% endtab %}

{% tab title="Linux" %}
Use the following command to download the CLI, and then give it the right permissions and save it somewhere in your path.

```bash
wget https://app.fuzzbuzz.io/releases/cli/0.0.1/linux/fuzzbuzz
```
{% endtab %}
{% endtabs %}

To make sure everything works, run:

```bash
fuzzbuzz auth login
```

If after entering your credentials, you see `Successfully logged in!`, you're all set up. Head to the next page to use the CLI and platform to find and fix your first bug.

