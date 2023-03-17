---
layout: post
title:  "A short story of naming git-branch from a lazy coder"
author: "Tuna"
comments: false
category: Scripting
tags: git lazy python bash
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

![git branch](/images/2018-11-02/git-cover.webp "git branch")

Well, I’m a developer. Using **git** is somehow my daily job. I’m not very keen on typing command on the console, especially with long commands. In my previous company, we didn’t create a branch for each feature, just `dev` or `master` branch for all (of course, `release-xxx` branch for releasing)<!--more-->. After joining the current company and feature branches are mandatory, I start seeing my bad on this. First, I use UI for switching between branches. My tools go from Android Studio (which I mainly use for Android dev) to SourceTree (which I have known for a long time) and to Fork (a new UI git tool). I’m happy with Fork.

With the support from UI tools, I usually created very long branch name like

```bash
git checkout -b ABC0123-fix-for-wrong-stuffs-someone-did-wrongly
```

(which `ABC0123` is feature ID). However, whenever I use the console to switch between branches, it doesn’t work well although it has quite descriptive information. I decided to change, not that long branch name anymore. No more `-fix-for-wrong-stuffs-that-someone-did-wrongly` , the new branch naming will go with just

```bash
git checkout -b ABC0123
```
or

```bash
git checkout -b ABC0123-fix
```

Well, with the new naming, I can switch branches easier but I don’t know what is a branch for. I have to do one more check: copy the branch name, look up in the feature dictionary (JIRA). I’m a super lazy dev, and also lack of short-term memory ability. After a day or two, I forget.

Hmm, I’m lazy and I’m in a dilemma of branch naming. Long names contain enough information but require typing very much. Short names are easy to type but lack of information.

I need to change once more. My requirement is: a short name with information.

First approach: create my own script for saving and reading the description with just branch name. Here is my code:

{% highlight python linenos %}
# git-branch-info.py
import sys, json, os.path
def read_file(filepath):
    if not os.path.exists(filepath):
        return {}
    with open(filepath) as file:
        return json.loads(file.read())

def get_branch_info(filepath, git_path, branch):
    tree = read_file(filepath)
    git_tree = tree.setdefault(git_path, {})
    return git_tree.setdefault(branch, 'Not found')

def set_branch_info(filepath, git_path, branch, info):
    tree = read_file(filepath)
    git_tree = tree.setdefault(git_path, {})
    git_tree[branch] = info
    with open(filepath, 'w') as file:
        file.write(json.dumps(tree))

def main(args):
    filepath = args[1]
    git_path = args[2]
    branch = args[3]
    if args[0] == 'get':
        branch_info = get_branch_info(filepath, git_path, branch)
        print(branch_info)
    else:
        info = args[4]
        set_branch_info(filepath, git_path, branch, info)

if __name__ == '__main__':
    main(sys.argv[1:])
{% endhighlight%}

With this python script, I created bash files for automatically getting such required information

{% highlight bash linenos %}
# git-branch-info.sh
infoFile="$HOME/.git-branch-info.json"
gitPath=`git rev-parse --show-toplevel`
workingBranch=`git symbolic-ref --short HEAD`
command=${1:-'get'}
python3 $(dirname "$0")/git-branch-info.py "$command" $infoFile   
        $gitPath $workingBranch "$2"

# git-checkout-with-info.sh
git checkout $1
sh $(dirname "$0")/git-branch-info.sh

# git-new-branch-with-info.sh
git checkout $1
sh $(dirname "$0")/git-branch-info.sh new "$2
{% endhighlight%}

Yes, I was happy with this script. Very smart! Hahaha.

Soon, I realized that how stupid I was (just 5 minutes after pushing those files). I remembered that we could set a description to a branch:

```bash
git config branch.<branchName>.description <description>
```

and also get the description:

```bash
git config branch.<branchName>.description
```

I came up with a new solution: using branch description for saving and reading branch information. Luckily, I just knew how to use `git alias` a couple days ago. So, I could embed the new commands into aliases. Here is my final solution:

```bash
to = !git checkout $1 && git config branch.$1.description && :
new = !git checkout -b $1 && 
       git config branch.$1.description \""${2:-'Not set'}"\" && :
desc = !git config branch.`git symbolic-ref --short        
        HEAD`.description && :
setdesc = !git config branch.`git symbolic-ref --short 
           HEAD`.description \""$1"\" && :
```

Usage:

```bash
git to branchname # switch branch and print description
git new branchname "a description" # create a new branch with 
                                     description
git desc
git setdesc "a description"
```

![first result](/images/2018-11-02/git-bash-result-1.webp)

For listing all having description branches:

```bash
branches=`git branch --list`
for branch in $branches; do
    cleanBranchName=${branch//\*\ /}
    description=`git config branch.$cleanBranchName.description`
    [ -z "$description" ] || printf "%-15s %s\n" "$branch" "$description"
done
```

Phew!

---

## Update
### 2019/05/03
I add a new bash code for no branch name required as below.
{% highlight bash linenos %}
allBranches=`git branch`
word=0123456789abcdefghijklmnopqrstuvwxyz
for i in $(seq 1 ${#word})
do
    newBranch=$1${word:i-1:1}
    if [[ $allBranches != *$newBranch*  ]]; then
        echo "-> $newBranch"
        git checkout -b $newBranch
        git config branch.$newBranch.description \""${2:-'Not set'}"\"
        exit 0
    fi
done
echo "Fail. Please clean up or choose another prefix"
{% endhighlight %}

and set the alias for git in `~/.gitconfig`

```bash
new = !sh ~/path/to/script.sh \""$@"\" && :
```

With the new script, we just need to type:

```bash
git new <prefix> [description]
```

The branch name will be a combination of prefix and one of the characters in `[0–9a-z]`.

### 2019/05/13
New script for naming in sequence (similar to above but in reverse order)

{% highlight bash linenos %}
allBranches=`git branch`
word=0123456789abcdefghijklmnopqrstuvwxyz
wlen=${#word}
for i in $(seq ${wlen-1} -1 1)
do
    newBranch=$1${word:i-1:1}
    preBranch=$1${word:i-2:1}
    if [[ $allBranches == *\ $preBranch* ]] && [[ $allBranches != *\ $newBranch*  ]] ; then
        echo "-> $newBranch"
        git checkout -b $newBranch
        git config branch.$newBranch.description "${2:-Not set}"
        exit 0
    fi
done
echo "Fail. Please clean up or choose another prefix"
{% endhighlight %}

### 2019/05/22

I found a bug for the 2019/05/13’s script that it cannot create a new branch if there are no branches matching `prefixN` which N is a character from `word` . Correct: change the if condition to:

```bash
[[ ($allBranches == *\ $preBranch*  || $i == 1) && $allBranches != *\ $newBranch* ]]
```
{% highlight bash linenos %}
Full script:

allBranches=`git branch`
word=0123456789abcdefghijklmnopqrstuvwxyz
wlen=${#word}
for i in $(seq ${wlen-1} -1 1)
do
    newBranch=$1${word:i-1:1}
    preBranch=$1${word:i-2:1}
if [[ ($allBranches == *\ $preBranch*  || $i == 1) && $allBranches != *\ $newBranch* ]] ; then
        echo "-> $newBranch"
        git checkout -b $newBranch
        git config branch.$newBranch.description "${2:-Not set}"
        exit 0
    fi
done
echo "Fail. Please clean up or choose another prefix"
{% endhighlight %}