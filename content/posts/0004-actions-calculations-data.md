---
title: "Decomposing code into actions, calculations and data"
date: 2025-06-09
draft: false
tags: ["grokking-simplicity"]
series: ["Notes on ‘Grokking Simplicity’ (Eric Normand, 2021)"]
series_order: 1
---

{{< alert "circle-info" >}}
Chapter 3: _Distinguishing actions, calculations and data_
{{< /alert >}}

What we really want with decomposing our code into actions, calculations or data is to separate out the actions as much as possible. The reason is that calculations and data are the more ‘replicable’ portions of our code, and are thus easier to reason about.

Put differently, we’re trying to minimise portions of code that might be trickier -- or just plain tricky -- to understand.

## Plan program with _timeline diagrams_

A timeline diagram describes, in broad detail, the steps a progam takes to do something. Each step is categorised as an action, calculation or a piece of data. The resulting diagram might look like this:

{{<mermaid>}}
graph LR;
A["Read text file (action)"] --> B["Clean data (calculation)"];
B --> C["Model data as records (calculation)"];
C --> D["Output as JSON (action)"];
{{</mermaid>}}

Next, we want to break each step down into smaller steps, keeping in mind the following:

- **Actions** can comprise actions, calculations and data

{{< alert "comment" >}}
In the case of actions comprising data, reading in a file is an action that is associated with text data stored in a file -- this is the level of granularity we want to break our initial timeline diagram down into.
{{</alert>}}

- **Calculations** can comprise calculations and data
- **Data** can only comprise more data

What’s nice about the last point is that data are generally static or inert. There are some downsides to this, which will be covered in the next section.

The author refers to the above as taking an **ACD perspective** to understanding, or planning, a program. I might adopt this term in future articles of this series.

## Data

We represent data with _data types_; these could be types from the standard library or types that we define ourselves, such as _sum_ or _product_ types. [(Refer to this article for more details.)](https://manishearth.github.io/blog/2017/03/04/what-are-sum-product-and-pi-types/)

Data’s usefulness comes from its characteristics:

- It can be transmitted into and stored as different forms because it is **serialisable**. This means that it could live forever – it certainly outlives actions and calculations.
- It can be **compared** with another piece of data for equality, which is something that we cannot do with functions and actions.
- The same piece of data can be applied in different use cases as it is **open to interpretation**, which makes it flexible. Note that this is a double-edged sword, because without the right contextual information, it may not be clear how to interpret a given piece of data.
- Its **inertness** allows its structure to be easily understood (even if one might need additional context to interpret it correctly).

## Calculations

A calculation is some computation that takes an input (or inputs) and returns an output. Repeatedly running this computation using the same inputs will not change the value of the output. This makes calculations akin to the mathematical definition of functions, and what distinguishes them from actions.

Compared to actions:

- Calculations are **easier to test** because one can always reason about the output based on the input(s).
- **Automated checks, or static analysis,** can be more easily performed.
- Calculations are **highly composable** since a calculation’s output can be used as another calculation’s input.
- One doesn’t have to worry about the **order** in which things are run. (Think about an action that takes no inputs and instead uses a global variable: the latter must be initialised before the action can be run.)

A calculation’s main drawback -- which is also one shared with actions -- is that **to see what it does, one needs to run it**. For short calculations one might be able to get away with inspecting the source code, but what if a calculation is imported from elsewhere, or is very long and/or complicated? Running said calculation would be the easiast way to see how it works.

Here, data have an advantage over calculations. The former do not require running for users to know what they do. Where feasible, it might thus make more sense to use data instead of calculations.

## Actions

Actions either affect their environment or are affected by their environment. (One often reads that actions, or side effects, are anything that interact with the world, but what this ‘world’ is is often undefined, which I find unfortunate.)

Here are some examples of actions:

- Calls to side-effecting functions and methods. Such functions could print to the terminal, or open a window with a visualisation, or send an e-mail, or make a call to an API or database, and so on.
- Constructors, such as getting the current timestamp.
- Writing to mutable variables (global or otherwise).
- Reading from mutable variables (global or otherwise).

{{< alert "comment" >}}
Why would we say that the above examples are actions? The reason is that their results vary depending on:

- The **order** in which they are run
- **How often** they are run

This notion of actions being **time-dependent** is good to keep in mind.
{{</alert>}}

Actions pose a dilemma because they can make code difficult to work with, but all real-world software needs them to be useful. Here are some approaches for working with them:

- **Use fewer actions.** Where possible, use calculations instead of actions.
- **Keep actions small.** Refactor an (existing) action to extract as many calculations and pieces of data out of it as possible.
- **Restrict actions to interactions with the outside environment.** Avoid global mutable variables and minimise (non-global) mutable ones. Ideally, work only with calculations and data.
- **Limit an action’s dependence on time.** In other words, limit the chances for an action’s results to vary depending on the order in which it is run and how often it is run.

## Example

Normand provides an example to show the ACD perspective in action (pun unintended). I’ll use a subset of the example here, and will also break down the steps using my own way of thinking about it.

The situation is this: we want to send the best-value discount coupons to our high-value customers. A simple timeline diagram of _actions_ might look like this:

{{<mermaid>}}
graph LR;
A["Get list of high-value customers"];
B["Get list of best-value discount coupons"];
C["Send best-value coupons to high-value customers"];
A --> C;
B --> C;
{{</mermaid>}}

## Fetching customers and coupons

Customer and coupon information count as **data** and live in databases. Fetching them is an **action**. Now, assuming that the databases already have dimensions that indicate which customers are high-value and which coupons are best-value, we can filter on these dimensions and get the records we want.

### Modelling customer and coupon information

After retrieving customer and coupon records, we still need to structure them in a way that we can work with in later steps. This entails designing **calculations**, and here we want to keep the modelling of the data _separate_ from its retrieval. In doing so, we can test our calculation and ensure that it returns the right data types every time.

In the case where the databases do not have dimensions allowing us to filter on the criteria we want, we add an additional step. Having extracted the data and created types, we want to write more calculations containing _business logic_ to identify our desired customers and coupons.

By keeping the modelling and business logic calculations separate, we allow for them to be reusable. In future situations, if someone needs to model the data and apply business logic, they can _compose_ both calculations together. Otherwise, they can pick and choose whichever calculation suits them best.

### Prepping the e-mails

We might think of sending an e-mail as a single action. This is certainly true, but we could extract data to keep the action as small as possible. An e-mail has specific fields that are required for it to be sent:

- Sender
- Recipient
- Subject
- Body

These can be prepared in advance by way of a record type. We could then write a calculation to feed coupons to this type, perhaps writing yet more calculations to decide how many coupons or which ones to include.

### Sending the e-mails

Now that we have a collection of e-mail records that are populated with customer, coupon and communication details, we are ready to send them out. In this step, sending the e-mails is just that -- a loop of actions that sends them to their recipients. There are no calculations required.

This means that, if we encounter a problem sending the e-mails, we don’t have to worry about whether a calculation might be responsible for the error. This frees up time to investigate network issues and anything else that might disrupt the process of sending out an e-mail. In more straightforward terms, debugging is now easier.

## Conclusion

The ACD perspective incentivises planning code before writing it. While this adds considerable time to the development process, it also gives the programmer a very clear picture of what’s going on.

Conveniently, decomposing code into its action, calculation and data components contributes towards architecting a piece of software. The components can subsequently be visualised using tools like the [C4 model](https://c4model.com/) for sharing with colleagues.