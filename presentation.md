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
* A deep dive into git internals or "plumbing" commands

---

# What is stored when you commit?

![](images/dunes.jpg)

---
# The Staging Index
## (i.e. proposed next commit)

* The staging index is a place to build the next commit
* On checkout, the staging index is populated to reflect the state of the checked-out commit
* As new changes are staged, the index is updated
* On commit, the index becomes the commit snapshot

---
# The Snapshots
## (i.e. the content)

* To store staged changes, each affected file and directory entity is check-summed and saved separately
* These "snapshots" are efficiently stored copies of entire files[^1], or pointers to snapshots of contained files
* Snapshots can be recovered by SHA-1 checksum

[^1]: Eventually, git will compress versions of the same file together to save space when necessary, e.g. if you want to push to a remote. But looking up the file by checksum will still return you the complete file.

---
# The Commit
## (i.e. content + meta-data)

* Content: pointer to project root snapshot[^2]
* Meta-Data: Commit message, author and committer
* History: pointer(s) to parent commit(s)
    - `<commit>^X` accesses the X'th parent
    - `<commit>~X` accesses the X'th-gen ancestor

[^2]: This snapshot is built from the staging index. Thus, a commit has access to the _complete project state_ at that point in history.

---
## Visualize commit storage

![inline fit](images/two-commits.png)

---
## Example: Understanding `git reset` options

`git reset [<mode>] HEAD~1`

* **mode = soft:** moves branch back to first parent; index and working directory unchanged _(changes remain "staged")_
* **default (mode = mixed):** moves branch back to parent; staging index reset to parent _(changes "unstaged")_
* **hard:** moves branch back to parent; index and working directory reset _(commit and changes erased entirely)_

---
![right fit](images/mv-example.png)

# Representing Changes

* Git does not directly save any _actions_ that you took, only the state
* Differences are _derived_ by comparing snapshots
* Actions are _inferred_
* Example (right): git recognizes the rename because the file contents is the same, equivalent to `git mv`

---
## Important implication!

* Git's ability to track a file's history[^3] depends on the file being recognizably the same file between commit snapshots
* i.e. the following may break the file history:
    `$ git mv file.py other.py`
    _<make lots of changes to `other.py`>_
    `$ git add --all; git commit`

[^3]: You may be taking advantage of this feature in, say, GitHub, even if you aren't using it locally.

---

# Branches, History, and Navigation

![](images/trees.jpg)

---
# The Commit "Tree"[^4]

* As previously mentioned, commits know about their parents
* Together, the commits and parent relations form the commit "tree", or history
* Multiple commits can have the same parent, which forms a natural "branching" structure

[^4]: Technically, it's not quite a tree, because merge commits have two or more parents. But, it seems easier to think about as an almost-tree than as a rooted connected directed acyclic graph. :sweat_smile:

---
# Branches Are Just Pointers

* A git branch is represented as a reference to a commit (which defines the "end" of the branch)
* The branch reference moves forward if new commits are added[^5] while that branch is checked out
* Deletion of a branch amounts to deletion of _the pointer only_: the commits are still in the database

[^5]: This is in contrast to tags (similarly just pointers to commits), which stay put unless explicitly moved.

---
# Where am I?

* The `HEAD` reference determines what is "checked out"
* If a branch is checked out, `HEAD` points to the branch ref
* The "unattached HEAD" state occurs when `HEAD` points directly to a commit
* Either way, the associated snapshot is identical[^6] to the state of the working directory

[^6]: Assuming that the working directory is clean, that is.

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
# The Reflog
## Understand your actions and get out of trouble

---
# What is the reflog?

* The reflog is a **local-only** log of all changes to git _ref(erence)s_, which are pointers to git objects
    - Refs include: branches, tags, HEAD
* A freshly cloned repo has an empty reflog
* The reflog can help you understand how your actions affect the commit tree
* The reflog can be used to **return to a previous state**

---
# What can I find in the reflog?

* Some of the changes recorded in the reflog:
    - new commits (including merge commits, cherry-picks)
    - modifications to commits
    - branch or commit checkouts
* Things that are not recorded in the reflog:
    - fetches
    - pushes to a remote

---
# View the reflog

![left inline fit](images/reflog.png) ![right inline fit](images/git-log-g.png)

---
# Example
## Recover a modified commit

![right fit](images/amend-example.png)

* Recall that commit "modification" actually creates a _new_ commit
* Previous "version" still exists
* Can create new branch to point to old commit:
`$ git branch recovery 82d84b2`
* Same technique can be used to recover a branch that was deleted

---

## @csojinb


![](images/young-clara-sunglasses.jpg)
