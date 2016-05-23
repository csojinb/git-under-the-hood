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

![](images/trees.jpg)

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

## @csojinb


![](images/young-clara-sunglasses.jpg)
