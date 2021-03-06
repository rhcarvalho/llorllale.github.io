---
layout: post
title: Golang - Optional Arguments for APIs
excerpt: Functional arguments are cool but add unnecessary complexity.
date: 2019-08-18
author: George Aristy
tags:
- go
- golang
- design-patterns
- api
---

I was recently directed towards Dave Cheney's article [*Functional options for friendly
APIs*](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) where he shares his
thoughts on designs for optional parameters in APIs. Dave ends with a proposal for *functional*
arguments that are optionally passed to a type constructor. There is no question this design is
superior to having a single constructor with lots of arguments.

However:

Dave's design is overkill for 99% of use cases and imposes an unnecessary tax on both the maintainer
and the consumer of these APIs.
{: .notice}

Developers integrating with these APIs are *consumers*, so are readers (aka. code reviewers).

## My proposal

A *simpler* alternative: two constructors, one is default, the other accepts a *config* struct.

Here is my proposed design for Dave's constructors in [term](https://github.com/pkg/term):

{% highlight go %}
package term

// I identified just three options after a quick scan of the README:
// Baud rate, and either CBreakMode or RawMode.
type Options struct {
    CBreakMode bool  // Defaults to RawMode if false
    Baud               int
}

func Default(name string) (*Term, error) {...}

func Custom(name string, options Options) (*Term, error) {...}
{% endhighlight %}

## What we gain...

### In terms of usage

Decreased verbosity: occurrences of the symbol `term` is decreased. The magnitude of this benefit
increases linearly with the number of optional parameters:

{% highlight go %}
package consumer

import "github.com/pkg/term"

func DaveDesign() {
    // default
    term, err := term.Open("/dev/ttyUSB0")

    // custom
    term, err := term.Open(
        "/dev/ttyUSB0",
        term.Speed(57600),
        term.CBreakMode,
    )
}

func MyDesign() {
    // A ctor named 'Default' immediately conveys the possibility of
    // customization to a consumer
    term, err := term.Default("/dev/ttyUSB0")

    // custom
    term, err := term.Custom(
        "/dev/ttyUSB0",
        term.Options{
            Baud:               57600,
            CBreakMode: true,
        }
    )
}
{% endhighlight %}

### In terms of maintenance

Decreased number of unit tests: reducing the set of options to a [value object](https://en.wikipedia.org/wiki/Value_object) renders tests for them *needless*.

## What we lose...

### In terms of usage

Nothing as far as I can see.

The symbol `Default` clearly signals the possibility of custom `Term`s such that a developer
consuming this API would seek out alternatives if required. This means this design has no added
confusing aspects.

### In terms of maintenance

N/A. We *improve* maintainability by reducing the number of artifacts we need to test.

Any validations and/or computations can be extracted unto their own functions (whether static or
member functions) of the constructor's type.
