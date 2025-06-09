---
title: "Setting up Hugo and Blowfish"
date: 2025-05-30
draft: false
tags: ["hugo", "blowfish", "github-actions"]
---

{{< lead >}}
Install Hugo and Blowfish, and learn how to deploy the website on GitHub Pages. Also learn some cool features for blog entries.
{{< /lead >}}

This first post covers how to set up this blog. It is built with the Hugo static site generator and uses the [Blowfish theme](https://blowfish.page/). As this website’s source code is stored on GitHub, check out the [Hugo documentation](https://gohugo.io/host-and-deploy/host-on-github-pages/) for details on how to host the site on GitHub Pages. (GitHub’s default static site generator is Jekyll, so extra steps are required.)

## Initial impressions

I found setting up a Hugo static site on GitHub Pages to be a fair bit more involved than doing the same on GitLab Pages. When I set up my first static site on GitLab, the process also took some time, but I don’t remember having to jump through this many hoops. One restriction that GitHub imposes, which GitLab doesn’t, is that to use GitHub Actions one’s repo must be _public_. With GitLab Pages one could set up a static site but keep the repo private.

The trade-off is that one gets 2,000 CI/CD minutes per month on GitHub, as opposed to GitLab’s 500. (GitLab, for its part, offers 10 GB of storage to GitHub’s 500 MB.) This applies to the free tiers of both services, to be clear.

## Hugo and Blowfish theme installation

This assumes that a Git repository for GitHub Pages has been created. Follow the steps [here](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) to get this prerequisite sorted. Clone the repository and navigate to its root level in the terminal.

The next steps are to install the required tools. I use [Homebrew](https://brew.sh/) to install the following:

- [Hugo](https://formulae.brew.sh/formula/hugo) (installation of Go isn’t necessary)
- [Node.js](https://formulae.brew.sh/formula/node) (this isn’t strictly necessary, but having it allows us to use Blowfish’s CLI tool for installation and configuration)

Find the Blowfish documentation for installation [here](https://blowfish.page/docs/installation/). I use the CLI-based `blowfish-tools` to install the theme.

```bash
npx blowfish-tools
```

This shows a splash screen and the following interactive menu:

```bash
Welcome to Blowfish tools.
I can help you setup a new project from scratch or configure an existing one (or both).
Please choose one of the options below, start typing to search.
? What do you need help with? …
Setup a new website with Blowfish
Start from a configured template
Install Blowfish on an existing website
Exit
```

Use the arrow keys to navigate up and down, and press Enter to select the option.

Select `Setup a new website with Blowfish`. The following text appears:

```bash
Welcome to Blowfish tools.
I can help you setup a new project from scratch or configure an existing one (or both).
Please choose one of the options below, start typing to search.
✔ What do you need help with? · Setup a new website with Blowfish
✔ Hugo is available
✔ Git is available
? Where do you want to generate your website (. for current folder)? › newSite
```

**Create the website folder as a subdirectory of the Git repository.** If you enter `.` for the current folder, the process will be aborted following an error message.

Go through the options and change whatever is desired. Hit the Escape key to escape the interactive menu.

## Folder movements (this gets a little confusing)

Right, so far the folder structure of our Git looks something like this:

```bash
.
├── newSite
└── README.md
```

The `newSite/` folder was created by Blowfish and contains all the assets required to launch a Hugo static site.

There are two things to take note of.

1. `newSite/` contains a `.git/` folder, and its parent (our Git repo) also contains a `.git/` folder. The one in `newSite/` must be removed.
2. We need to move _everything_ in `newSite/` up one level, into that of the Git repo. This is because, later on, we will create a `.github/workflows/hugo.yaml` file, which will be evaluated in conjunction with the `config/` folder (which, when Blowfish initialised the website, was in the `newSite/` folder).

A bit confusing, yes? Yes, it is.

{{< alert "fire" >}}
Don’t forget to move all the **dotfiles** in `newSite/` up one level, too!
{{< /alert >}}

## Returning to the interactive menu

Use the same command to re-enter the interactive menu:

```bash
npx blowfish-tools
```

There is an important caveat here: **be at the top level when running this command.** If you are at any other level, the Blowfish installer will think that you are trying to create a brand new website and you’ll see the same message as that shown above.

You know you’re at the correct level when you see this:

```bash
Welcome to Blowfish tools.
I can help you setup a new project from scratch or configure an existing one (or both).
Please choose one of the options below, start typing to search.
? What do you need help with? …
Enter full configuration mode (all options)
Configure menus
Configure overall site
Configure site author
Configure homepage
Configure header
Configure footer
Configure article pages
Configure list pages
Configure taxonomy pages
Configure term pages
Configure images
Generate a new site section (e.g. posts)
Generate a new article
Run a local server with Blowfish
Generate the static site with Hugo
Update Blowfish installation
Exit
```

This is the interactive menu that is navigated through using the arrow keys. Do all the customisations that need doing before moving on to the next step, which is preparing the repo for deployment to GitHub Pages.

## Starting a local server

In the terminal, run `hugo server` and go to `http://localhost:1313` in any internet browser. Anytime changes are saved, they are immediately visible in the local version of the website.

## Overriding the default fonts

This took the longest amount of time to figure out and renewed my hatred of CSS. Since I found it an involved process (for someone with barely any experience with CSS), I’ve decided to make this [its own entry](https://iankohdes.github.io/posts/0002-overriding-default-fonts-in-blowfish/).

## Pre-deployment and deployment

Follow the steps listed [here](https://gohugo.io/host-and-deploy/host-on-github-pages/). Steps 1 to 4 are straightforward.

Step 5 involves specifying the following in a `hugo.toml` file:

```
[caches]
  [caches.images]
    dir = ':cacheDir/images'
```

Important question: where is the file located? If you look through your Hugo folder structure, you’d see that there are _two_ `hugo.toml` files: one at the top level, and one inside the `config/_default/` folder. We will add the above lines to the `hugo.toml` in the `config/_default/` folder.

With steps 6 and 7 we have two options. We could either make the relevant changes in the terminal, or we could use GitHub’s GUI to do this for us. In both cases the action is the same. First, we create a `.github/workflows/` folder _at the same level as_ the `config/` folder. Then, we add an empty `hugo.yaml` file into the `.github/workflows/` folder. Third, we paste the [following content](https://gohugo.io/host-and-deploy/host-on-github-pages/#step-7) into the empty YAML file.

Doing this in the terminal is simple. Once we are done we need to push our changes, and the CI/CD pipeline will take over deploying our website into production.

If we wish to take the GitHub GUI option, we first need to push our changes up to step 5. For steps 6 and 7, click on the Actions tab in the top ribbon. Click on the button that says ‘New workflow’ and search for the Hugo workflow. Select that and, once the page loads, there will be the step to create the `.github/workflows/hugo.yaml` file. This file is already pre-filled but you can replace its contents with that from the [Hugo documentation](https://gohugo.io/host-and-deploy/host-on-github-pages/#step-7). Commit the changes for the CI/CD process to start.

The CI pipeline has two nodes: `build` and `deploy`. Once both are successfully completed, the website will be live.

## Debugging: ‘Your push would publish a private email address.’

When pushing to Git, there might be an error that looks like this:

```bash
remote: error: GH007: Your push would publish a private email address.
```

This never happened to me in GitLab because I could keep my repo private. But this is GitHub, and in order to mitigate the issue one can consult [this Stack Overflow thread](https://stackoverflow.com/questions/43863522/error-your-push-would-publish-a-private-email-address). Simply put, set the global user e-mail to GitHub’s anonymised no-reply e-mail address. (This address can be obtained [here](https://github.com/settings/emails).)

```bash
git config --global user.email "{ID}+{username}@users.noreply.github.com"
```

This sets the e-mail address for all repositories. To set the e-mail addres for only one:

```bash
git config user.email "{ID}+{username}@users.noreply.github.com"
```

Next, reset the author information for the last commit:

```bash
git commit --amend --reset-author --no-edit
```

Finally, try pushing again. `git push` should work this time.

## Alert boxes (or callouts)

Unfortunately, this theme doesn’t implement callouts in the way that Obsidian does. It’s a bit more backwards and more inconvenient to write.

{{< alert "circle-info" >}}
This is how you create an _alert_ box. You can insert any icon listed [on this page](https://blowfish.page/samples/icons/) for added effect.
{{< /alert >}}

And this is what it looks like in code:

```text
{{</* alert "circle-info" */>}}
This is how you create an _alert_ box. You can insert any icon listed [on this page](https://blowfish.page/samples/icons/) for added effect.
{{</* /alert */>}}
```

## Adding a lead paragraph

{{<lead>}}
This is what a lead paragraph looks like. Use this at the beginning of a post to summarise its main idea.
{{</lead>}}

Followed by regular text. In code the syntax is:

```text
{{</* lead */>}}
This is what a lead paragraph looks like. Use this at the beginning of a post to summarise its main idea.
{{</* /lead */>}}
```

## Shortcodes

{{< alert "circle-info" >}}
Shortcodes are a kind of markup syntax.
{{< /alert >}}

Hugo has a tonne of additional functionality that is exposed by way of _shortcodes_. The [Blowfish documentation on shortcodes](https://blowfish.page/docs/shortcodes/) provides a thorough description of what’s available. The alert boxes in the previous section are one such example of shortcodes. The [icons](https://blowfish.page/samples/icons/) shortcode is another. {{< icon "star" >}}{{< icon "sun" >}}{{< icon "moon" >}}

By the way, [Mermaid diagrams](https://blowfish.page/docs/shortcodes/#mermaid) are created using shortcodes!

{{< keywordList >}}
{{< keyword >}}Keywords:{{< /keyword >}}
{{< keyword >}}learn{{< /keyword >}}
{{< keyword >}}to use{{< /keyword >}}
{{< keyword >}}shortcodes!{{< /keyword >}}
{{< /keywordList >}}
