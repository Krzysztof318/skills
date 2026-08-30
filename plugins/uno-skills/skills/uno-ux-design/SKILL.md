---
name: uno-ux-design
description: Use when designing an Uno Platform screen, page, or feature - before or while writing the XAML. Turns a brief into structure - the one action, the hierarchy, the density, the states, the motion, and the few style decisions worth making. Trigger on "design this screen", "how should this page work", "what does this feature need", or when a screen is about to be built and no layout exists yet. Pairs with uno-design-review, which grades what this skill produces.
license: MIT
metadata:
  author: Krzysztof Kasprowicz
  copyright: Copyright © 2026 Krzysztof Kasprowicz
  repository: https://github.com/Krzysztof318/skills
---

# Uno UX Design

A screen is a set of decisions, and a green build proves nothing about them. This skill is the first
half of shipping a good screen: it turns a brief into structure before XAML exists, so that what
gets built is something `uno-design-review` can pass rather than something it has to reject.

The Studio plugin answers *how* - how a `Feed` reports its states, how a route is qualified, how a
`Responsive` breakpoint is declared. This skill answers *what*: what the screen should say, in what
order, at what density, with what motion. Where a project states its own conventions, the project
wins over every number below.

## What this skill is not

It carries no API reference, for the same reason the review skill does not:

- **MVUX, navigation, Toolkit controls, themes** - the `uno-*` skills, where that plugin is loaded.
- **Everything else about the platform** - the `UnoDocs` MCP server: `uno_platform_docs_search` for
  breadth, then `uno_platform_docs_fetch` on the page a search names.

And it is not the gate. When the design is built, `uno-design-review` judges the result against the
ledger. Designing to that ledger is cheaper than repairing to it.

## 1. Design order

Structure before style, always. Style on top of broken structure is decoration on a mistake.

1. **The action.** What is the one thing a person comes here to do? A screen with no primary
   action is a dashboard of orphans. If the brief does not name one, ask; it cannot be guessed
   from the data.
2. **The sequence.** In what order does the user meet the content? The most-scanned screen in any
   application is the inbox, and it works because the order is the point.
3. **The states.** Loading, empty, error, success - designed as content, not as afterthoughts. A
   feed owns these states, so the design must say what each of them renders.
4. **The density.** How much of this fits, and how much is a second screen's problem.
5. **The motion.** What moves, when it moves, and what it tells the person.
6. **The style.** The last decision, not the first. It is the only one a user will name, and the
   only one that is cheap to change later.

## 2. Flows

- **Predictable back.** A back action returns to where the person was, on every head, with the
  scroll position they left. Where the platform owns back (system bar, browser), the layout keeps
  its content clear of it and does not reimplement it.
- **One structure.** The app has one navigation structure - a shell, a tab bar, a navigation view -
  and every screen lives inside it. A screen that builds its own navigation is a second app.
- **Depth is earned.** Every level below the shell must be worth the back-press it costs. A detail
  that fits the parent screen as an expansion does not get its own route.
- **No dead ends.** Every screen either completes its action, leads somewhere, or says why it
  cannot. A screen that does none of the three is a wall.
- **The destructive path is longer than the accidental one.** Deleting, leaving with unsaved work,
  and sending on a misfire all get a step that cannot be reached by thumb; undo, where it exists,
  is the better step.

## 3. Hierarchy

A screen has one primary action, one dominant heading, and one place the eye lands first.

- **Weight matches importance.** Size, weight, and contrast are spent on importance, not on
  decoration. If two elements compete, the screen has no primary action, and step 1 of section 1
  is where that is fixed.
- **Three levels, not five.** Primary, secondary, tertiary. A fourth level is the design saying
  everything at once. Where the theme's typography styles exist, they are the scale; where they do
  not, the project defines one, once, and the screens reference it.
- **Proximity groups before boxes do.** Related elements sit close; unrelated elements sit apart.
  A border is for a boundary that proximity cannot express.
- **The first screenful is the design.** What a person sees before scrolling is the screen. What
  is below it is the appendix. If the primary action is below the fold on a phone-width window,
  the hierarchy has not been done.
- **The secondary supports, never competes.** A secondary action sits near the primary, smaller,
  quieter, and never in the primary's visual register - two filled buttons side by side is two
  primary actions, and the screen has no answer to which to press.

## 4. Density and scale

Numbers below are starting points, not law. A project's scale, once it has one, outranks them.

