A Guide To Undoing Mistakes With Git
====================================

Working with code is a risky endeavour: There are countless ways to shoot yourself in the foot! But if you use Git as your version control system, then you have an excellent safety net. A lot of “undo” tools will help you recover from almost any type of disaster.

In this first article of our two-part series, we will look at various mistakes — and how to safely undo them with Git!

### Discard Uncommitted Changes in a File

Suppose you’ve made some changes to a file, and after some time you notice that your efforts aren’t leading anywhere. It would be best to start over and undo your changes to this file.

The good news is that if you haven’t committed the modifications, undoing them is pretty easy. But there’s also a bit of bad news: You **cannot bring back the modifications** once you’ve undone them! Because they haven’t been saved to Git’s “database”, there’s no way to restore them!

With this little warning out of the way, let’s undo our changes in `index.html`:

```
    $ git restore index.html
```

This command will restore our file to its last committed state, wiping it clean of any local changes.

### Restore a Deleted File

Let’s take the previous example one step further. Let’s say that, rather than modifying `index.html`, you’ve **deleted it entirely**. Again, let’s suppose you haven’t committed this to the repository yet.

You’ll be pleased to hear that `git restore` is equipped to handle this situation just as easily:

```
    $ git restore index.html
```


The `restore` command doesn’t really care _what_ exactly you did to that poor file. It simply recreates its last committed state!

### Discard Some of Your Changes

Most days are a mixture of good and bad work. And sometimes we have both in a single file: Some of your modifications will be great (let’s be generous and call them genius), while others are fit for the garbage bin.

Git allows you to work with changes in a very granular way. Using `git restore` with the `-p` flag makes this whole undoing business much more nuanced:

```
    $ git restore -p index.html
```

Git takes us by the hand and walks us through every **chunk of changes** in the file, asking whether we want to throw it away (in which case, we would type `y`) or keep it (typing `n`):

Using “git restore” with the “-p” option to discard parts of a file’s changes.

