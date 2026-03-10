---
title: "Change author information for all Git commits"
date: 2026-03-10
draft: false
tags: ["git", "github"]
---

Sometimes we want to update _all_ of our previous Git commits so that they use a specific author name and/or e-mail address. One use case is when previous commits include a mix of e-mail addresses, and one or more of them is _private_.

We can see our previous commits and commit information using `git log`.

To update the author information of all of our Git commits, we enter the following command:

```text
git filter-branch -f --env-filter \
"GIT_AUTHOR_NAME='Newname'; GIT_AUTHOR_EMAIL='newemail'; \
GIT_COMMITTER_NAME='committed-name'; GIT_COMMITTER_EMAIL='committed-email';" HEAD
```

{{< alert "comment" >}}
Make sure to update the information for all four fields, not just `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`!
{{< /alert >}}

To verify that the author commit information has indeed been updated, we once again type `git log`.

In future, we can check which e-mail address we are using with this command:

```text
git config --global user.email
```

The `--global` flag indicates the user e-mail address used for _all_ repositories. If we only want to see the e-mail address used for the current repository, then we type `git config user.email`. In both cases, we set our desired e-mail address by typing the command, followed by `"our.new.address@email.com"`.

Now that we’ve updated our author information, we may get an error message when trying to perform a `git pull`. The message will look something like `refusing to merge unrelated histories`. This can be resolved with the following commands:

```text
git pull origin main --allow-unrelated-histories
git merge origin origin/main
```

Sources:

- [Change user name and e-mail of previous commits](https://stackoverflow.com/a/2920001)
- [Resolve pull issues by allowing unrelated histories](https://stackoverflow.com/a/45272685)