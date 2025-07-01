---
title: "Analyse an anime episode’s (Japanese) subtitles"
date: 2100-01-01
draft: false
tags: ["anime", "term-frequency", "japanese", "subtitles", "psycho-pass"]
series: ["Anime Subtitle Project"]
series_order: 1
---

{{< alert "github" >}}
[Click here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles/tree/main) for the link to the analysis’s GitHub repository.
{{</ alert >}}

Ever thought of analysing subtitles written in one of the world’s most complicated writing system? I have, and that’s the basis for this post. Taking a look at the Japanese-language subtitles of one anime episode, I calculate some metrics and present my findings in subsequent sections.

It’s also my first attempt at natural language processing (NLP), which is why I want to keep things simple. This means looking at Japanese characters, not words. (In Chinese and Japanese, a word could comprise one or more characters.) In future, I hope to explore subtitles at the word level, and present analyses of greater complexity and depth.

{{< alert >}}
This analysis is restricted to Japanese **characters**, not words, and therefore doesn’t feature any tokenisation.
{{</ alert >}}

For now, allow me to start with some contextual information that’ll be helpful for understanding what follows.

## Context and high-level overview

My analysis has the following steps:

- retrieval and ingestion of source data (the `.srt` file),
- cleaning,
- processing, and
- calculation of metrics.

I’ll describe these metrics momentarily.

To start with, I have selected the first episode of the first season of _Psycho-Pass_, which I’ll refer to as S01E01 hereafter. The series was produced by [Production I.G](https://www.production-ig.co.jp/) and released in 2012. [Wikipedia](https://en.wikipedia.org/wiki/Psycho-Pass) describes it as a ‘cyberpunk psychological thriller’ set in a dystopian 22nd-century Japanese society.

Without giving too much away, the plot revolves around a police investigation unit that hunts criminals possessing what the series refers to as high ‘crime coefficients’. Based on an individual’s crime coefficient, an overarching AI system prescribes a response that could range from stun-and-capture to immediate execution to instant vaporisation. No second chances, no appeals.

I liken _Psycho-Pass_ to a particularly violent and detailed version of _Ghost in the Shell: Stand-Alone Complex_.

Coming back to the topic at hand, I have **two key interests**: the programming aspect (which covers data preparation and processing) and the analysis aspect.

I thus spend a great deal of time on the former. The reason is that I intend to implement this phase in [Rust](https://www.rust-lang.org/). (I’m learning the language and this is a good use case for project-based learning.) I recognise that Rust is a non-standard language of choice in an NLP context, and have chosen it because:

- it is statically typed,
- it has a strong type system and I can easily create sum and product types to better model my domain,
- it compiles to binary so I don’t have to grapple with virtual machines like the JVM, and
- it has the most user-friendly error messages I’ve ever encountered (compared to other languages I’ve used like Python, Kotlin and Haskell).

A cynic might comment that I’ve chosen to make life unnecessarily difficult for myself, which is fair. Perhaps I’ll find that I’ve bitten off more than I can chew and switch back to Python for future NLP analyses. Perhaps not. 🙃

Now that you have the overall context, let’s consider the metrics I’ll calculate, because the entire data preparation process lays the foundation for them.

## Metrics

- [Term frequency](https://www.seobility.net/en/wiki/Term_Frequency) (or TF)
    - Calculated for all categories of characters.
    - Calculated per category of [kanji](https://en.wikipedia.org/wiki/Kanji), [hiragana](https://en.wikipedia.org/wiki/Hiragana) and [katakana](Katakana).
- Proportion of _unique_ kanji that are [_jōyō_ kanji](https://en.wikipedia.org/wiki/J%C5%8Dy%C5%8D_kanji).
- Proportion of _unique_ kanji that are [_hyōgai_ kanji](https://en.wikipedia.org/wiki/Hy%C5%8Dgai_kanji).
- Proportion of _non-unique_ characters that are katakana.

I will describe these metrics in a little more detail once we get to the results section.

Next, we move on to the data preparation phase. This is split into two stages: _data cleaning_ and _data processing_.

## Data cleaning

## Data processing
