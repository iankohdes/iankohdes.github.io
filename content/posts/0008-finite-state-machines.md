---
title: "Finite state machines"
date: 2026-04-04
draft: false
tags: ["finite state machines", "concepts"]
---

A [finite state machine (FSM)](https://en.wikipedia.org/wiki/Finite-state_machine) is a behavioural model. It lets us design mechanisms to transition from one state to another. A key constraint is that our system can only be in one state at a time (i.e. states are mutually exclusive). As we’ll soon see, this constraint is an advantage.

{{< alert "comment" >}}
In software engineering, people often use both terms interchangeably. There is, however, a theoretical different between the two: FSMs have a finite number of states, while state machines have an *infinite* number of states and are therefore more general.
{{< /alert >}}

An FSM has three components:

- states (configurations of values),
- events (or inputs), and
- transitions.

## Finite state machines as graphs

The combination of a current state and an event triggers a transition to the next state. This might be easier to see as a directed – though not necessarily acyclic – graph.

{{<mermaid>}}
graph LR
    a((Start)) --> b((Read))
    b((Read)) -->|tired| c((Nap))
    c((Nap)) -->|awake| b((Read))
    b((Read)) -->|hungry| e((Snack))
    e((Snack)) -->|tired| c((Nap))
{{</mermaid>}}

From the trivial example above we can immediately see the relationships among the different states, along with specific constraints in system logic. For example, we know that it is possible to transition between the *Read* and *Nap* states in both directions. It is not possible, however, to transition from *Snack* back to *Read*; one can only return to the *Read* state through *Nap*. Therefore, the combination of the *Snack* state and *awake* input is illegal.

This highlights one of the most important (and coolest) aspects of FSM: **they allow us to make illegal states unrepresentable.**

Without an FSM, we would have had to model the above example as a series of if-then statements to get the same result.

```text
if state == Read && input == tired then Nap
if state == Read && input == hungry then Snack

if state == Snack && input == tired then Nap
⋮
```

Aside from looking clumsy, it’s very error prone. Someone could inadvertently slip in a statement like this:

```text
if state == Snack && input == awake then Read  // Illegal
```

But, as our FSM’s graphical representation clearly indicates, this is not possible and the statement is a potentially serious error in business logic.

## State-transition tables

When building an FSM, we want to map all allowable combinations of current states and inputs to new states. A tabular representation of this is known as a [state-transition table](https://en.wikipedia.org/wiki/State-transition_table#Two-dimensions), which is like a slightly more enriched version of the graph we saw earlier as it also shows error (illegal) states.

To illustrate this, here’s a more real-world use case for an FSM. Imagine we are parsing a subtitle file. The SRT format is probably the most popular out there, and it contains groupings of subtitles that are separated by single blank lines. Here’s a snippet of the subtitles from the 2014 film *Interstellar*:

```text
1682
02:14:27,643 --> 02:14:30,604
Cooper, you can’t ask TARS
to do this for us.

1683
02:14:30,813 --> 02:14:34,566
He‘s a robot. So you don’t have to
ask him to do anything.

1684
02:14:34,858 --> 02:14:36,193
<i>Cooper, you asshole!</i>

1685
02:14:36,485 --> 02:14:38,487
Sorry, you broke up a little bit there.

1686
02:14:38,821 --> 02:14:41,073
It’s what we intended, Dr. Brand.

1687
02:14:41,490 --> 02:14:43,784
It’s our only chance to save people on Earth.
```

If you’ve watched the film before, you’ll probably recall this scene. Notice that each grouping of subtitles starts with an index (unsigned integer), followed by a timing (start and end timestamps to display the subtitle text), and one or more lines of subtitles. Subtitle lines may contain simple HTML mark-up, as in the grouping with the index 1684. The parsing logic could therefore be represented by the following transition table.

| Current state | Input: *digit* | Input: *timestamps* | Input: *text* | Input: *blank line* |
| ------------- | -------------- | ------------------- | ------------- | ------------------- |
| *Empty*       | *Index*        | error               | error         | error               |
| *Index*       | error          | *Timing*            | error         | error               |
| *Timing*      | error          | error               | *Subtitle*    | error               |
| *Subtitle*    | error          | error               | *Subtitle*    | *Empty*             |

Notice how, in the table’s last row, there are two transitions. If a *Subtitle* state receives another line of text, then the state remains unchanged. The combination of a *Subtitle* state and *blank line* input serves as a reset to the *Empty* state, such that the parser is ready for a new subtitle grouping. The FSM of our subtitle parser thus looks like this:

{{<mermaid>}}
graph LR
	a((Start)) --> b((Empty))
	b -->|Digit| c((Index))
	c -->|Timestamps| d((Timing))
	d -->|Text| e((Subtitle))
	e -->|Text| e
	e -->|Blank line| b((Empty))
{{</mermaid>}}

## When not to use finite state machines

When we have:

- infinite state spaces,
- only two states (use Booleans instead), and
- states that are not mutually exclusive (i.e. users can toggle two or more states at the same time).
