---
title: Version control for one
categories:
- tools
feature_image: "https://picsum.photos/2560/600?image=872"
image: "https://jmtayloruk.github.io/assets/searching-documents-test-image.jpg"
---

This post makes the case for using version control when you are just writing your own code and maybe don't expect to work with anyone else on it, or to share it with anybody else.
There are plenty of articles out there about [how to use version control tools like git](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004668),
and [why they are useful for teams working on the same code](https://www.atlassian.com/git/tutorials/why-git#git-for-developers).
I don't know of any articles explaining why version control is valuable even when it's just you. This post aims to do that.

## What is version control?

I won't duplicate all the information already out there, but in one sentence:
version control is a [properly structured](http://phdcomics.com/comics/archive_print.php?comicid=1531) way of recording and tracking the changes that you make to your code. 
If you are writing anything more complicated than a few lines of code that you never plan to modify again, you would benefit from using version control.

Using a tool like `git`, it just takes a few clicks to review and *commit* (i.e. archive) a version of your work, 
along with information about what/why you made the changes/additions that you made.
By doing this you build up a "tree" of commits that retains the full modification history of the files in your "repository" (a project you are working on).
This probably works a bit differently from your OS's built-in backup system.
WIth version control, a version is only committed when you decide to make that happen - it doesn't happen automatically at regular intervals like a backup.
But *every* version you commit will always be there for you to look back at - there is no culling of old backups.  
Version control and backups therefore complement each other.

In this post I'll describe various scenarios and workflows where version control is useful.
I'll refer to the `git` version control system, which is the *de facto* standard these days, but the principles apply to any version control software. 
This post isn't intended as a tutorial for installing or using `git`, but I'll mention "what you install" in the next section. 
Throughout the rest of the post I'll mention commands for use with `git`, but I'm assuming you will look up the documentation or follow a "how-to" tutorial if you are unfamiliar with them.

## What is git?

`git` is a [program installed on your computer](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
Once you have installed `git`, you have the choice to interact with it, and manage your projects, [via a GUI](https://git-scm.com/downloads/guis) (i.e. you click within that GUI program to perform actions like committing your changes),
or on the command line on [Linux/Mac OS](https://git-scm.com/book/en/v2/Getting-Started-The-Command-Line) or [Windows](https://www.atlassian.com/git/tutorials/git-bash)
(i.e. you type text commands to perform actions like committing your changes).
So, you write and save your code in whatever editor you would normally use for that, but when you are ready to *commit* a version of your work,
you use either your `git` GUI program or the command line to perform that task.

At the end of this post I say a bit more about options for installing `git` on your computer.

A couple of bits of basic terminology to help orient yourself with this and other articles about version control:

- **repository**: a directory containing the code (and other files) for a self-contained project you are working on. `git` will keep track of the changes that take place within your repository.
You can have more than one repository; `git` will treat them as completely independent projects.
- **commit** (noun): a checkpoint for your code, where you have made a conscious decision to save a version your code in its current state, after you have made some changes. Each commit is retained forever as part of the **history** of your repository. 
- **commit** (verb): the act of saving a new version of your code as a commit to the repository.
- **remote repository**: a site "in the cloud" (e.g. [github.com](https://github.com)) that stores a mirror copy of your repository.  

## Typical workflow

When you start a new project (or make the decision to bring an existing project under version control) you gather your files together in a directory and designate it as a *git repository*.
These files could include source code, configuration files, documentation, or example data.
Typically you would not include very large (hundreds of MB) data files as part of the repository, and would store those elsewhere.

As you work, you make regular *commits* of your work to the repository, committing the latest version of the files you have changed.
The typical scale of a commit you make could be "add the effects of gravity", "fix bug in outlier detection code", etc.
Since it makes sense to use separate commits for separate changes with distinct purposes, you might sometimes make multiple commits in an hour,
but other times you might work for days and then make one commit of a major change.
Either way, you will save your work in your usual file editor a lot more often than you will commit your changes in `git`.
The act of committing your changes is a natural time to use git's tools to help you review your work and spot any mistakes you have made in your code (see next section). 

I am mostly talking about using version control for computer code, but in principle it can be used for tracking changes to anything.
One of the biggest benefits of version control workflows is the easy and natural ability to review your changes,
so it makes most sense to use it for projects that mostly involve text-based files where you (and your tools) can easily compare and display changes between versions.
So, if you write papers using LaTeX, that process also lends itself naturally to version control - especially if collaborating between multiple authors (though this post is about version control as an individual, rather than as a team).
If you write in Word documents (or images, or binary data files), you can still put them under version control if you wish, but there are fewer benefits because it won't be easy for git's tools to show you what has changed between versions.
You may still find it helpful, however, especially if it is important for you to be able to maintain and refer back to a history of changes to your datasets.

In the following sections I will elaborate on some of the specific scenarios where you will benefit from using version control.

## Natural checkpoints

Committing a set of changes represents a natural checkpoint for you to review the changes you have made to your code.
You can easily review what you've modified since the last time you committed your changes (`git difftool *`).
This is a good time to reflect on what you've changed and whether you are happy with what you have written.
But this is also a lifesaver moment. Have you ever made a "temporary change" to your code ("I'll just disable gravity for a moment, to see what happens...")
and then realised two weeks later that all your calculations have been meaningless because you forgot to undo that change?
When you review your changes before a commit, `git` will flag up that line for you as one that you have changed - and that's your prompt to change it back.

Incidentally, the first thing to do after you install `git` on a computer should be to set up a proper GUI for browsing changes to your files.
It's essential that you can clearly and conveniently see differences between versions of your files.
Otherwise you won't get into the habit of following good workflows with `git`, and won't get the full benefits out of it.
On a Mac, installing [Xcode](https://developer.apple.com/xcode) gives you access to the FileMerge tool, which is a very nice interface for looking at differences between files.
On Linux or Windows, I recommend [diffmerge](http://www.sourcegear.com/diffmerge), which is structured very similarly.

## True backup history

Because `git` keeps a record of every commit you make, you can browse back through - or revert to - any previous version of your files if you wish.
But you can do more than that. You can [jump back and forth through history](https://git-scm.com/docs/git-checkout#_examples) ([`git checkout <hash>`](https://git-scm.com/docs/git-checkout)).
Making changes to a paper now you've got the reviewers' comments back? 
Maybe you've been improving your code in the meantime, but now you can't replot the graph from the paper because you've changed the code?
You can `git checkout` the version of the code you committed at the time, or even [go back to a specific date](https://stackoverflow.com/questions/6990484/how-to-checkout-in-git-by-date).

What if you realise your code is not working for scenario X? You know it used to work, you haven't looked at scenario X for six months, and you've inadvertantly broken *something* in the interim. But what was it!?
In the worst case, you can go back and forth through your commit history and figure out which commit broke it.
You can do that using a manual [binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm), or [let git help you](https://git-scm.com/docs/git-bisect) using [`git bisect`](https://git-scm.com/docs/git-bisect).
The binary search has saved my bacon a few times.

Want to figure out when and why you wrote a line of code? [`git blame`](https://git-scm.com/docs/git-blame) is traditionally used to find out "whose fault" a particular line of code is, but it's useful even if you're the only author.
If you're staring at a line of code and can't work out why it's there, `git blame` will tell you when you wrote it, and which commit it was written as part of.
Especially if you've written informative comments alongside your commits, that may help you figure out the purpose of that particular piece of code!

Finally, the `git` history also serves as an electronic lab book - a "when/who/why" record of changes made to your project files. This could be private, shared with colleagues, or fully public.
Like an electronic lab book, it provides a fairly good historical record that could be useful evidence of priority, who has contributed what and when, etc.
That historical record is near-indelible: it would be technically very difficult (though not completely impossible) to rewrite history.

## Cloud backup

Laptop stolen? Computer gone up in flames? Hopefully you have a physically separated backup of the contents of your hard drive! 
At least your code should be safe either way though: as long as you `git push` a copy of your code to a remote repository like [github.com](https://github.com) there is always a backup of your repository in the cloud.
Even if your computer dies, you always have a remote copy of your code.

Note that you can use github but still set your repository to be "private" so it's only accessible to you, using your user credentials.
Although github is often used for open-source projects that are visible to anybody on the internet, yours doesn't have to be.

Incidentally, if you *do* choose to make your repository publicly visible, this can be useful for making available the code underpinning your scientific publication,
or for showcasing your work as part of future job applications.

## Keeping multiple computers in sync
Even if you're working alone, you might have copies of your code on multiple computers. How do you keep the different copies in sync?
Easy! With a remote repository, `git push` and `git pull` will let you ensure the copies on each computer stay in sync.
By using [stashing](https://git-scm.com/docs/git-stash#_examples) (`git stash; git pull; git stash pop`) you can even keep local changes while picking up the changes you made on another computer.
Git is generally pretty good at *merging* changes, so your local changes to one file can often be folded in seamlessly with "upstream" changes made on another computer.
I can't believe I went through my whole PhD (before I discovered `git`) doing manual merges to keep two computers in sync...

## Project-managing big changes
If you're working on a more ambitious long-term project, sooner or later you'll find yourself thinking about making a dramatic restructure or rethink of your code.
But that takes time, and you probably don't want to completely break your code while you work on that.
You probably still want to be able to use your code for calculations in the meantime.
This is one use-case where [branches](https://git-scm.com/docs/user-manual#what-is-a-branch) come in.
You can `git branch` to work on your dramatic change on a new branch, but you can then set aside your work in progress for later (`git commit` or `git stash`)
and go back to your previously-working code when you need to do calculations with it. You can even continue to make (and commit) more routine little changes to the existing code on your main branch.
When you're ready to switch over to the new dramatically-changed code, `git merge` will take care of bringing it all back together for you.

## A remark about Jupyter notebooks
In the last few years I've been using Jupyter notebooks more and more. Any type of file can be in a git repository, so the notebooks can live in there just like the rest of your code.
But there's one catch to watch out for. Every time you *run* a notebook, the contents of the file change (to store the *output* from your code). 
If you commit your .ipynb file including that output, it can make it very difficult to use `git difftool` to see changes from one commit to the next.
I am not aware of a good universal solution to this problem. My best suggestion is just to "restart and clear output" in your notebook (and save it), before you `git commit` your changes.
If you're in the habit of consulting `git difftool` before you commit - which you should be - then you will immediately spot if you forget to do this.
Some diff-viewing programs (including Visual Studio Code) will let you filter out any changes to the output, to make it easier to identify changes to the *code* in your notebook.

## Installing git

Personally I use `git` from the command line. That's not everyone's preference, but it does give you clear control - and hopefully understanding - of what you are doing. It also means you can start simple, just using a repertoire of a few commands that do the basics you need. This post isn't intended as a tutorial for installing `git`, but [there is guidance here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

Others prefer to use `git` by [clicking buttons in one of many available GUI programs](https://git-scm.com/downloads/guis). In fact, if you are already using [Visual Studio Code](https://code.visualstudio.com) as your development environment then [you have built-in access to version control](https://code.visualstudio.com/docs/editor/versioncontrol). [Matlab can also integrate with version control](https://uk.mathworks.com/help/matlab/source-control.html). However, these environments (and their tutorials) tend to assume you are already very familiar with the terminology and processes of version control, and would probably be rather overwhelming to a first-timer. For that reason I probably wouldn't recommend using these for your first taste of version control.

## Conclusion
Use version control! If you're writing anything more than completely trivial code, I can guarantee you won't regret investing time to learn the basics of `git`.  
