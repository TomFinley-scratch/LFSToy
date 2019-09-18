# What is this?

This is meant to serve as a very light illustration of what LFS is. Once
you've installed LFS, 

## Setup of LFS

I created this on Github as a barebones repo.

https://github.com/TomFinley/LFSToy

Then I created two files with identical content `theme.txt` and and
`theme-lfs.txt` (in this repo).

    git lfs install
    git lfs track "*-lfs.txt"

The latter command added a `.gitattributes` file. Next I `add`, `commit`, and
`push` these off to origin, more or less as usual.

    git add .gitattributes theme*.txt
    git commit -m "Quality family entertainment."
    git push

The last command's output may be worth investigating:

    Uploading LFS objects: 100% (1/1), 125 B | 0 B/s, done
    Enumerating objects: 6, done.
    Counting objects: 100% (6/6), done.
    Delta compression using up to 12 threads
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (5/5), 596 bytes | 596.00 KiB/s, done.
    Total 5 (delta 0), reused 0 (delta 0)
    To https://github.com/TomFinley/LFSToy
    c40e8d5..63e437a  master -> master

## Changes

Let's imagine I change the file. I edited both `theme.txt` and `theme-lfs.txt`
(what those changes are, we shall see below, or you can view the commit
history).

See the output of `git diff theme.txt`:

```diff
diff --git a/theme.txt b/theme.txt
index 3d3ff01..b58ea6a 100644
--- a/theme.txt
+++ b/theme.txt
@@ -1,10 +1,13 @@
 They bite,
 and fight!
+And bark!
 They fight and bite and fight!
+And bark!

 Bite bite bite,
-fight fight fight!
+woof woof woof.

 The
 Itchy & Scratchy
+& Poochie
 Show!
\ No newline at end of file
```

Now see the output of `git diff theme-lfs.txt`:

```diff
diff --git a/theme-lfs.txt b/theme-lfs.txt
index a163335..314e095 100644
--- a/theme-lfs.txt
+++ b/theme-lfs.txt
@@ -1,3 +1,3 @@
 version https://git-lfs.github.com/spec/v1
-oid sha256:685d65ee9f9e81297aa84961dc1019818248afe2347ba2152bd2bc3c4590ce0f
-size 125
+oid sha256:c95c23d5b24fd3cfa6bea286455e260d2c5e0af2312ea2505f0628d03c63f0e7
+size 155
```

Even though the *content* of the files is the same from a user's perspective,
`git`'s tracking of these files differs enormously.

    git add *.txt
    git commit -m "Poochie is one outrageous dude. He's totally in my face!"
    git push

Note that the `git push` output will have more information about uploading
this new file.

# Recloning

Now, I have a complete history in my own machine, but let's imagine I do a fresh repo. Going back to my directory where I originally cloned, let me clone this repo.

    git clone https://github.com/TomFinley/LFSToy LFSToy2
    cd LFSToy2

We'll note that the files are there just as if we had just checked them out. However, let me imagine I check out my first "real" commit. (I just provide the commit hash for this.)

    git checkout 84b1d37

The files are again just there. So the LFS support is working.

Now let's do something a bit trickier. Let's go again to the clone directory, and clone yet again.

    git clone https://github.com/TomFinley/LFSToy LFSToy3
    cd LFSToy3

The files are right there. But now let's be extra tricky. I am going to *disable* my Internet connection (in my case by just )

    git checkout 84b1d37

Now instead of just working, something more interesting happens.

    Downloading theme-lfs.txt (125 B)
    Error downloading object: theme-lfs.txt (685d65e): Smudge error: Error downloading theme-lfs.txt (685d65ee9f9e81297aa84961dc1019818248afe2347ba2152bd2bc3c4590ce0f): batch response: Post https://github.com/TomFinley/LFSToy.git/info/lfs/objects/batch: dial tcp: lookup github.com: no such host

    Errors logged to E:\src\LFSToy3\.git\lfs\logs\20190918T101251.1345066.log
    Use `git lfs logs last` to view the log.
    error: external filter 'git-lfs filter-process' failed
    fatal: theme-lfs.txt: smudge filter lfs failed

Ha! However note that `theme.txt` is at that version, because by default `git
clone` fetches the complete history of the repo. (LFS files are an exception
to that!) But because there was an initial. I can do a `git reset --hard HEAD`
to get things back into their proper state, reconnect my Ethernet cable, and
checkout again, and then it will work.

## How To Not Download

Note that all these downloads of the LFS files happened automatically upon
checkout under these default assumptions. There's an interesting configuration
option, [explained here](https://github.com/git-lfs/git-lfs/issues/2717), that
may be worth exploring if you have a *super* large repo. With such a
configuration, a checkout or initial clone would not download the file by
default, but would instead defer that until someone takes an affirmative step
to download file(s).