- **Spacing.** 4, 8, 12, 16, 24, 32. Related things sit 4-8 apart, sibling blocks 16-24, sections
  32 or more. Where the scale does not carry a value the layout wants, the scale is wrong, not the
  layout.
- **Type.** The theme's typography style for the role; a hand-typed size is a screen drifting out
  of the scale one control at a time. Body text no smaller than the theme's minimum, because a
  raised system text scale will meet it.
- **Reading width.** Prose that runs the full width of a wide window is unreadable at its middle.
  Cap a reading measure and center what is left; data and rows do not need the cap, paragraphs do.
- **Breakpoints.** The Toolkit's defaults are narrowest 150, narrow 300, normal 600, wide 800,
  widest 1080. A layout is designed for each declared breakpoint in the file that declares it; a
  screen that holds at 300 and at 1080 is designed for both.
- **Touch.** 48dp on Android, 44pt on iOS, for anything a thumb reaches. The glyph may stay small;
  the hit region may not.
- **What a row carries.** A row that holds a title, a line of detail, and a timestamp is a row.
  A row that also carries a subtitle, a badge, a secondary timestamp, and two buttons is a card
  pretending to be a row, and the list has become a form.

## 5. State and feedback

- **Design the four states as content.** A feed reports loading, error, empty, and value; each is
  a different screen and each is rendered: the loading state names what is arriving, the empty
  state names what is missing and the action that fills it, the error state says what failed in
  the person's terms with the retry, the value state is the thing the screen was for.
- **Feedback arrives before 100ms.** A press is answered the moment it lands - a pressed visual on
  the control, not a result later. Where the work is longer than the eye's patience, the loading
  state is the continuation of the feedback, not a replacement for it.
- **Progress is proportionate.** A spinner for a moment, a skeleton for a shape that is known,
  progress for a task with a length. A task with a length that has no progress is a spinner the
  user is timing.
- **Errors sit where the work happened.** Field-level errors render beside the field, say what is
  wrong in the person's terms, and keep what was typed. A summary that scrolls the person away
  from the field is a second error.
- **Progressive disclosure.** Default to what is needed now: the summary on the row, the detail on
  demand, the advanced settings behind a step. A screen that shows everything at once is a screen
  that has decided nothing.

## 6. Motion

Motion is information: something arrived, something left, something happened here.

- **Durations.** 100-150ms for a press or a toggle, 200-300ms for a state change, 300-500ms for a
  screen transition. Feedback the person is waiting on never exceeds 300ms.
- **Easing.** Entrances ease out, exits ease in, and a thing that moves from A to B eases in-out.
  Linear is for a spinner, where the waiting is the point.
- **Transform and opacity.** What moves is a transform and an opacity. What re-lays out per frame
  is stutter, on every head and worst on the slowest one.
- **Nothing decorative.** Motion that does not tell the person something is animation, and
  animation on a work screen is noise. A screen transition that plays the same way for a person
  who asked the system to stop animating is not a feature.
- **The cut is a design decision.** Most screens want no transition at all. A cut is faster,
  cheaper, and honest; a transition is earned by a screen that is part of a sequence the person
  follows.

## 7. Style without a brand

Where the brief does not carry a brand, the defaults are the risk, because they read as
unconsidered:

- **The surface pair, not the palette.** One surface, one text pair from the theme's semantic
  brushes, and one accent for the primary action. A screen that reaches for three accents has
  three opinions.
- **Elevation is earned.** A raised surface means raised importance. A list where every row is a
  card is a list where nothing is raised, and it is also half as dense.
- **One treatment, applied.** Rounded corners, dividers, shadows: chosen once, applied everywhere,
  absent everywhere else. A mix is not an aesthetic, it is a lack of decision.
- **The test.** A screen that would pass for any application, with any brand, is a screen that has
  not been designed. The one thing a person will remember - the primary action, the shape of the
  data, the density - is the design. Everything else is wallpaper.

## 8. Handoff to the review

The design is done when it can be built, and it is built to be graded:

1. `uno-design-review` runs on the result before it ships. Its ledger is the specification's
   acceptance test; a finding there is a design decision that was made in the code instead of here.
2. Where the design deliberately breaks a rule the ledger holds, the report says so and names the
   reason, so the review records it as decided rather than finding it as a defect.
3. Where a check could not be run - no device, no theme switch, no empty dataset - the handoff
   says so. A design that cannot be verified in one of the four states is a design that has not
   met one of them.
