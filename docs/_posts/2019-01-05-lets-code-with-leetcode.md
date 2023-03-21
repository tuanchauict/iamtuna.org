---
layout: post
title:  "Let's code with Leetcode"
author: "Tuna"
comments: false
category: Leetcode
tags: leetcode lazy python
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

Mmm…, I don’t know how to start this post, actually, it’s not about Leetcode but about my laziness (yes, very similar to the story about [git naming](/2018-11-02/short-store-of-naming-git-branch))<!--more-->.

I’m practicing my algorithm skills with Leetcode and I’m a lazy coder on every aspect. This time is about writing test cases for testing my code. For instance, let’s get the question [#7. Reverse Integer](https://leetcode.com/problems/reverse-integer/) as an example. 

> Given a signed 32-bit integer `x`, return `x` with its _digits reversed_. If reversing `x` causes the value to go outside the signed 32-bit integer range `[-231, 231 - 1]`, then return `0`.

After writing a solution like this:

```python
def sol(x):
    intmax = 2 ** 31 - 1
    intmin = -2 ** 31
    is_positive = x >= 0
    x = abs(x)
    result = 0
    while x > 0:
        pop = x % 10
        pop = pop if is_positive else -pop
        x //= 10
        result = 10 * result + pop
    if result > intmax or result < intmin:
        return 0
    return result
```

I would usually write test code like this

```
print sol(1)
print sol(12)
print sol(123456789)
```

and hit `run`

When the number of test cases reaches 5 or more, it’s hard to recognize which result goes with which input (yes, we have to move the eye from console to the code editor).

For solving parameters vs result problem, I came up with a simple solution:

```python
def test(fun, *params):
    print params, fun(*params)
```

and the test code would be:

```python
test(sol, 1)          # 1 1
test(sol, 12)         # 12 21
test(sol, 123456789)  # 123456789 987654321
```

Easier, right? Yeah, but not so good. First, I have to repeat `test` and also `sol` every time a new test case added. I’m lazy and I don’t accept that. Second, the output is messive. I don’t like this mess.

The second problem is easier to fix with just a little effort for formatting.

```python
def test(func, params, expected=None):
    format = '%10s ' * len(params)
    param_text = format % params
    result = func(*params)
    if expected is not None:
        correct = result == expected
        print "%s -->  %-10s  |  %s" % (param_text, result, correct)
    else:
        print "%s -->  %s" % (param_text, result)
```

With this solution, I added the expected value to check whether the test case result was correct but I still have to repeat `test` and `sol` . Besides, there is a hardcoded value for formatting parameters: `%-10s` . What if the parameter is very long or very short? Not so good at all. We are only able to solve the hardcoded problem only if we know all the parameters of all test cases before printing out the result.

```python
def test(func, testcases):
    ...
```
The above code was an idea in my mind. However, it has never been implemented because I then recalled Python’s `decorator` . Here is my final solution:

{% highlight python  linenos %}
def testcase(*testcases):
    def build_format():
        max_param_len = 0
        max_result_len = 0
        num_params = 0
        for case in testcases:
            num_params = len(case) - 1
            for param in case[1:]:
                str_param = str(param)
                max_param_len = max(max_param_len, len(str_param))
            max_result_len = max(max_result_len, len(str(case[0])))
        
        param_format = ("%" + "%ds" % (max_param_len + 2)) * num_params
        result_format = "%s  -->  %-" + "%ds" % (max_result_len + 2) + "  |  %s"
        return param_format, result_format
    
    def deco(fun):
        def wrapper():
            param_format, result_format = build_format()
            
            for case in testcases:
                expected = case[0]
                params = case[1:]
                result = fun(*params)
                
                param_text = param_format % params
                
                test_result = 'UNCHECKED' if expected is None else 'PASSED' if result == expected else 'FAILED'
                
                print result_format % (param_text, result, test_result)
        
        return wrapper
    
    return deco
{% endhighlight %}

and my test code will be:

```python
@testcase(
    (1, 1), # (expected_value, param0, param1,...)
    (21, 12),
    (987654321, 123456789),
)
def sol(x):
    ...
    return y
sol() # <--- I want to remove this too
```

Phew!!!

## Updates
### 2019-04-02

Finally, I could remove the last method call on the above sample code which I intended to remove

```python
sol() # <--- I want to remove this too
```

My final `testcase` is here. I also added `log()` for printing debug logs during testing.

{% highlight python linenos %}
_config_ = {
    'log': True
}

def testcase(*testcases, **kwargs):
    if 'log' in kwargs:
        _config_['log'] = kwargs['log']
    else:
        _config_['log'] = len(testcases) == 1
    
    def build_format():
        max_param_len = 0
        max_result_len = 0
        num_params = 0
        for case in testcases:
            num_params = len(case) - 1
            for param in case[1:]:
                str_param = str(param)
                max_param_len = max(max_param_len, len(str_param))
            max_result_len = max(max_result_len, len("%s" % case[0]))
        
        param_format = ("%" + "%ds" % (max_param_len + 2)) * num_params
        result_format = "%s  -->  %-" + "%ds" % (max_result_len + 2) + "  |  %s"
        return param_format, result_format
    
    def deco(fun):
        import sys
        import StringIO
        
        def wrapper():
            param_format, result_format = build_format()
            
            for case in testcases:
                expected = case[0]
                params = case[1:]
                param_text = param_format % params
                output = StringIO.StringIO()
                if not _config_['log']:
                    sys.stdout = output
                result = fun(*params)
                sys.stdout = sys.__stdout__
                
                if expected is None:
                    test_result = 'UNCHECKED'
                else:
                    is_passed = result == expected
                    test_result = 'PASSED' if is_passed else 'FAILED: expected %s' % expected
                    
                print result_format % (param_text, result, test_result)
                if test_result != 'PASSED':
                    print output.getvalue(),
                output.close()
        
        if kwargs.get('run', True):
            print "Test result for %s:" % fun.__name__
            return wrapper()
        else:
            return wrapper
    
    return deco
{% endhighlight %}

After using this, I just need to add `@testcase(...)` and then hit run. No more `sol()` call. I also added a configuration for autorun a method or not in case we implement more than a solution within a file.

### 2022-03-21

I decided to make the repository for leetcode test public today. You can check the final version of the `@testcase` from [here](https://github.com/tuanchauict/lctest). The `testcase` in this repository is more advanced than the code I shared in this note and the repo also has more stuffs for leetcode practicing.

Actually, you can also able to install `lctest` package with `pip`. Since I haven't finalized the APIs of the repo yet, I haven't written a document for it yet. By the way, since you are a developer, it's quite easy for you to see what is inside `lctest`.

Happy leetcoding!
