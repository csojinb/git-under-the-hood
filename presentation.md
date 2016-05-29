slidenumbers: true

# git
## a peek under the hood

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

# What does git store when you commit?

![](images/dunes.jpg)

---
## Core Concept: History as Snapshots

* To understand how git stores your commits, it's useful to understand the central "philosophy"
* Git "thinks" about version history as a series of **snapshots**, rather than a series of deltas
* A **snapshot** is a complete copy[^1] of the project at a particular point in history

[^1]: Unchanged files are not stored multiple times. And, eventually, git will compress versions of the same file together to save space when necessary, e.g. if you want to push to a remote. But the snapshot still decompresses to a complete project copy.

---
![right fit](images/mv-example.png)

# Representing Changes

* Git does not directly save any **actions** that you took, only the **state**
* Differences are **derived** by comparing snapshots
* Actions are **inferred**
* Example (right): git recognizes the rename because the **file contents** is the same
    * Equivalent to `git mv`

---
## Important implication!

* Git's ability to track a file's history[^2] depends on the file being recognizably the same file between commit snapshots
* i.e. the following may break the file history:

    `$ git mv file.py other.py`

    _<make lots of changes to `other.py`>_

    `$ git add --all; git commit`

[^2]: You may be taking advantage of this feature in, say, GitHub, even if you aren't using it locally.

---
# Snapshot Storage

* A file snapshot is stored as a text blob, and a directory snapshot is represented as a "tree" object
* Each snapshot is check-summed and stored by SHA-1 value
* Directory trees point to the SHAs of files and directories they contain
* The project snapshot is just the "tree" for the project root directory

![right fit](images/snapshot.png)

---
# Building A Commit

* To make a commit, first you need to stage some changes
* The staging area[^3] is just another project snapshot tree
* As changes are staged, new snapshots are created of the affected files/directories, and the staging area is updated
* On commit, the staging area becomes the commit snapshot

[^3]: Sometimes referred to as the "index".

---
# The Meta-Data

* The final commit object contains a pointer[^4] to the **project snapshot** (the content) and some **meta-data**
* The meta-data includes the author, the commit message, and pointer(s) to the **parent commit(s)**[^5]
* Note that if either the content or the meta-data is amended, the new commit will have a different SHA checksum value

[^4]: The "pointer" is SHA of the project snapshot

[^5]: The initial commit has no parents, and merge commits have two or more.

---
## Visualizing commit storage

![inline fit](images/two-commits.png)

---
## Example: Understanding `git reset` options
## Slated for deletion, or at least cleanup

`git reset [<mode>] HEAD~1`

* **mode = soft:** moves branch back to first parent; index and working directory unchanged _(changes remain "staged")_
* **default (mode = mixed):** moves branch back to parent; staging index reset to parent _(changes "unstaged")_
* **hard:** moves branch back to parent; index and working directory reset _(commit and changes erased entirely)_

---
# Why are branches "cheap"?

![](images/trees.jpg)

---
## Branching (structure) comes for free

![right fit](images/history.png)

* Together, commits and parent relations form the git history DAG[^6]
* Multiple commits can share a parent => natural "branching" structure
* Could theoretically manage divergent version paths without an explicit "branch" concept[^7]

[^6]: It can be further specified as a rooted connected directed acyclic graph. :open_mouth: Note that the history is _not_ a tree because commits can have multiple parents, but it is tree-like in other respects.

[^7]: It would involve manually tracking commit SHAs, though. :scream:

---
## A git branch (object) is just a pointer

![right fit](images/history-branches.png)

* Git's "branch" object (stored as reference to a commit SHA) affords two major conveniences:
    * Nice name for checkouts, etc
    * The checked-out branch moves forward with each new commit
* Note: the **master** branch is (technically) the same as any other[^8]

[^8]: Everyone has one because the branch created by `git init` is called "master" by default.

---
## Ergo, branches are cheap

* Creating a branch == creating a SHA reference: cheap!
* Because git only creates new file snapshots for modified files, they are also cheap to maintain[^9]
* Deleting a branch deletes the ref only: also cheap!
    - Bonus: the commits still exist and [can be recovered](#recover-branch)

[^9]: Relative to other VCSs that maintain an entirely seperate project copy per branch.

---
## Merges are (fairly) easy

* To merge, git compares branches to their best **merge base**
* The merge base (most recent common ancestor) is easily determined from the commit graph
* Unlike a simple 3-point merge, git preserves granular history info by replaying commits from one branch onto the other
* This allows git to correctly handle many tricky merge situations, e.g. file renames

---
## Example merge scenario

![right fit](images/merge-base.png)

To merge `bar` into `foo`:
    ```
    $ git checkout foo
    $ git merge bar
    ```

* Compute diffs `(C - B)` and `(D - C)`
* Apply diffs in order onto `E`
* Turn the result into a merge commit

---
# Homeless information!

* The checked-out branch moves forward when a new commit is made[^10]


[^10]: This is in contrast to tags (similarly just pointers to commits), which stay put unless explicitly moved.

---
# How do checkouts work?

---
# Where am I?

* The `HEAD` reference determines what is "checked out"
* If a branch is checked out, `HEAD` points to the branch ref
* The "unattached HEAD" state occurs when `HEAD` points directly to a commit
* Either way, the associated snapshot is identical[^11] to the state of the working directory

[^11]: Assuming that the working directory is clean, that is.

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
## Recover a deleted branch
# NEEDS FIXING

<a name="recover-branch"/>

![right fit](images/amend-example.png)

* Recall that commit "modification" actually creates a _new_ commit
* Previous "version" still exists
* Can create new branch to point to old commit:
`$ git branch recovery 82d84b2`
* Same technique can be used to recover a branch that was deleted

---

## @csojinb


![](images/young-clara-sunglasses.jpg)
