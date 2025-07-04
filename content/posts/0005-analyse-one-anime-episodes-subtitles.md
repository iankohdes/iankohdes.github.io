---
title: "Analyse an anime episode’s (Japanese) subtitles"
date: 2100-01-01
draft: false
tags: ["anime", "term-frequency", "japanese", "subtitles", "psycho-pass"]
series: ["Anime Subtitle Project"]
series_order: 1
---

{{< alert "github" >}}
[Click here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles/tree/main) to access the GitHub repository.
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

Without giving too much away, the plot revolves around a police investigation unit that hunts criminals possessing what the series refers to as high ‘crime coefficients’. Based on an individual’s crime coefficient, an overarching AI system prescribes a mandatory response that ranges from stun-and-capture to immediate execution to instant vaporisation. No second chances, no appeals.

I liken _Psycho-Pass_ to a particularly violent and graphic version of _Ghost in the Shell: Stand-Alone Complex_.

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

I describe these metrics in a little more detail in the results section. But first, we start with the ingestion stage.

## Ingestion

Our very first step for this analysis sees us ingesting a raw subtitle file. I download the subtitle text for S01E01 of *Psycho-Pass* from [Jimaku](https://jimaku.cc/), a website that hosts subtitles for Japanese anime and even has API endpoints that one can use.

The ingestion process is simple: I read in a subtitle file and return a single, concatenated string. To understand what the process entails, it helps to have an idea of what subtitle files look like. These typically have an `.srt` extension and their contents consist of groupings of lines.

Here are the first 10 groupings of the subtitle file we’ll analyse.

```text
1
00:00:12,846 --> 00:00:24,899
♪～

2
00:00:46,921 --> 00:00:47,839
（狡噛(こうがみ)）フゥ～…

3
00:01:10,361 --> 00:01:11,112
（狡噛）うっ…！

4
00:01:14,324 --> 00:01:15,283
（狡噛）くそっ！

5
00:01:42,644 --> 00:01:47,440
（足音）

6
00:01:47,565 --> 00:01:50,235
（槙島(まきしま)）その傷で よくやるもんだ

7
00:01:52,654 --> 00:01:54,280
（朱(あかね)）きっと彼らは―

8
00:01:54,405 --> 00:01:56,157
一目 見て
分かったはずだ―

9
00:01:57,242 --> 00:01:59,994
２人は
初めて出会うより 以前から…―

10
00:02:00,161 --> 00:02:01,955
ああなる運命だったんだろう―
```

For convenience, I call such groupings **subtitle units**. Each subtitle unit has three general components:

- an index number,
- a pair of timestamps, and
- one or more lines of subtitle text.

After reading in the subtitle file, I *normalise* its contents before extracting the text. The reason is that newlines are represented by `\r\n`, indicating that the subtitles have been prepared on a Windows machine. (Unix-like machines represent newlines with `\n`.)

Below is a sample of the text when debug printed in Rust (before normalising).

```text
"\u{feff}1\r\n00:00:12,846 --> 00:00:24,899\r\n♪～\r\n\r\n2\r\n00:00:46,921 --> 00:00:47,839\r\n（狡噛(こうがみ)）フゥ～…\r\n\r\n3"
```

Normalisation involves replacing all instances of `\r\n` with `\n`:

```rust
let normalised_raw_content: String = raw_content.replace("\r\n", "\n");
```

Only after normalising am I able to extract the subtitle text from each unit and concatenate all of them into a single string. And with that, I’m now ready for the next phase: data preparation. This is split into two stages: *data cleaning* and *data processing*.

## Data cleaning

## Data processing
