slidenumbers: true

# Git
## A Peek Under the Hood

### Clara Bennett
### PyCon 2016

![](images/vasa.jpg)

---
# My Goal

_To improve your mental model of **git** and give you tools to continue learning and become a more effective git user._


This talk is not:

* An introduction to git for novice users
* A tutorial in "advanced" git techniques
* Related to **GitHub** in any way

---

# What is stored when you commit?

![](images/dunes.jpg)

---
# The Snapshot
## (i.e. the content)

* When changes are staged, each affected file and directory entity is check-summed, compressed, and stored separately[^1]
* Directory entities (including the project root) point to the SHA-1 checksums of the files and directories they contain


[^1]: Eventually, git will compress versions of the same file together to save space when necessary, e.g. if you want to push to a remote. But looking up the file by checksum will still return you the complete file.

---

# The Commit
## (i.e. content + meta-data)

* Pointer a top-level directory snapshot[^2]
* Commit message
* Author and committer
* Pointer(s) to parent commit(s)

[^2]: Thus, commit objects have access to the _complete state_ of the project at that point in history.

---

![inline fit](images/two-commits-terminal.png) ![inline fit](images/two-commits.png)

---
![right fit](images/mv-example.png)

# Representing Changes

* Git does not directly save any _actions_ that you took, only the state
* Differences are _derived_ by comparing snapshots
* Actions are _inferred_
* Ex: git recognizes the rename because the file contents is the same

---

# Branches, History, and Navigation

![](images/trees.jpg)

---
# The Commit "Tree"[^3]

* As previously mentioned, commits know about their parents
* Together, the commits and parent relations form the commit "tree", or history
* Multiple commits can have the same parent, which forms a natural "branching" structure

[^3]: Technically, it's not quite a tree, because merge commits have two or more parents. But, it seems easier to think about as an almost-tree than as a rooted connected directed acyclic graph. :sweat_smile:

---
# Branches Are Just Pointers

* A git branch is represented as a reference to a commit (which defines the "end" of the branch)
* The branch reference moves forward if new commits are added[^4] while that branch is checked out
* Deletion of a branch amounts to deletion of _the pointer only_: the commits are still in the database

[^4]: This is in contrast to tags (similarly just pointers to commits), which stay put unless explicitly moved.

---
# Where am I?

* The `HEAD` reference determines what is "checked out"
* If a branch is checked out, `HEAD` points to the branch ref
* The "unattached HEAD" state occurs when `HEAD` points directly to a commit
* Either way, the associated snapshot is identical[^5] to the state of the working directory

[^5]: Assuming that the working directory is clean, that is.

---
```
[~/dev/repo]$ git clone url .
```

![inline fit](images/branches1.png)

---
```
[~/dev/repo](master)$ git checkout -b topic
```

![inline fit](images/branches2.png)

---
```
[~/dev/repo](topic)$ git commit
```

![inline fit](images/branches3.png)

---
```
[~/dev/repo](topic)$ git commit
```

![inline fit](images/branches4.png)

---
```
[~/dev/repo](topic)$ git checkout master; git pull
```

![inline fit](images/branches5.png)

---
## @csojinb


![](images/young-clara-sunglasses.jpg)
