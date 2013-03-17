---
layout: post
title: My GitHub workflow
---

There is plenty of git and github workflows, models and tips all over
the web[^1]. I have evaluated a bunch of them, and came up with this
guidelines.

I am following it on a daily basis and it is working great so far, thus
I think it is worth sharing.

#### 1. Update your local master
~~~
$ git checkout master
[master]$ git pull
~~~
{: .bash}

I keep the `master` a production-ready branch. It is always green on my
CI and it is the starting point for all feature branches.

I never (ok, _rarely_) commit changes directly into it. Only merge from
other branches. This helps keeping its history clean and eases reverting
or rolling back features.

#### 2. Create a feature branch
~~~
$ git checkout -b brach-name
~~~
{: .bash}

Define a convetion to name your branches. Having name initials and
feature name helps to know what the branch is about and who is
in charge of it.

Sum up the feature name to a single verb and noun sentence. This is
enough to pass the idea of what is being done without cluttering.

It is also good to add ticket/story id. This lets you go straight to
the related issue in your management tool.

You end up with a branch name like: `vh-1337-add-profile-phone`

#### 3. Create short commits, early and often
Now you can craft your spell in safety without disturbing anyone.

Commits should have as little files as possible. Group them logically.
Add the minimum files needed to get the job done. Even if it is a
one-file commit. You will thank yourself when you need to cherry pick in
the future.

Commit early and often. As soon as you commit your changes, easier it
will be to create short commits.

Elaborate well formed commit messages[^2]. It helps convey your
work. If you are facing difficulty to explain your changes, you are
likely trying to commit too much. Split it into two or more commits.

Sometimes we commits changes that do not create real value, e.g.
remove white spaces, fix line endings. Sometimes we write crappy
commit messages. There is no problem on doing it, as long as you fix it
later.

Learn to use `git add --patch` option.

#### 4. Keep your branch updated
It is a good idea to get changes from `master` sometimes to check that
your actual work is still compatible and to get the latest updates.

Prefer merge over rebase[^3]. It avoids history rewriting and it is
easier to fix conflicts.

#### 5. When done, publish it to staging/QA
Set a `staging` branch to merge your feature branches in. This allows
you to have multiple features branches tested at the same time at the
same staging server.

I would not mind messing with the history in the `staging` branch, thus
merging in the feature branch---with its non-useful or crappy
commits---will do.

Just merge with non-fast forward to keep commits grouped together.

~~~
[branch-name]$ git checkout staging
[staging]$ git pull
[staging]$ git merge --no-ff branch-name
~~~
{: .bash}

Fix conflicts and any failing tests and you are ready to go.

#### 6. Squash commits, open pull request and ask for code review
Now that your feature passed the QA tests you can move on.

Remember the crappy commit messages? Now it is time to fix that.
Interactive rebase your feature branch with `master`, squash and rewrite
where you see fit.

Try to limit it to only one commit. It will be easier if you need to
rollback the feature in the future and keeps your `master` history
clean.

~~~
[branch-name]$ git fetch
[branch-name]$ git rebase -i origin/master
~~~
{: .bash}


Open a pull request. I suggest using hub[^4] so you do not
break your workflow leaving the terminal to use the web interface.

~~~
[branch-name]$ hub pull-request
~~~
{: .bash}

Add to the pull request smart commit messages[^5] and links to your
story in the management tool. For example:

> Add field telephone to user profile fixes #1337
>
> Telephone is a required field. Validation is done in the model,
> form text field masked via javascript as (00) 0000-0000.
>
> ref.: https://www.pivotaltracker.com/story/show/1337

Ask your team to review your code and share their
impressions commenting right on it[^6]. After refactorings and
corrections you can merge the pull request.

#### 7. Cleaning up
With your pull request merged, you can safely delete your feature
branch. GitHub has made it very straightforward[^7]. Although, it is
important to know how to do it on your own:

~~~
[master]$ git push origin :branch-name
~~~
{: .bash}

Delete it locally as well.

~~~
[master]$ git branch -d branch-name
~~~
{: .bash}

Clear stale remote-tracking branches removed by someone else (or via GitHub):

~~~
[master]$ git remote prune origin
~~~
{: .bash}

That's it! The post ended bigger than expected, and I apologize for
that. I hope you enjoy and I am looking forward to your opinions in
the comments below.

[^1]:
    * [Git Landscaping](http://robots.thoughtbot.com/post/29355216290/git-landscaping)
    * [Remote Branch](http://robots.thoughtbot.com/post/21306813001/remote-branch)
    * [Feature branch code reviews](http://robots.thoughtbot.com/post/2831837714/feature-branch-code-reviews)
    * [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model)
    * [A Git Workflow for Agile Teams](http://reinh.com/blog/2009/03/02/a-git-workflow-for-agile-teams.html)
    * [Git Branching - Branching Workflows](http://git-scm.com/book/ch3-4.html)
    * [Agile git and the story branch pattern](http://blog.hasmanythrough.com/2008/12/18/agile-git-and-the-story-branch-pattern)

[^2]: [A Note About Git Commit Messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)

[^3]: [Git merge vs. rebase](http://mislav.uniqpath.com/2013/02/merge-vs-rebase)

[^4]: [hub: the command-line wrapper for git](http://defunkt.io/hub/)

[^5]:
    * [Issues 2.0: The Next Generation](https://github.com/blog/831-issues-2-0-the-next-generation)
    * [Closing Issues via Commit
      Messages](https://github.com/blog/1386-closing-issues-via-commit-messages)

[^6]: [Inline commit notes](https://github.com/blog/622-inline-commit-notes)

[^7]: [Tidying up after Pull Requests](https://github.com/blog/1335-tidying-up-after-pull-requests)

