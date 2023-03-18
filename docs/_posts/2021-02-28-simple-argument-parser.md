---
layout: post
title:  "A very simple argument parser"
author: "Tuna"
comments: false
category: Scripting
tags: python argument-parser lazy
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

I’m a guy who usually writes a lot of script files to automate my daily routine. To extend the flexibility of the script, a good piece of advice is to make the script accepts parameters passing from command<!--more-->.

There are tons of argument parser libraries out there, for every programming language we can imagine. It’s because the command-line application is mostly the 1st class resident of a programming language. I haven’t checked all of the libraries or dug deeply into a specific library (yes, I’m a lazy guy, too). I sometimes intended to apply some of the libraries for this purpose for my Python scripts (for example [argparse](https://docs.python.org/3/library/argparse.html)). However, I feel they are too complex for a lazy guy like me, and all of the time, I give up adopting a library and use raw argument array received from the environment instead (ie. `sys.argv` in Python).

After giving up too many times, I just figure out that I could make a simple one with very simple approaches:

{: .box-italic}
> 1. An argument parser parses the array of params only.
> 2. Supports getting an argument with position or with key for key-value.
> 3. Checks whether a key exists.

No more, no less!

The result of all of these approaches is this 14-LOC Python script:

{% highlight python linenos%}
class ArgumentParser:
    def __init__(self, args):
        self.positioned_arguments = list(filter(lambda x: not x.startswith('--'), args))
        self.named_argument_map = dict(map(lambda x: x.split("=", 1) if "=" in x else (x, None),
                                           filter(lambda x: x.startswith('--'), args)))

    def get(self, key_or_index):
        if type(key_or_index) is int:
            return self.positioned_arguments[key_or_index]
        else:
            return self.named_argument_map[str(key_or_index)]

    def has_key(self, key):
        return key in self.named_argument_map
{% endhighlight %}

Just a note: `positioned_arguments` doesn’t count key-value pair arguments. Key-value arguments can be seen as optional parameters.

### Conclusion
Do you need a long tutorial before starting using this parser?