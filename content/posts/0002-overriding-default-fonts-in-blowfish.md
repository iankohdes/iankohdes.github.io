---
title: "Overriding default fonts in the Blowfish theme"
date: 2025-05-31
draft: false
tags: ["hugo", "blowfish", "css"]
series: ["Setting up a Hugo website with the Blowfish theme"]
series_order: 2
---

{{<lead>}}
This follows from my [entry on setting up Hugo with the Blowfish theme](https://iankohdes.github.io/posts/0001-setting-up-hugo-and-blowfish/). I didn’t like that system-default sans-serif typefaces were used to render the site, and shuddered at the thought of Windows users seeing this site’s text in _Arial_.
{{</lead>}}

While attempting to override the default fonts, my experience was that the [official Blowfish documentation](https://blowfish.page/docs/advanced-customisation/#using-additional-fonts) was slightly incomplete. My inexperience also led me to create a few issues and I needed Gemini 2.5 to help me debug my way to success.

This entry covers the following points:

- Usage of `.woff2` font files instead of `.ttf`
- Adding variable-width fonts to the `custom.css` file
- Adding static (traditional) fonts to the `custom.css` file
- Adding a custom CSS parameter to `./config/_default/hugo.toml`
- Ensuring that `custom.css` is properly referenced by `head.html` (specific to the Blowfish theme)

Following all of them allowed me to use my desired typefaces.

## Using `.woff2` font files

The [Web Open Font Format (WOFF)](https://en.wikipedia.org/wiki/Web_Open_Font_Format) is a format for _web-only_ fonts. The files are compressed for fast loading and recognised by a multitude of browsers. The advice is to [always use WOFF2](https://web.dev/articles/font-best-practices#use_woff2) for web font formats since they come with performance benefits.

When fonts are downloaded from [Google Fonts](https://fonts.google.com/), however, they are provided as `.ttf` files. The [Yabe Webfont website has an online tool](https://webfont.yabe.land/en/misc/convert-ttf-woff2/) that can convert `.ttf` fonts into `.woff2` ones. It uses [Google’s **woff2** tool](https://github.com/google/woff2) for the conversion (and compression). I used the website to convert the fonts I downloaded from Google Fonts: [IBM Plex Sans](https://fonts.google.com/specimen/IBM+Plex+Sans) and [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono).

Once I had the font files ready, I added them all to the `./static/fonts/` folder (at the site root level).

## Adding variable-width fonts

First I created the `./assets/css/custom.css` file. This is where _all overriding CSS code_ should live, not just the syntax for overriding default fonts. I used the variable-width version of IBM Plex Sans, which comes in two files: one for the upright style and one for the italic.

```css
@font-face {
    font-family: "IBM Plex Sans Variable";
    src: url("/fonts/IBMPlexSans-VariableFont_wdth,wght.woff2") format("woff2-variations");
    font-weight: 100 700;
    font-style: normal;
}

@font-face {
    font-family: "IBM Plex Sans Variable";
    src: url("/fonts/IBMPlexSans-Italic-VariableFont_wdth,wght.woff2") format("woff2-variations");
    font-weight: 100 700;
    font-style: italic;
}

body {
    font-family: "IBM Plex Sans Variable", sans-serif;
    font-weight: 400;
    font-style: normal;
}

em {
    font-style: italic;
}

strong {
    font-weight: 700;
}
```

In the `@font-face` element, notice that we use the same `font-family` value for both the upright and italic font files. This provides a consistent name to reference for the CSS text elements. We distinguish between the upright and italic styles using the `font-style` field. As for `font-weight`, on Google Fonts we see that IBM Plex Sans has a weight ranging from 100 to 700 (with all possible integer values in between, such as 350, 239 and so on). We thus enter `100 700` to represent this range.

The `src` field has two values: `url` and `format`. For the URL, take note of the file path. From the site root level, our overriding CSS syntax lives in `./assets/css/custom.css` and our font files live in the `./static/fonts/` directory. This path doesn’t look at all like it’s relative to our `custom.css` file, and this is what confused me at first.

(Disclosure: I got this explanation from Gemini 2.5.) When Hugo builds the site, `./static/fonts/` becomes `/fonts/`. Once a CSS file is processed by Hugo and lives in the `public` directory, any relative files mentioned in said CSS file are resolved relative to the _generated CSS file’s_ location. By providing an absolute path starting with `/`, the browser looks for the font files (or any other resource) relative to the site root. Seeing as `.static/fonts/` is now just `/fonts/`, it is correct to start the path for `url` with that. In this regard the [official documentation’s example](https://blowfish.page/docs/advanced-customisation/#using-additional-fonts) is correct.

(Explanation also from Gemini.) Moving on, the `format(woff2-variations)` hint is needed to let the browser know that `IBMPlexSans-VariableFont_wdth,wght.woff2` is a variable font, which then helps with optimising rendering.

Now we are finally able to specify just _how_ we wish to override the default fonts. We do so using the `body`, `em` and `strong` elements, the latter two representing italicised and bold text respectively. Note that, for the `body`, the `font-style` should only be `normal` and _not_ `normal italic` (as a way of being lazy).

```css
/* Negative example */
body {
    font-family: "IBM Plex Sans Variable", sans-serif;
    font-weight: 400;
    font-style: normal italic; /* <-- Don’t do this! */
}
```

The normal font style sets the default for the body text, while `em` sets the style for italicised, or emphasised, text. The same logic holds for `strong`.

## Adding static fonts

The process is generally similar compared to variable-width fonts: first we specify `@font-face` so that Hugo (or the browser) knows where to find the font resources, and then we add the overriding syntax. I used the static JetBrains Mono for my website and explain my decision at the end of this section.

```css
@font-face {
    font-family: "JetBrains Mono";
    src: url("/fonts/JetBrainsMono-Regular.woff2");
    font-weight: 400;
    font-style: normal;
}

@font-face {
    font-family: "JetBrains Mono";
    src: url("/fonts/JetBrainsMono-Italic.woff2");
    font-weight: 400;
    font-style: italic;
}

@font-face {
    font-family: "JetBrains Mono";
    src: url("/fonts/JetBrainsMono-Bold.woff2");
    font-weight: 700;
    font-style: normal;
}

@font-face {
    font-family: "JetBrains Mono";
    src: url("/fonts/JetBrainsMono-BoldItalic.woff2");
    font-weight: 700;
    font-style: italic;
}

code {
    font-family: "JetBrains Mono";
}
```

Once again, we use a consistent `font-family` value and, in each instance of `@font-face`, we specify the characteristics of the font files using the `font-weight` and `font-style` fields. This is similar to what we did with the variable-width IBM Plex Sans fonts, except that we didn’t specify a range of weight values (since the fonts are static and each has only the one weight).

Overriding the default font is super easy: we just reference JetBrains Mono in the `code` element and we’re done!

_Now, since JetBrains Mono is also available as a variable-width font, why didn’t I want to use that in my `custom.css` file?_

Gemini explained that the ‘most common HTML elements’ for displaying code are:

- `<code>` for inline code snippets
- `<pre>` for preformatted text such as code blocks
- `<kbd>` for user input
- `<samp>` for sample output from a computer program
- `<var>` for a variable in a mathematical expression or programming context

This was the sample syntax that Gemini provided to override the default HTML elements:

```css
/* Apply JetBrains Mono to common code elements */
code,
pre,
kbd,
samp,
var {
    font-family: "JetBrains Mono", monospace; /* Always include a generic fallback like monospace */
    font-weight: 400; /* Default weight for code elements (e.g., Regular) */
    font-style: normal; /* Default style for code elements */
    /* You might want to adjust font-size if it looks too large/small compared to your body font */
    /* font-size: 0.9em; */
}

/* How to make code bold (like <strong>) */
/* If you have <strong> inside <code> or <pre> */
code strong,
pre strong,
kbd strong,
samp strong,
var strong {
    font-weight: 700; /* Or bolder, like 800 if you want ExtraBold */
}

/* How to make code italic (like <em>) */
/* If you have <em> or <i> inside <code> or <pre> */
code em,
code i,
pre em,
pre i,
kbd em,
kbd i,
samp em,
samp i,
var em,
var i {
    font-style: italic; /* This will automatically pull from JetBrainsMono-Italic-Variable.woff2 */
}

/* Example for custom bold code */
.bold-code {
    font-weight: 700;
}

/* Example for custom italic code */
.italic-code {
    font-style: italic;
}
```

**Too much effort.**

And thus, I went with the static JetBrains Mono fonts. If you’re reading this and want to customise your coding font with a variable-width one, go right ahead and try that syntax (I’ve not tested it for correctness).

## Custom fonts added but not recognised

After adding my custom CSS, I noticed that the site used the fall-back sans-serif typeface. This meant that my custom fonts weren’t loaded, and unfortunately this was also the part that the [Blowfish documentation](https://blowfish.page/docs/advanced-customisation/#using-additional-fonts) didn’t quite cover. I explored two options that I describe below, either of which should do the trick. (I ended up doing both but using one or the other is likely going to be fine.)

## Adding a custom CSS parameter

This is a simpler and neater alternative to making adjustments to `head.html` (covered in the next section). Go into `./config/_default/hugo.toml` and add this:

```css
[params]
  customCSS = ["css/custom.css"]
```

If this works, then great! If not then move to the next section.

## Making changes to `head.html`

Here, we directly add our custom CSS _after_ the theme’s CSS. It’s a more robust approach that should do the trick.

Open the `./themes/blowfish/layouts/partials/head.html` and copy all of its contents. Then paste them in a `./layouts/partials/head.html` file (create it if it’s not already there).

{{< alert "circle-info" >}}
It is generally not a good idea to modify files inside the `./themes/blowfish/` directory. By adding customisations to CSS and HTML files defined outside said directory, we maintain a separation of form and changes to the defaults. At least this is the advice I read in Hugo-related forums.
{{</alert>}}

Next, identify where the CSS-related elements are. For Blowfish’s `head.html`, this is a snippet of what the CSS-related portion of the file looks like:

```html
  <!-- This is a snippet! -->
  {{ $cssScheme := resources.Get (printf "css/schemes/%s.css" (.Site.Params.colorScheme | default "blowfish")) }}
  {{ if not $cssScheme }}
  {{ $cssScheme = resources.Get "css/schemes/blowfish.css" }}
  {{ end }}
  {{ $assets.Add "css" (slice $cssScheme) }}
  {{ $cssMain := resources.Get "css/compiled/main.css" }}
  {{ $assets.Add "css" (slice $cssMain) }}
  {{ $cssCustom := resources.Get "css/custom.css" }}
  {{ if $cssCustom }}
  {{ $assets.Add "css" (slice $cssCustom) }}
  {{ end }}
  {{ $bundleCSS := $assets.Get "css" | resources.Concat "css/main.bundle.css" | resources.Minify | resources.Fingerprint
  "sha512" }}
  <link type="text/css" rel="stylesheet" href="{{ $bundleCSS.RelPermalink }}"
    integrity="{{ $bundleCSS.Data.Integrity }}" />
```

Thank God they’re all bundled in the same place!

Now we add the following syntax _after_ the above CSS-related lines:

```html
  {{ $style := resources.Get "css/custom.css" | minify | fingerprint }}
  <link rel="stylesheet" href="{{ $style.RelPermalink }}">
```

With this change, the `custom.css` file (with the correct paths specified) and setting the custom CSS parameter in my `hugo.toml` file, I was able to change the site’s default typefaces to IBM Plex Sans and JetBrains Mono. Hopefully this thorough guide has helped and you’re now able to prettify your Blowfish-themed site.

{{< alert "comment" >}}
I have to say: Gemini was surprisingly helpful _and_ accurate throughout this entire exercise. Considering that Hugo themes can be somewhat variable in how they’re set up, the LLM saved me a lot of time with its responses and got my site to the state I wanted it to be without much hassle. Or maybe I just wrote really good prompts – who knows.
{{</alert>}}

## Concluding remarks

It’s a bit crazy that this entry about such a small issue is _longer_ than the [entry](https://iankohdes.github.io/posts/0001-setting-up-hugo-and-blowfish/) about setting up a themed Hugo site on GitHub Pages. Clearly I am a CSS noob. Front-end development has never been my thing and, with this recent experience, it will likely never be an interest I want to cultivate.

But with that said, I quite like how my site looks now. 😊✌🏼 Would I prefer to have built it using WordPress, Medium, Substack or Squarespace? Nah.

_Now to tackle the issue of overriding the default favicon…_