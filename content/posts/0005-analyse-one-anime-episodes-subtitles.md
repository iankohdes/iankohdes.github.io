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

Ever thought of analysing subtitles written in one of the world’s most complicated writing systems? I have. Using the Japanese-language subtitles of one episode of an anime series, I calculate some metrics and present my findings.

It’s my first attempt at natural language processing (NLP) so I wish to keep things simple. This means focusing on Japanese characters, not words. (In Chinese and Japanese, a word could comprise one or more characters.) Put differently, this analysis features no tokenisation. In future, I hope to explore subtitles at the word level and present analyses of greater complexity and depth.

For now, allow me to start with some contextual information that’ll be helpful for understanding what follows.

## Context and high-level overview

My analysis has the following steps:

- retrieval and ingestion of source data (the `.srt` file),
- cleaning,
- processing, and
- calculation of metrics.

I’ll describe these metrics momentarily.

To start with, I have selected the first episode of the first season of _Psycho-Pass_, hereafter referred to as S01E01. The series was produced by [Production I.G](https://www.production-ig.co.jp/) and released in 2012. [Wikipedia](https://en.wikipedia.org/wiki/Psycho-Pass) describes it as a ‘cyberpunk psychological thriller’ set in a dystopian 22nd-century Japanese society.

Without giving too much away, the plot revolves around a police investigation unit that hunts criminals possessing what the series refers to as high ‘crime coefficients’. Based on an individual’s crime coefficient, an overarching AI system prescribes a mandatory response that ranges from temporary paralysis to immediate execution to instant vaporisation. No second chances, no appeals.

I liken _Psycho-Pass_ to a particularly violent and graphic version of _Ghost in the Shell: Stand-Alone Complex_.

Coming back to the topic at hand, I have **two key interests**: the programming aspect (which covers data preparation and processing) and the analysis aspect.

I thus spend a great deal of time on the former. The reason is that I intend to implement this phase in [Rust](https://www.rust-lang.org/). (I’m learning the language and this is a good use case for project-based learning.) I recognise that Rust is a non-standard language of choice in an NLP context, and have chosen it because

- it is statically typed,
- it has a strong type system and I can easily create sum and product types to better model my domain,
- it compiles to binary so I don’t have to grapple with virtual machines like the JVM, and
- it has the most user-friendly error messages I’ve ever encountered (compared to other languages I’ve used like Python, Kotlin and Haskell).

A cynic might comment that I’ve chosen to make life unnecessarily difficult for myself, which is fair. Perhaps I’ll find that I’ve bitten off more than I can chew and switch back to Python for future NLP analyses. Perhaps not. 🙃

Now that you have the overall context, let me introduce the metrics I’ll calculate, because the entire data preparation process lays the foundation for them.

## Metrics

- [Term frequency](https://www.seobility.net/en/wiki/Term_Frequency) (or TF)
    - Calculated for all categories of characters.
    - Calculated per category of [kanji](https://en.wikipedia.org/wiki/Kanji), [hiragana](https://en.wikipedia.org/wiki/Hiragana) and [katakana](Katakana).
- Proportion of _unique_ kanji that are [_jōyō_ kanji](https://en.wikipedia.org/wiki/J%C5%8Dy%C5%8D_kanji).
- Proportion of _unique_ kanji that are [_hyōgai_ kanji](https://en.wikipedia.org/wiki/Hy%C5%8Dgai_kanji).
- Proportion of _non-unique_ characters that are katakana.

I describe these metrics in a little more detail in the results section. We first move on the ingestion stage.

## Ingestion

{{< alert "github" >}}
The ingestion code is stored in `ingestion.rs` and can be viewed [here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles/blob/main/src/dataprep/ingestion.rs).
{{</alert>}}

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

Only after normalising am I able to extract the subtitle text from each unit and concatenate all of them into a single string. The entire ingestion step looks like this.

```rust
pub fn ingest_subtitle_file(filepath: &str) -> std::result::Result<String, Box<dyn std::error::Error>> {
    let raw_content: String = fs::read_to_string(filepath)?;
    println!("{raw_content:?}");
    let normalised_raw_content: String = raw_content.replace("\r\n", "\n");

    let subtitle_units: Vec<&str> = normalised_raw_content.split("\n\n").collect();
    let subtitles: String = subtitle_units
        .iter()
        .flat_map(|x| get_subtitles_from_unit(x))
        .collect();

    Ok(subtitles)
}

fn get_subtitles_from_unit(subtitle_unit: &str) -> Vec<&str> {
    subtitle_unit.split('\n').skip(2).collect()
}
```

One might question the need for the `get_subtitles_from_unit` function, since all it does is apply a method chain to a subtitle unit. The chain could be used directly in the [closure](https://doc.rust-lang.org/book/ch13-01-closures.html) in `ingest_subtitle_file` instead.

I’ve chosen to retain `get_subtitles_from_unit` as it makes the code neater and more readable. I’m also able to add internal documentation to the function and explain _why_ I’m specifically skipping the first two elements. (You can read the documentation in `ingestion.rs`.)

It is particularly useful that one can simply collect the extracted subtitle text into a single value of `String` type. `collect` makes life so much easier!

With that, I’m now ready for the data preparation phase. This is split into two stages: *data cleaning* and *data processing*.

## Data cleaning

{{< alert "github" >}}
All cleaning functions and data types are stored in `cleaning.rs` and can be viewed [here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles/blob/main/src/dataprep/cleaning.rs).
{{</alert>}}

Data cleaning is done by a function called `clean_subtitles`. Its only argument is a (borrowed) string, which is nice because it's scalable – I could apply `clean_subtitles` on text from one subtitle unit, or on a concatenated string containing text from an entire anime series. In this section we take a look at what happens behind the scenes.

Cleaning involves the following three steps that are executed _in this order_:

- remove parentheses and their contents,
- remove unwanted characters, and
- convert small kanas to their regular-sized counterparts.

`clean_subtitles` returns a `Result` that, in the successful case, can be unwrapped into a `String` type.

### Remove parentheses and their contents

To understand why we _don’t_ want content enclosed in parentheses, it’s helpful to see what such subtitles look like.

```text
2
00:00:46,921 --> 00:00:47,839
（狡噛(こうがみ)）フゥ～…
```

In this subtitle unit we see two types of parentheses. One is the ‘regular’ pair that most of the world uses, `()`. The other, `（）`, is used in East Asian languages like Japanese and Chinese.

Parenthesised content provides contextual information that is unavailable to viewers who do not toggle the subtitles. (Examples include character names and descriptions of sounds.) Since they aren’t part of the dialogue, I prefer to exclude them from further analysis.

The core logic uses a counter whose range of values is always positive (and includes zero) due to the `saturating_sub` method. When processing one character in a string, if the counter is at `0`, the character is retained. If the counter has any value other than `0`, the character is removed. The counter’s value increments or decrements depending on whether the logic encounters an opening or closing parenthesis.

```rust
fn remove_parentheses_and_contents(input: &str) -> String {
    let mut result = String::new();
    let mut depth: u32 = 0;

    for char in input.chars() {
        match char {
            '(' | '（' => depth += 1,
            ')' | '）' => depth = depth.saturating_sub(1),
            _ if depth == 0 => result.push(char),
            _ => {}
        }
    }

    result
}
```

A weakness of the logic, as seen in the above code listing, is that it does not check for equal numbers of opening and closing parentheses. An excess of closing parentheses likely has a limited impact since decrements are saturated at zero; an excess of opening parentheses, however, could severely truncate a string and return an incorrect value.

I have no intention to address this issue in the current project. My stance might change in a subsequent one if need be.

### Remove unwanted characters

This is probably the most self-explanatory of the cleaning steps. Yet, because Japanese is not alphabet based, defining the set of unwanted characters isn’t as straightforward as one might hope.

{{< alert "language" >}}
The Japanese writing system uses _three_ scripts at the same time: kanji, hiragana and katakana. All are derived from Chinese characters: kanji is fully based on (traditional) Chinese characters, hiragana is based on cursive writing and katakana is based on fragments of Chinese characters.

Each script has its own use case. Kanji is used for [content words](https://en.wikipedia.org/wiki/Content_word) and can also be written in hiragana. Hiragana representations of kanji aid in pronunciation, and hiragana characters also indicate grammatical features like particles and subject markers. Finally, katakana is used for loan words and technical terms.
{{</alert>}}

It’s quite easy to filter out characters in the hiragana and katakana [syllabaries](https://en.wikipedia.org/wiki/Syllabary), because they are relatively few and amount to 100+ characters in total. The full set of kanji, on the other extreme, amounts to between 40,000 and 50,000 characters.

The majority of kanji are rarely or never used in daily life. One could thus adopt a pragmatic approach and use a set of commonly-used kanji as a basis for filtering. This will work most of the time, but risks accidentally filtering out rarely-used kanji if they should appear in a subtitle unit. More importantly, one would first need to compile such a list.

{{< alert "comment" >}}
This list does exist, by the way! I have a dictionary collection of a little over 13,000 kanji with definitions, stroke counts and pronunciation guides, among other metadata. I shan’t use it in this project but intend to do so in future ones.
{{</alert>}}

Instead, I take a more manual approach to defining my blacklist of unwanted characters. Taking the single, concatenated string of subtitle text for S01E01, I deduplicate it and sort its characters.

My reasoning  – and hope – is that the kanji, hiragana and katakana characters would be lumped together, while all other unwanted characters are bunched together before and after the Japanese ones. In this instance my deduction pays off, and this is the result:

```text
 ()01269―…♪々あいうえおかがきぎくぐけげこごさざしじすずせぜそぞただちっつづてでとどなにぬねのはばぱびぶへべぼぽまみむめもゃやょよらりるれろわをんァアィイゥウェエォオカガキギクグケゲコサザシジスセタダチ刻前剤力助効動包区千厄去取受口可合同向君告味命員問噂噛器回囲圧在地型執基報場塊塚声大夫奮女好威娘婆婚嫌嬢子守安完官定宜宣害家寄対専小就尽局届属島巣己席帯常年度座廃引張強当影役彼征待後得心忘応念怒思性悟望本朱来柄染根格械棄棒検様槙機欲止正死段殺民気求治況泥活流浪浮深混準潜濁災無照片物犬犯状狙狡狩猟獣獲現理生用画界療発登的皆監目相真眠着知研破確社私窟立笛米精納紹終結給絶継続綻緊緒締練縢繰罪羽老考者耐聖香駆高鳴！（）１２３４？ＫＴ～･
```

The Japanese characters are sorted such that hiragana appears first, followed by katakana and finally by kanji.

Take a look at the unwanted characters at the end of the string. These are full-width and thus intended to complement Japanese words. We also see the Japanese tilde, ～, which differs noticeably from its Western counterpart, ~.

{{< alert "language" >}}
In the above listing, you’ll spot characters with slight variations of themselves. These variations contain diacritics, of which Japanese has two types: [_dakuten_ and _handakuten_](https://en.wikipedia.org/wiki/Dakuten_and_handakuten).

Take the following as an example:

```text
かきくけこ
がぎぐげご
```

The characters か, き, く, け and こ represent the K-row of the hiragana table and are pronounced [ka], [ki], [ku], [ke] and [ko] respectively. These characters’ pronunciations undergo a slight change when the _dakuten_ is applied, such that the syllable-initial plosive [k] becomes voiced: [ga], [gi], [gɯ], [ge] and [go].
{{</alert>}}

At any rate, my list of unwanted characters can be compiled by visual inspection:

```text
()01269―…♪ ！（）１２３４？ＫＴ～･
```

Removing these characters is a matter of filtering the concatenated string for characters that are _not_ in this list.

### Convert small kanas to their regular-sized counterparts

{{<alert>}}
I will likely remove this cleaning step in future analyses when I start tokenising my text. Small kana convey important information and converting them into regular size strips that information away. In the worst-case scenario, conversion could interfere with tokenisation and return inaccurate results.
{{</alert>}}

As we learnt in the previous section, the kana syllabaries are used for grammatical elements, loan words and technical terms. Hiragana is also used as an alternative script to kanji. There is a complication to the kana scripts, and that is the existence of small kana characters in both hiragana and katakana.

{{< alert "language" >}}
Certain hiragana and katakana characters have smaller versions of themselves. Small kana are more prevalent in katakana than in hiragana, although not all of the small katakana are used in Japanese – several are used in the [Ainu language](https://en.wikipedia.org/wiki/Ainu_language), which is [written](https://en.wikipedia.org/wiki/Ainu_language#Special_katakana_for_the_Ainu_language) entirely using the katakana syllabary.

Within Japanese, the small kana are used for purposes such as

- forming digraphs ([_yōon_](https://en.wikipedia.org/wiki/Y%C5%8Don)),
- extending vowel length,
- indicating double consonants (っ/ッ only), and
- indicating glottal stops at the ends of words or sentences.

っ and ッ belong to the hiragana and katakana syllabaries respectively. These syllabaries have a one-to-one mapping to each other, and thus have identical numbers of characters (not counting the extended katakana characters used outside Japanese).

**Small note:** small kana are _not_ analogous to the lowercase letters used in Latin-based alphabets. Japanese is not an alphabet-based language.
{{</alert>}}

While the small kana are distinct from their regular-sized counterparts, I convert them to regular size for simplicity’s sake. (The alternative would be to remove them from the analysis.)

Below is the function that performs the conversion. The value that is passed as the second parameter, `kana_mapping`, is created in the `clean_subtitles` function. We’ll soon see its creation in the next subsection.

```rust
#[derive(Deserialize, Eq, PartialEq, Hash, Debug)]
struct SmallKana(char);

#[derive(Deserialize, Eq, PartialEq, Hash, Debug)]
struct RegularKana(char);

fn convert_mini_kana_to_regular(
    input: &char,
    kana_mapping: &HashMap<SmallKana, RegularKana>,
) -> char {
    let typed_input = SmallKana(*input);

    let unwrapped_output: char = match kana_mapping.get(&typed_input) {
        Some(regular_kana) => regular_kana.0,
        None => typed_input.0
    };

    unwrapped_output
}
```

`convert_mini_kana_to_regular` reminds me a bit of Haskell because of the pattern matching, newtypes (`SmallKana` and `RegularKana`) and `Option` type. Even the derivation of the traits is similar to defining a new type and then deriving type classes to give it some default functionality.

### Full cleaning function

It’s difficult to visualise how the cleaning steps work together to clean a string of subtitle text. Here’s the full cleaning process for context, with the internal documentation stripped out for clarity. (You can read it in `cleaning.rs` if you wish.)

```rust
const MINI_KANA_JSON_PATH: &str = "data/raw/mini_kana_mappings.json";
const UNWANTED_CHARACTERS_PATH: &str = "data/raw/unwanted_characters.txt";

#[derive(Deserialize, Eq, PartialEq, Hash, Debug)]
struct SmallKana(char);

#[derive(Deserialize, Eq, PartialEq, Hash, Debug)]
struct RegularKana(char);

pub fn clean_subtitles(raw_input: &str) -> Result<String, Box<dyn std::error::Error>> {
    let unwanted_characters_raw = fs::read_to_string(UNWANTED_CHARACTERS_PATH)?;
    let unwanted_characters: HashSet<char> =
        unwanted_characters_raw.chars().collect();

    let mini_kana_mappings: HashMap<SmallKana, RegularKana> =
        ingest_json_file(MINI_KANA_JSON_PATH)?;

    let parentheses_and_their_contents_removed: String =
        remove_parentheses_and_contents(raw_input);

    let unwanted_chars_removed_and_small_kana_as_regular: String =
        parentheses_and_their_contents_removed
            .chars()
            .filter(|x: &char| !unwanted_characters.contains(x))
            .map(|x: char| convert_mini_kana_to_regular(&x, &mini_kana_mappings))
            .collect();

    Ok(unwanted_chars_removed_and_small_kana_as_regular)
}

fn remove_parentheses_and_contents(input: &str) -> String {
    let mut result = String::new();
    let mut depth: u32 = 0;

    for char in input.chars() {
        match char {
            '(' | '（' => depth += 1,
            ')' | '）' => depth = depth.saturating_sub(1),
            _ if depth == 0 => result.push(char),
            _ => {}
        }
    }

    result
}

fn convert_mini_kana_to_regular(
    input: &char,
    kana_mapping: &HashMap<SmallKana, RegularKana>,
) -> char {
    let typed_input = SmallKana(*input);

    let unwrapped_output: char = match kana_mapping.get(&typed_input) {
        Some(regular_kana) => regular_kana.0,
        None => typed_input.0
    };

    unwrapped_output
}
```

At this point, we have finished the first stage of the data preparation phase. The next stage processes the cleaned data for use in the analysis.

{{< alert "circle-info" >}}
In `clean_subtitles`, I apply a function called `ingest_json_file` but have said nothing about it so far. Its code is found in `ingestion.rs` – [click here](https://github.com/iankohdes/examining-one-anime-episodes-subtitles/blob/main/src/dataprep/ingestion.rs) to view it.
{{</alert>}}

## Data processing
