---
title: "Analysing an anime episode’s subtitles"
date: 2025-06-18
draft: false
tags: ["anime", "term-frequency", "japanese", "subtitles", "psycho-pass"]
series: ["Exploring and analysing an anime episode’s (Japanese) subtitles"]
series_order: 1
---

{{< alert "github" >}}
[Click here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles) for the GitHub repository.
{{</alert>}}

## Background

For a long time now, I’ve thought that working with text might be an interesting pursuit. I’ve always enjoyed writing, so combining the written word with programming and data analysis suits me nicely. The only problem was that I’ve never been able to think of a suitable topic, in the sense that I could never find a topic that I’d be personally interested or invested in.

That’s recently changed thanks to a few pieces of information coming together in my head. First, I’ve got a trip to Japan towards the end of the year. Second, I learnt not too long ago that the Japanese language has the most complicated writing system on the planet. Third, I know that people upload subtitles of series and films on the web in various languages. This holds for Japanese anime, too.

Finally, years ago I read a computational linguistics module at university. It was my first exposure to programming (we used Python), and I remain convinced that I would have failed had it not been for the help of a kindly computer science student who likely enrolled in the module for a free A. Over a decade later, I’m happy to report that I now can program a little.

And, with that, it wasn’t difficult for my attention to fall on analysing Japanese subtitles. This article shall be the first in a series, and the proceeding sections introduce my plan in a high-level fashion.

{{< alert "graduation-cap" >}}
To Duc Nguyen: we’ve not spoken or seen each other since that module, but thanks for your help getting my super inefficient Python code to run! I still smile when I recall how everyone’s code took _hours_ to complete, while yours was done within 10 seconds.
{{</alert>}}

## Introduction

The idea is to retrieve the Japanese language subtitles of one anime episode, process and clean the text, and calculate some  metrics which I’ll then present (hopefully as a visualisation). Since this is my first exposure to natural-language processing (outside university), one of my goals is to keep this as simple as possible to start with.

Specific details on the subtitle file and how I retrieve it are described in a subsequent article.

For the analysis, I’ve settled on the first episode of the first season of _Psycho-Pass_, hereafter referred to as S01E01. (There are three seasons in total.) The series was produced by [Production I.G](https://www.production-ig.co.jp/) and released in 2012. [Wikipedia](https://en.wikipedia.org/wiki/Psycho-Pass) describes it as a ‘cyberpunk psychological thriller’ set in a dystopian 22nd-century Japanese society.

I would’ve picked one of the _Gundam_ series since I rather enjoy the mecha genre of anime, but decided to go for something very different at the last minute. Now that I’ve also started rewatching the first season of _Psycho-Pass_, I might as well commit and analyse its subtitles.

## Data preparation

This project, for me, is as much about the technical aspects of preparing the data as it is about analysing it. I wish to see how I can model the subtitle data and work with it in a type-driven way.

Furthermore, as this is a hobby project, I’m going to use a statically-typed language to prepare the text. Python would be a more appropriate choice for sure, but I’m a glutton for punishment and want to experience firsthand the challenges of prototyping with a strong type system. (You’ll soon see which language this is, but you might have guessed that it isn’t Go.)

## Metrics

{{< alert >}}
It is important to note that this analysis is conducted at the **level of individual characters, not words.** To keep things simple, no tokenisation is performed, which means I shan’t split the subtitle text into components such as parts-of-speech.

I will mention this repeatedly throughout the series.
{{</alert>}}

- [Term frequency](https://www.seobility.net/en/wiki/Term_Frequency) (or TF)
    - Overall
    - Per category of [kanji](https://en.wikipedia.org/wiki/Kanji), [hiragana](https://en.wikipedia.org/wiki/Hiragana) and [katakana](Katakana)
- [_Jōyō_ kanji](https://en.wikipedia.org/wiki/J%C5%8Dy%C5%8D_kanji):
    - Percentage of unique kanji in the episode that are found in the _jōyō_ kanji
    - Percentage of _jōyō_ kanji that are represented in the episode
- [_Kyōiku_ kanji](https://en.wikipedia.org/wiki/Ky%C5%8Diku_kanji):
    - Percentage of unique kanji in the episode that are found in the _kyōiku_ kanji
    - Percentage of _kyōiku_ kanji that are represented in the episode
- Proportion of unique kanji that are [_hyōgai_ kanji](https://en.wikipedia.org/wiki/Hy%C5%8Dgai_kanji)
- Percentage of (non-unique) characters that are katakana

Metric definitions to follow in a subsequent article.

## Next step

I have my idea, and I have my metrics. Now I need to prepare and shape the data before I can start thinking about materialising these metrics. That’s what the next article will focus on. Factoids about the Japanese language, anime and subtitle files will be introduced as necessary.