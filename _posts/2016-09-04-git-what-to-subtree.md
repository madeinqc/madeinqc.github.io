---
layout: post
title: Git - What is subtree?
comments: true
description: What is git subtree command, what can it do and how to use it. How to manage subfolder on other branches.
image: /public/img/2016/09/04/git-what-is-subtree/subtree.png
---

I had a problem with git recently. I was developing a plugin for the [Atom text editor](https://atom.io/). I am developing in `ClojureScript` ([Check this out if you want a preview of what this is](http://clojurescriptkoans.com/)) that is compiled to `JavaScript`. I then faced the (sad?) reality of Atom releases. The plugin must be at the root of the git repository on the `master` branch.

This is not going to work out for me as my file setup is like this:

```
.                     # This is the root of my git repo
├── LICENSE.md
├── README.md
├── assets
├── plugin            # This is where my plugin is compiled
├── post-release.sh
├── project.clj
├── release.sh
├── src               # This is my ClojureScript source
└── target
```

I was facing a problem: I want the content of my `plugin` folder to be pushed on a `master` branch.

After searching for a bit, I discovered... the git `subtree`!

*OK I know it's not a big surprise, it is in the title, but bear with me.*

Git subtree allow you to take part of your repository (with a prefix) and act on it.

The usage is:

```
usage: git subtree add   --prefix=<prefix> <commit>
   or: git subtree add   --prefix=<prefix> <repository> <ref>
   or: git subtree merge --prefix=<prefix> <commit>
   or: git subtree pull  --prefix=<prefix> <repository> <ref>
   or: git subtree push  --prefix=<prefix> <repository> <ref>
   or: git subtree split --prefix=<prefix> <commit...>

    -h, --help            show the help
    -q                    quiet
    -d                    show debug messages
    -P, --prefix ...      the name of the subdir to split out
    -m, --message ...     use the given message as the commit message for the merge commit

options for 'split'
    --annotate ...        add a prefix to commit message of new commits
    -b, --branch ...      create a new branch from the split subtree
    --ignore-joins        ignore prior --rejoin commits
    --onto ...            try connecting new tree to an existing one
    --rejoin              merge the new branch back into HEAD

options for 'add', 'merge', and 'pull'
    --squash              merge subtree changes as a single commit
```

Let's explain each of the functions with an example.

Create an empty repo (`git init`) and add these empty files:

```
.
├── README.md
└── sub
    └── sub-README.md
```

Commit the files, then add something in each README.md and commit again.

You should have a history (`git log`) that looks like this:

```
commit 5474344ea02e43f9df3c53a3ccf283a5fe1e88ee
Author: Marc-Antoine Sauve <madeinqchw@gmail.com>
Date:   Sun Sep 4 13:51:35 2016 -0400

    Add titles in README files

commit cfcc9844265afdda99b92bb2cfd97fc5b229684b
Author: Marc-Antoine Sauve <madeinqchw@gmail.com>
Date:   Sun Sep 4 13:50:56 2016 -0400

    Add the README files
```

## The **add** command

```
git subtree add   --prefix=<prefix> <commit>
git subtree add   --prefix=<prefix> <repository> <ref>
```

`git subtree add` will add a subtree to your *current repository* and add it as a subfolder. The source can be a `branch`, a `remote branch` or a `commit`.

Let's continue with our example, we will add our first commit (the empty files) to our current repository under the folder `empty-files`.

`git subtree add --prefix=empty-files cfcc9844265afdda99b92bb2cfd97fc5b229684b`

The result is now

```
.
├── README.md
├── empty-files
│   ├── README.md
│   └── sub
│       └── sub-README.md
└── sub
    └── sub-README.md
```
And a new commit appear in our logs
```
commit 6214367b9abc510e20798ffab15ab57c1825bfd2
Merge: 5474344 cfcc984
Author: Marc-Antoine Sauve <madeinqchw@gmail.com>
Date:   Sun Sep 4 13:52:26 2016 -0400

    Add 'empty-files/' from commit 'cfcc9844265afdda99b92bb2cfd97fc5b229684b'

    git-subtree-dir: empty-files
    git-subtree-mainline: 5474344ea02e43f9df3c53a3ccf283a5fe1e88ee
    git-subtree-split: cfcc9844265afdda99b92bb2cfd97fc5b229684b
```

## The **split** command

```
git subtree split --prefix=<prefix> <commit...>

options for 'split'
    --annotate ...        add a prefix to commit message of new commits
    -b, --branch ...      create a new branch from the split subtree
    --ignore-joins        ignore prior --rejoin commits
    --onto ...            try connecting new tree to an existing one
    --rejoin              merge the new branch back into HEAD
```

This command allows you to create a new tree for this particular folder. This will allow you to make a new branch out of it. I will not go in depth with the options as this is outside the scope of this post.

Let's split our `sub` folder to a new branch called `separated`.

`git subtree split --prefix=sub -b separated`

Checkout the branch to find this only file
```
.
└── sub-README.md
```

## The **merge** ~~command~~ trap

Let's assume that our `sub-README.md` file changed on the `separated` branch according to this commit

```
commit a0c39f1ffb74a271b1ba897747e90f36d9f0a7f9
Author: Marc-Antoine Sauve <madeinqchw@gmail.com>
Date:   Sun Sep 4 13:55:31 2016 -0400

    Add detail to sub README
```

We now want to bring these modifications to the master branch

If you want to merge you could try the `git subtree merge` command.

`git subtree merge --prefix=sub a0c39f1`

But the command is not able to merge and we end up with a conflict.

```
<<<<<<< HEAD
# Sub README
||||||| merged common ancestors
=======
# Sub README
I am a README
>>>>>>> a0c39f1ffb74a271b1ba897747e90f36d9f0a7f9
```

We can see that the merge HEAD is the initial value, the merged common ancestor is empty (why?) and the new commit is on the bottom.
A normal merge is expected to resolve automatically since only one side changed. A real merge would look like this:

```
<<<<<<< HEAD
# Sub README
||||||| merged common ancestors
# Sub README
=======
# Sub README
I am a README
>>>>>>> a0c39f1ffb74a271b1ba897747e90f36d9f0a7f9
```

For the second scenario to happen, they would need to share the same ancestor and the merge would resolve by itself.

### How to solve the ancestor problem

A simple way to make git aware that they share the common ancestor is to use the `merge` command with a `subtree` strategy for merging instead of the `subtree merge` command. That way the merge will resolve properly.

We can merge `separated` into the current branch with the `subtree` strategy with this command.

`git merge -s subtree separated`

Which merge without conflict. The following commit is created.

```
commit a5bd4f7c19904adca556335c0ca023d4f87df591
Merge: 6214367 a0c39f1
Author: Marc-Antoine Sauve <madeinqchw@gmail.com>
Date:   Sun Sep 4 15:53:35 2016 -0400

    Merge branch 'separated'
```

## Pull and push

The `subtree pull` and `subtree push` work as the normal pull and push, except that they work on the specified subdirectory only. For example, you can push a subdirectory directly on a new branch.

## Conclusion

This is all the time I have for this post, so enjoy these useful links to learn more about the `subtree` and `submodule` in git!

## Useful links

[6.7 Git Tools - Subtree Merging](https://git-scm.com/book/en/v1/Git-Tools-Subtree-Merging)
[About Git subtree merges - Github](https://help.github.com/articles/about-git-subtree-merges/)
[Mastering Git subtrees](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec#.dywbc81ip)
[Alternative - Git subrepo](https://github.com/ingydotnet/git-subrepo#readme)