If you’re using a Git desktop user interface, you can go even deeper. Apps like these allow you to select which code to keep, discard, and stage not only at the level of chunks, but even for **individual lines of code**. One of such tools is [Tower](https://www.git-tower.com/?utm_source=smashingmagazine&utm_medium=guestpost&utm_campaign=undoing-mistakes-01), the one that yours truly is working on.

Desktop GUIs like [Tower](https://www.git-tower.com/?utm_source=smashingmagazine&utm_medium=guestpost&utm_campaign=undoing-mistakes-01) allow you to discard (and stage) changes at the level of individual lines.

### Fix the Very Last Commit

Raise your hand if you’ve never made a typo in a commit message or never forgotten to add one last change. No hands? That’s what I thought. Because messing up a commit is so terribly common, Git makes it very easy to fix such mistakes.

Let’s look at a prime example of a bad commit message:

[![](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_400/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/f0975d23-2d01-4137-b905-b5f12b7f67f5/commit-with-typo-2x.png)](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/f0975d23-2d01-4137-b905-b5f12b7f67f5/commit-with-typo-2x.png)

A tragic case of a badly mistyped commit message! ([Large preview](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/f0975d23-2d01-4137-b905-b5f12b7f67f5/commit-with-typo-2x.png))

Using the `--amend` option allows you to change this very last commit (and _only_ this one):

```
    $ git commit --amend -m "A message without typos"
```    


In case you’ve also **forgotten to add a certain change**, you can easily do so. Simply stage it like any other change with the `git add` command, and then run `git commit --amend` again:
```
    $ git add forgotten-change.txt
    
    $ git commit --amend --no-edit
```    

The `--no-edit` option tells Git that we don’t want to change the commit’s message this time.

### Revert the Effects of a Bad Commit

In all of the above cases, we were pretty quick to recognize our mistakes. But often, we only learn of a mistake long after we’ve made it. The bad commit sits in our revision history, peering snarkily at us.

[![](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_400/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/393dea41-d7d0-431f-9ecb-28c698c1a041/revert-commit-01-2x.png)](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/393dea41-d7d0-431f-9ecb-28c698c1a041/revert-commit-01-2x.png)

A bad commit, somewhere in the middle of our commit history. ([Large preview](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/393dea41-d7d0-431f-9ecb-28c698c1a041/revert-commit-01-2x.png))

Of course, there’s a solution to this problem, too: the `git revert` command! And it solves our issue in a very non-destructive way. Instead of ripping our bad commit out of the history, it creates a **new commit** that contains the opposite changes.

[![](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_400/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/4a7f6c5c-827f-433a-b0f6-6c52e5252e43/revert-commit-02-2x.png)](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/4a7f6c5c-827f-433a-b0f6-6c52e5252e43/revert-commit-02-2x.png)

Using “git revert” allows us to undo the effects of a bad commit by automatically creating a new one. ([Large preview](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/4a7f6c5c-827f-433a-b0f6-6c52e5252e43/revert-commit-02-2x.png))

Performing that on the command line is as simple as providing the revision hash of that bad commit to the `git revert` command:

```
    $ git revert 2b504bee
```

As mentioned, this will _not_ delete our bad commit (which could be problematic if we have already shared it with colleagues in a remote repository). Instead, a **new commit** containing the reverted changes will be automatically created.

### Restore a Previous State of the Project

Sometimes, we have to admit that we’ve coded ourselves into a dead end. Perhaps our last couple of commits have yielded no fruit and are better off undone.

[![](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_400/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/b2bd12f2-9574-434d-bc1e-04801ec27451/reset-before-and-after-2x.png)](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/b2bd12f2-9574-434d-bc1e-04801ec27451/reset-before-and-after-2x.png)

The “git reset” command allows us to return to a previous state. ([Large preview](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/b2bd12f2-9574-434d-bc1e-04801ec27451/reset-before-and-after-2x.png))

Luckily, this problem is pretty easy to solve. We simply need to provide the SHA-1 hash of the revision that we want to return to when we use the `git reset` command. Any commits that come after this revision will then disappear:

```
    $ git reset --hard 2b504bee
```    

The `--hard` option makes sure that we are left with a **clean** working copy. Alternatively, we can use the `--mixed` option for a bit more flexibility (and safety): `--mixed` will preserve the changes that were contained in the deleted commits as local changes in our working copy.

The “--mixed” option preserves the contained changes.

### Recover Lost Commits Using the Reflog

By now, you’ve probably noticed that, when it comes to undoing mistakes, almost anything is possible with Git! This includes **undoing an undo**. Let’s say we’ve realized that the `git reset` that we just performed above was not our brightest idea. We’re afraid that we’ve lost valuable commits, sending us into panic mode.

As you can guess now, we can fix this problem, too — with the help of a particular tool. `reflog` is a kind of journal in which Git protocols all movements of the `HEAD` pointer. In other words, any time we commit, checkout, merge, rebase, cherry-pick, etc., a new entry will be created in this journal. Luckily, this also happens when we use `git reset`!

Let’s open reflog with a simple command of `git reflog`. Take a look at what we have:

[![](https://res.cloudinary.com/indysigner/image/fetch/f_auto,q_auto/w_400/https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/0819576a-1ffe-4d18-8cb0-3c0479463780/git-reflog-2x.png)](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/0819576a-1ffe-4d18-8cb0-3c0479463780/git-reflog-2x.png)

Reflog is a protocol of every HEAD pointer movement in our local repository. ([Large preview](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/0819576a-1ffe-4d18-8cb0-3c0479463780/git-reflog-2x.png))

The first thing to know about reflog is that it’s **ordered chronologically**. Therefore, it should come as no surprise to see our recent `git reset` mistake at the very top. If we now want to undo this, we can simply return to the state before, which is also protocoled here, right below!

We can now copy the commit hash of this safe state and create a new branch based on it:

```
    $ git branch happy-ending e5b19e4
```

Of course, we could have also used `git reset e5b19e4` to return to this state. Personally, however, I prefer to create a new branch: It comes with no downsides and allows me to inspect whether this state is really what I want.

### Restore a Single File From a Previous State

Until now, when we’ve worked with committed states, we’ve always worked with the complete project. But what if we want to **restore a single file**, not the whole project? For example, let’s say we’ve deleted a file, only to find out much later that we shouldn’t have. To get us out of this misery, we’ll have to solve two problems:

1.  find the commit where we deleted the file,
2.  then (and only then) restore it.

Let’s go search the commit history for our poor lost file:

```
    $ git log -- <filename>
```

The output of this lists all commits where this file has been modified. And because `log` output is sorted chronologically, we shouldn’t have to search for long — the commit in which we deleted the file will likely be topmost (because after deleting it, the file probably wouldn’t show up in newer commits anymore).

With that commit’s hash and the name of our file, we have everything we need to bring it back from the dead:

```
    $ git checkout <deletion commit hash>~1 -- <filename>
```

Note that we’re using `~1` to address the commit _before_ the one where we made the deletion. This is necessary because the commit where the deletion happened doesn’t contain the file anymore, so we can’t use it to restore the file.

### You Are Now (Almost) Invincible

During the course of this article, we’ve witnessed many disasters — but we’ve seen that virtually nothing is beyond repair in Git! Once you know the right commands, you can always find a way to save your neck.

But to really become invincible (in Git, that is), you’ll have to wait for the **second part of this series**. We will look at some more hairy problems, such as how to recover deleted branches, how to move commits between branches, and how to combine multiple commits into one!

In the meantime, if you want to learn more about undoing mistakes with Git, I recommend the free “[First Aid Kit for Git](https://www.git-tower.com/learn/git/first-aid-kit?utm_source=smashingmagazine&utm_medium=guestpost&utm_campaign=undoing-mistakes-01)”, a series of short videos about this very topic.

See you soon in part two of this series! [Subscribe to the Smashing Newsletter](https://www.smashingmagazine.com/the-smashing-newsletter/) to not miss that one. ;-)
