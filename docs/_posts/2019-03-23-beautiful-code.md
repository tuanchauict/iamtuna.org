---
layout: post
title:  "Beautiful code"
author: "Tuna"
comments: false
category: Code review
tags: code-review
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

Some days ago, my friend requested me to help him review his code. Well, it was my daily job, so, I pleased to help<!--more-->.

> *Disclaim: Sample code is written in TypeScript, however, I’m not familiar with the language. So, correct me if I’m wrong.*

Here is the code:

{% highlight typescript linenos %}
function getSrcTokensAndDestTokens(network) {
    const tokenInputs = this.srcTokens.concat(this.destTokens);
    
    for (const token of tokenInputs) {
        if (network.tokens[token].hidden || network.tokens[token].delisted) {
            console.log(`GSI doesn't support ${token} yet!`);
            return;
        }
    }
    
    if (this.srcTokens.length === 0 && this.destTokens.length === 0) {
        console.error("srcTokens and destTokens can't be both empty!");
        return;
    }
    
    if (this.srcTokens.length === 0) {
        for (const token in network.tokens) {
            if (!network.tokens[token].hidden && !network.tokens[token].delisted) {
                this.srcTokens.push(token);
            }
        }
    }
    
    if (this.destTokens.length === 0) {
        for (const token in network.tokens) {
            if (!network.tokens[token].hidden && !network.tokens[token].delisted) {
                this.destTokens.push(token);
            }
        }
    }
    // ...
    return {
        // ...
    }
}
{% endhighlight %}

What’s wrong with this code?

This method implementation is long, let’s break it aparts first, so, we can talk on each piece of the function.

## 1
```ts
function getSrcTokensAndDestTokens(network)
```

What is `src` `dest` ? I guess they are `source` and `destination` respectively. But we shouldn’t do that, we shouldn’t guess. Or, we shouldn’t make readers guess what did we want to say. So, let’s change the function name

```ts
function getSourceTokensAndDestinationTokens(...)
```
or
```ts
function getSourceAndDestinationTokens(...)
```

(Update: if you feel `Destination` long, we can use `Target` )

Next, `network` may be okay, however, it’s too common. Is it the internet network? or LAN network? or personal network? or Umbala Network? In this case, we should give it a more specified name. Maybe `gsiNetwork` .


## 2
```ts
const tokenInputs = this.srcTokens.concat(this.destTokens);
```
Yes, we should use `source` and `destination`.

## 3
```ts
for (const token of tokenInputs) {
    if (network.tokens[token].hidden || 
        network.tokens[token].delisted) {
        console.log(`GSI doesn't support ${token} yet!`);
        return;
    }
}
```

Mmm… we have a `for-loop` and an `if` for checking error here. `getSrcTokensAndDestTokens` is a caller, we should simplify the error check within only an if for that purpose. Something like
```ts
if (!isValidTokenInputs) {
    return
}
```

Before refactoring, let’s take a look into the loop’s body. `network.tokens[token]` is called twice. Besides, `hidden` and `delisted` are not a good name for boolean values. Let’s refactor

```ts
const token = network.tokens[tokenInput] // #1
if (token.isHidden || token.isDelisted) {
    // ...
}
// Note: 
// #1: we should use a similar name to the loop iterator, here is 
// `tokenInputs`, therefore, we should name `tokenInput` instead of
// `token`
```

Next

```ts
console.log(`GSI doesn't support ${token} yet!`);
```

This log is hard to search when we have tons of logs. Why? Because `${token}` is in the middle. The output log looks like:

```
GSI doesn't support KJHKJKY4edf5ljbjdbLLBLHJBERB yet!
GSI doesn't support ljdilhKHKJBJHJGVKUYIlJNKJNBH yet!
GSI doesn't support KJHJK yet!
```

I also think it’s not grammar correct (maybe I’m wrong). We shouldn’t hardcode too. It’s should be

```ts
const ERROR_LOG_NOT_SUPPORTED_TOKEN = "Token not supported" // #1
// ...
console.log(ERROR_LOG_NOT_SUPPORTED_TOKEN, token)
// Note:
// #1: We may add ERROR CODE
```

Okay, let’s make the `for-loop` fancier

```ts
const ERROR_LOG_NOT_SUPPORTED_TOKEN = "Token not supported"
function isValidTokenInputs(network, tokenInputs) {
    for (const tokenInput in tokenInputs) {
        const token = network.tokens[tokenInput]
        if (token.isHidden || token.isDelisted) {
            return [false, tokenInput] // We should create a type instead of using an array or
                                       // a pair because it looses context information
        }
    }
    
    return [true, ""]
}
function getSrcTokensAndDestTokens(network) {
    // ...
    const [isSourceTokenValid, wrongSourceTokenInput] = 
            isValidTokenInputs(network, tokenInputs)
    if (!isSourceTokenValid) {
        console.log(ERROR_LOG_NOT_SUPPORTED_TOKEN, wrongSourceTokenInput)
        return 
    }
    //...
}
```

Isn’t it easier to read? I hope so

## 4
```ts
if (this.srcTokens.length === 0 && this.destTokens.length === 0) {
    console.error("srcTokens and destTokens can't be both empty!");
    return;
}
```
Well, the same for the hardcoded error log. We should have a constant for searching not only inside the log file but also within the code. Name it!

## 5
```ts
if (this.srcTokens.length === 0) {
    for (const token in network.tokens) {
        if (!network.tokens[token].hidden && 
            !network.tokens[token].delisted) {
            this.srcTokens.push(token);
        }
    }
}
if (this.destTokens.length === 0) {
    for (const token in network.tokens) {
        if (!network.tokens[token].hidden && 
            !network.tokens[token].delisted) {
            this.destTokens.push(token);
        }
    }
}
```
Firstly, these two if-s do the same thing. Secondly, we have depth nested block here. Yeah, for me, 3 is too many. Finally, can you spot these two logics are the same

```ts
// #1
network.tokens[token].hidden || network.tokens[token].delisted
// #2
!network.tokens[token].hidden && !network.tokens[token].delisted
```

I hope you can see it. Here is the discrete mathematics transformation rule

```
A | B = !!(A | B) = !(!A & !B)
```

Follow DRY rule, we should define a method to check a token is valid or not

```ts
function isTokenValid(token) {
    return !token.isHidden && !token.isDelisted
}
```

The DRY rule also helps use to unify the two if-s into one function. (This writing is long and I’m lazy to type more explanation. Sorry!)

```ts
function getValidTokens(tokens) {
    const result = []
    for (const token in tokens) {
        if (isTokenValid(token) { 
            result.push(token)
        }
    }
}
```
And the two if-s will be

```ts
const validNetworkTokens = getValidTokens(network.tokens) // #1
if (this.sourceTokens.length === 0) {
    this.sourceTokens = validNetworkTokens // #2
}
if (this.destinationTokens.length === 0) {
    this.destinationTokens = validNetworkTokens // #2
}
// NOTE:
// #1: We can optimise by checking whether we need to create the 
// `validNetworkTokens` with check there is at lest a length === 0 
// #2: We may need to clone the `validNetworkTokens` to avoid same 
// reference problem
```

## 6
It’s your task to combine all of the above code into the final solution :)

Happy coding!!!
