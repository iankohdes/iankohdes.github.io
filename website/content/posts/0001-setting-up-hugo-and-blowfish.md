---
title: "Setting up Hugo and Blowfish"
date: 2025-05-30
draft: false
tags: ["hugo", "blowfish"]
---

This first post covers how to set up this blog. It is built with the Hugo static site generator and uses the [Blowfish theme](https://blowfish.page/). As this website’s source code is stored on GitHub, check out the [Hugo documentation](https://gohugo.io/host-and-deploy/host-on-github-pages/) for details on how to host the site on GitHub Pages. (GitHub’s default static site generator is Jekyll, so extra steps are required.)

I tried to customise the typefaces to use IBM Plex Sans and JetBrains Mono, but could never get the CSS to work so I gave up. This website uses the default settings and themes. _I hate CSS._

```rust
fn main() {
	let greeting: &str = "Hello, world!";
	println!("{greeting}");
}
```

## Hugo and Blowfish theme installation

This assumes that a Git repository for GitHub Pages has been created. Follow the steps [here](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) to get this prerequisite sorted. Clone the repository and navigate to its root level in the terminal.

The next steps are to install the required tools. I use [Homebrew](https://brew.sh/) to install the following:

- [Hugo](https://formulae.brew.sh/formula/hugo) (installation of Go isn’t necessary)
- [Node.js](https://formulae.brew.sh/formula/node) (this isn’t strictly necessary, but having it allows us to use Blowfish’s CLI tool for installation and configuration)

Find the Blowfish documentation for installation [here](https://blowfish.page/docs/installation/). I use the CLI-based `blowfish-tools` to install the theme.

```bash
> npx blowfish-tools
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

## Going back into the interactive menu

Use the same command to re-enter the interactive menu:

```bash
npx blowfish-tools
```

There is an important caveat here: **be at the level of the website directory when running this command.** If you are at any other level, the Blowfish installer will think that you are trying to create a brand new website and you’ll see the same message as that shown above.

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

This is the interactive menu that is navigated through using the arrow keys.

## Alert boxes (or callouts)

Unfortunately, this theme doesn’t implement callouts in the way that Obsidian does. It’s a bit more backwards and more inconvenient to write.

{{< alert "circle-info" >}}
This is how you create an _alert_ box. You can insert any icon listed [on this page](https://blowfish.page/samples/icons/) for added effect.
{{< /alert >}}

And this is what it looks like in code:

```raw
{{</* alert "circle-info" */>}}
This is how you create an _alert_ box. You can insert any icon listed [on this page](https://blowfish.page/samples/icons/) for added effect.
{{</* /alert */>}}
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
