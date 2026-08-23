---
name: uno-design-review
description: Use before shipping an Uno Platform screen, page, or XAML layout. Reviews what was built for theming discipline, adaptive layout, list virtualization, state completeness, platform feel, accessibility, and placeholder content. Trigger on "review this screen", "is this page ready", "check this XAML", or when a screen is implemented and about to be committed. Also use when a screen looks generic, cramped, or unfinished and the reason is not obvious.
license: MIT
metadata:
  author: Krzysztof Kasprowicz
  copyright: Copyright © 2026 Krzysztof Kasprowicz
  repository: https://github.com/Krzysztof318/skills
---

# Uno Design Review

A screen that compiles is not a screen that is done. This skill is the last filter before an Uno
Platform screen ships: a ledger of the defects that survive a green build, each with the signature
that reveals it, the search that finds it, and the response that repairs it.

Findings are reported against the built screen, never against a guess about it. Every finding names
a file and a line.

## What this skill is not

It carries no API reference. Uno documents its own surface far better than a checklist could, and
that documentation is reachable from the session:

- **MVUX, navigation, Toolkit controls, themes** - the `uno-*` skills, where that plugin is loaded.
- **Everything else about the platform** - the `UnoDocs` MCP server: `uno_platform_docs_search` for
  breadth, then `uno_platform_docs_fetch` on the page a search names.

Read those to learn *how* a control works. Read this to decide whether what you built with it is
good enough to ship. Where a project states its own conventions, that project wins over every rule
below.

## 1. Enforcement model

Three levels. Apply the weakest one that prevents the defect.

- **HARD BAN** - broken, inaccessible, or dishonest. No brief makes it acceptable.
- **AVOID BY DEFAULT** - a generated default that reads as unconsidered. Allowed the moment the
  request, the content, or the platform gives it a real job.
- **CONTEXTUAL WARNING** - legitimate with a cost. Requires a stated reason and a verification.

### Application order

1. An explicit request, the project's existing conventions, and real product behaviour come first.
2. Accessibility and honesty bans are never traded away.
3. `AVOID BY DEFAULT` applies only where the request leaves the choice open. Never reject a
   requested layout solely because it appears in this ledger.
4. Clearly labelled mock data is allowed in an explicitly requested prototype. Never present it as
   real, live, connected, or production-ready.
5. Where real content is unavailable, omit the element or leave a neutral placeholder, and list it
   in the handoff report (section 5).

## 2. Rules

Each rule states the level, what it forbids, the signature that reveals it, how to find it, when it
does not apply, and what to do instead.

### Theming and tokens

#### THM-001 - Colour written into a page or a control
- **Level:** HARD BAN
- **Rule:** A colour value belongs to the theme, not to a screen.
- **Failure signature:** `Background="#1C1B1F"`, `Foreground="White"`, `Fill="Red"`, or a
  `SolidColorBrush` declared in `Page.Resources`.
- **Detection:** `grep -rnE '(Background|Foreground|Fill|Stroke|BorderBrush)="(#|[A-Z][a-z])' --include='*.xaml'`
- **Exception:** The theme's own palette or brush dictionary, which is where colour is defined.
- **Preferred response:** Use the semantic brush that names the role, through `{ThemeResource ...}`.
  A literal stops following the system theme the moment the user switches it.

#### THM-002 - Font size or family typed by hand
- **Level:** AVOID BY DEFAULT
- **Rule:** Reach for the theme's typography styles rather than a number.
- **Failure signature:** `FontSize="14"`, `FontWeight="SemiBold"`, `FontFamily="Segoe UI"` scattered
  across pages, each screen inventing its own scale.
- **Detection:** `grep -rn 'FontSize=\|FontFamily=' --include='*.xaml'`
- **Exception:** A deliberate display treatment that the type scale genuinely does not carry, stated
  once in a style rather than repeated inline.
- **Preferred response:** Apply the typography style key for the role. A hand-typed size is how a
  screen drifts out of the type scale one control at a time.

#### THM-003 - A copied `ControlTemplate` where lightweight styling would do
- **Level:** CONTEXTUAL WARNING
- **Rule:** Override the resource keys a control reads before replacing the control's template.
- **Failure signature:** A full `ControlTemplate` in a page or a style dictionary, differing from the
  theme's by a colour and a corner radius.
- **Detection:** `grep -rn '<ControlTemplate' --include='*.xaml'`
- **Exception:** The interaction itself genuinely differs from the stock control.
- **Preferred response:** Lightweight styling. A copied template stops receiving the design system's
  fixes, and it is the reason a control looks correct today and wrong two releases later.

#### THM-004 - Verified in one theme only
- **Level:** HARD BAN
- **Rule:** A screen is reviewed in light and in dark before it ships.
- **Failure signature:** Text that vanishes, a border that disappears, or an icon that turns
  invisible after a theme switch. Almost always a consequence of THM-001.
- **Detection:** Screenshot the screen in both themes. Compare, do not assume.
- **Exception:** None. An application that follows the system theme is an application that is used
  in both.
- **Preferred response:** Fix the brush, then re-shoot both.

### Layout and adaptivity

#### LAY-001 - Magic-number spacing
- **Level:** AVOID BY DEFAULT
- **Rule:** Spacing comes from a scale, not from whatever number closed the gap.
- **Failure signature:** `Margin="10"`, `Margin="13,7,0,22"`, `Padding="15"` - values that appear
  once each and share no rhythm.
- **Detection:** `grep -rn 'Margin="\|Padding="' --include='*.xaml' | grep -v StaticResource`
- **Exception:** An optical correction that a scale genuinely cannot express, commented as such.
- **Preferred response:** Define a spacing scale once as resources and reference it. Unrelated
  numbers are what makes a screen read as assembled rather than composed.

#### LAY-002 - Layout branched on the running platform
- **Level:** HARD BAN
- **Rule:** A layout adapts to the space it is given, never to the operating system it runs on.
- **Failure signature:** `#if __ANDROID__` around a layout, or a check on the OS that changes a
  column width. It is wrong on a resized desktop window and wrong on a tablet.
- **Detection:** `grep -rn '#if __\|OperatingSystem.Is' --include='*.cs' --include='*.xaml.cs'`
- **Exception:** A genuinely platform-owned capability, which is not layout.
- **Preferred response:** `VisualStateManager` with adaptive triggers, or the Toolkit's responsive
  markup. Width is the input; platform is not.

#### LAY-003 - Fixed size on an element that carries text
- **Level:** HARD BAN
- **Rule:** Text-bearing containers size to their content.
- **Failure signature:** `Height="48"` on a button whose label is bound, `Width="200"` on a card
  holding a title. It clips the moment the system text scale is raised or the string is translated.
- **Detection:** `grep -rnE '(Width|Height)="[0-9]+"' --include='*.xaml'`, then check whether the
  element or a descendant carries text.
- **Exception:** Icons, avatars, and images at a deliberate size.
- **Preferred response:** `MinHeight` and `MinWidth` where a floor is needed, and let the content
  drive the rest. Verify at a raised system text scale, not only at 100%.

#### LAY-004 - `StackPanel` where a `Grid` is required
- **Level:** HARD BAN
- **Rule:** A `StackPanel` gives its children unbounded space along its orientation, so nothing
  inside it wraps, truncates, or shares the remaining width.
- **Failure signature:** A `TextBlock` with `TextWrapping="Wrap"` inside a horizontal `StackPanel`
  that never wraps; a long subject line pushing a timestamp off the screen edge.
- **Detection:** `grep -rn 'Orientation="Horizontal"' --include='*.xaml'`, then look for text inside.
- **Exception:** A short, bounded row of fixed-size elements.
- **Preferred response:** A `Grid` with a `*` column for the text and `Auto` for what follows it.
  This is the single most common reason an Uno screen looks fine in the designer and breaks on a
  narrow window.

#### LAY-005 - Edge-bound content without safe-area handling
- **Level:** HARD BAN
- **Rule:** Content stays clear of a notch, a rounded corner, a system bar, and a soft keyboard.
- **Failure signature:** A hard-coded top margin standing in for the status bar, or a bottom command
  bar sitting under the gesture indicator.
- **Detection:** `grep -rn 'SafeArea' --include='*.xaml'` and compare against the screens that reach
  a screen edge.
- **Exception:** A screen whose content never touches an edge.
- **Preferred response:** `SafeArea.Insets` from the Toolkit. A hard-coded margin is correct on
  exactly one device.

#### LAY-006 - One shape for every width
- **Level:** CONTEXTUAL WARNING
- **Rule:** A layout that holds from a phone to a wide desktop window is stated, not hoped for.
- **Failure signature:** A three-column grid that becomes three slivers at 400px, or a phone layout
  stretched to 1600px with a column of text running the full width.
- **Detection:** Screenshot at a narrow, a medium, and a wide width.
- **Exception:** A view that is genuinely single-width by design.
- **Preferred response:** Declare the collapse for every multi-column layout in the same file that
  declares the layout, and cap reading measures rather than letting a paragraph span a monitor.

### Collections and density

#### LST-001 - An unvirtualised collection
- **Level:** HARD BAN
- **Rule:** A list whose length is data-driven virtualises.
- **Failure signature:** `ItemsControl` bound to a large collection; or a `ListView` inside a
  `ScrollViewer` or a vertical `StackPanel`, which hands it unbounded height and silently realises
  every item. On a browser head this is the difference between a screen and a frozen tab.
- **Detection:** `grep -rn '<ItemsControl' --include='*.xaml'`, and for each `ListView` or
  `ItemsRepeater`, walk its ancestors for a `ScrollViewer` or an unconstrained `StackPanel`.
- **Exception:** A collection with a small, bounded, known length.
- **Preferred response:** `ListView` or `ItemsRepeater` given a constrained height by a `Grid` row,
  with its own scrolling. Never nest a scrolling list inside another scrolling container.

#### LST-002 - An item template deeper than the row needs
- **Level:** CONTEXTUAL WARNING
- **Rule:** Every element in a template is paid for once per realised row.
- **Failure signature:** Four nested panels, two borders, and a converter chain to render a line of
  text and a timestamp.
- **Detection:** Read the `DataTemplate`. Count the elements, multiply by the visible row count.
- **Exception:** A genuinely rich row that the product needs.
- **Preferred response:** Flatten to a single `Grid`. Row cost is the one performance number in a
  list that a user feels directly.

#### LST-003 - Every row rendered as a card
- **Level:** AVOID BY DEFAULT
- **Rule:** Elevation communicates hierarchy. A list where every row is elevated communicates none.
- **Failure signature:** A scrolling column of shadowed, rounded, margined cards, each holding one
  line of text. It halves the number of rows on screen and says nothing.
- **Detection:** Look for a card or a `Border` with a shadow inside a `DataTemplate`.
- **Exception:** A genuinely card-shaped collection - a gallery, a board, a small set of tiles.
- **Preferred response:** Plain rows separated by space or a divider, with the card reserved for
  what is actually raised above the rest.

#### LST-004 - A divider on every row
- **Level:** AVOID BY DEFAULT
- **Rule:** A separator marks a boundary that matters.
- **Failure signature:** A one-pixel line under all forty rows, drawing a ledger nobody asked for.
- **Detection:** Look for a bottom border or a divider inside a `DataTemplate`.
- **Exception:** Dense tabular data where the line genuinely aids tracking across columns.
- **Preferred response:** Group with space, and use the divider between groups.

### State completeness

#### STA-001 - Only the success state was built
- **Level:** HARD BAN
- **Rule:** Anything asynchronous has a loading, an empty, an error, and a success state.
- **Failure signature:** A screen that renders data and nothing else. It is blank while loading,
  blank when the collection is empty, and blank when the request failed - three different meanings
  rendered identically.
- **Detection:** For every async binding, ask which element renders each of the four states.
- **Exception:** None where the source can be slow, empty, or fail. That is every remote source.
- **Preferred response:** Build all four. Where the framework offers a view that switches on feed
  state, use it rather than hand-rolling visibility bindings.

#### STA-002 - An empty state that is an empty screen
- **Level:** AVOID BY DEFAULT
- **Rule:** An empty state says what would be here and how to put it here.
- **Failure signature:** A blank pane, or the word "No items".
- **Detection:** Run the screen with an empty collection and look at it.
- **Exception:** An empty region so small that a message would be noise.
- **Preferred response:** Name what is missing and give the action that fills it. The empty state is
  the first thing a new user sees, and it is usually the least designed.

#### STA-003 - A spinner that has no end
- **Level:** HARD BAN
- **Rule:** Every load either resolves, fails visibly, or times out visibly.
- **Failure signature:** An exception swallowed in a handler, leaving a progress ring turning
  forever. The user cannot tell a slow network from a dead screen.
- **Detection:** `grep -rn 'catch' --include='*.cs'` and check each for a state transition.
- **Exception:** None.
- **Preferred response:** Move to the error state, say what failed in the user's terms, and offer
  the retry.

#### STA-004 - Simulated success
- **Level:** HARD BAN
- **Rule:** A control reports what happened, never what would look good.
- **Failure signature:** A delay followed by a success message; a "Saved" toast on a screen wired to
  nothing; a hardcoded "Connected" label.
- **Detection:** `grep -rn 'Task.Delay' --include='*.cs'` and check what follows each.
- **Exception:** An explicitly requested prototype, labelled as one, and listed in the handoff
  report.
- **Preferred response:** Wire it, or leave the control visibly unwired. A fake success is a lie
  told to the person who has to trust the screen.

#### STA-005 - A control that looks disabled but is not
- **Level:** HARD BAN
- **Rule:** Unavailability is expressed with the state the platform owns.
- **Failure signature:** `Opacity="0.5"` on a button that still raises its click.
- **Detection:** `grep -rn 'Opacity="0' --include='*.xaml'`
- **Exception:** None.
- **Preferred response:** `IsEnabled`. It carries the visual, the hit-testing, and the accessibility
  announcement together; opacity carries the first and lies about the other two.

### Interaction and platform feel

#### INT-001 - No feedback on press
- **Level:** HARD BAN
- **Rule:** A press gives feedback where the platform gives feedback.
- **Failure signature:** A custom-templated tile that reacts only after its work completes, so a
  slow action reads as an ignored tap.
- **Detection:** Press every interactive element and watch. Custom templates are where this is lost.
- **Exception:** None.
- **Preferred response:** Keep the stock control's visual states, or reproduce pressed and hover in
  the template that replaced them.

#### INT-002 - A touch target below the platform minimum
- **Level:** HARD BAN
- **Rule:** 48x48dp on Android and Material, 44x44pt on iOS, for anything a finger reaches.
- **Failure signature:** A 16px icon button with no padding, comfortable with a mouse and unusable
  with a thumb.
- **Detection:** Measure the interactive bounds, not the glyph. The visual may stay small.
- **Exception:** A control genuinely unreachable by touch on every head that ships.
- **Preferred response:** Pad the target to the minimum and leave the glyph its size.

#### INT-003 - The drawing thread blocked
- **Level:** HARD BAN
- **Rule:** Nothing blocks the thread that draws.
- **Failure signature:** `.Result`, `.Wait()`, or `Thread.Sleep` in a handler or a constructor. A
  frozen window on desktop, a frozen tab in the browser.
- **Detection:** `grep -rnE '\.Result\b|\.Wait\(\)|Thread\.Sleep' --include='*.cs'`
- **Exception:** None on any path a UI thread reaches.
- **Preferred response:** `await`, all the way up.

#### INT-004 - Animating a layout property
- **Level:** CONTEXTUAL WARNING
- **Rule:** Animate transform and opacity. Animating `Width`, `Height`, or `Margin` runs a layout
  pass per frame and needs dependent animations enabled to run at all.
- **Failure signature:** `EnableDependentAnimation="True"` on a storyboard, or a hand-driven margin
  animation that stutters on a browser head.
- **Detection:** `grep -rn 'EnableDependentAnimation' --include='*.xaml'`, and read the target
  property of every storyboard beside it.
- **Exception:** A short, small, one-off transition where measurement shows it is free.
- **Preferred response:** `RenderTransform` or composition. Verify on the slowest head, which is
  usually WebAssembly.

#### INT-005 - One platform dressed as another
- **Level:** AVOID BY DEFAULT
- **Rule:** A control the platform draws is left drawing itself.
- **Failure signature:** An iOS-shaped switch templated onto Android, or a desktop context menu
  reproduced on touch.
- **Detection:** Compare a screenshot per head against that platform's own conventions.
- **Exception:** A deliberate, brand-owned control language applied consistently everywhere.
- **Preferred response:** Reach for a template when the interaction genuinely differs, not to make
  one platform resemble another.

### Accessibility

#### A11Y-001 - Contrast below WCAG AA
- **Level:** HARD BAN
- **Rule:** 4.5:1 for body text, 3:1 for large text and for the visual boundary of a control.
- **Failure signature:** Grey helper text on a tinted surface; a ghost button whose only edge is a
  faint stroke; placeholder text lighter than the label above it.
- **Detection:** Measure foreground against the surface actually behind it, in both themes.
- **Exception:** Text that is purely decorative and duplicated in an accessible form.
- **Preferred response:** Raise the foreground or change the surface. Where the theme's semantic
  pairing is used as intended, this rule rarely fires - it fires on hand-picked colours.

#### A11Y-002 - An icon-only control with no accessible name
- **Level:** HARD BAN
- **Rule:** Every control a person can reach announces what it does.
- **Failure signature:** A row of glyph buttons, each announced as "button".
- **Detection:** `grep -rn 'AutomationProperties' --include='*.xaml'` and compare against the
  icon-only controls. A tooltip is not a substitute.
- **Exception:** A control with visible text that already names it.
- **Preferred response:** `AutomationProperties.Name`, and a tooltip as well for the sighted user
  who does not recognise the glyph.

#### A11Y-003 - Focus not visible or not reachable
- **Level:** HARD BAN
- **Rule:** Every interactive element is reachable by keyboard, in a sensible order, with a visible
  focus indicator.
- **Failure signature:** A custom template that dropped the focus visual; a tab order that jumps
  across the screen; a dialog that does not take focus when it opens.
- **Detection:** Drive the screen with Tab and Enter only. Watch where focus goes and whether it can
  be seen.
- **Exception:** None on desktop or browser heads.
- **Preferred response:** Restore the focus visual in the template, set `TabIndex` where document
  order is wrong, and move focus into a dialog when it opens.

#### A11Y-004 - Motion that ignores the system setting
- **Level:** CONTEXTUAL WARNING
- **Rule:** Where a platform exposes a reduced-motion preference, honour it.
- **Failure signature:** A page transition that plays identically for a user who asked the system to
  stop animating.
- **Detection:** Check whether the preference is read at all, and confirm the API is implemented on
  the heads that ship before relying on it.
- **Exception:** Motion that carries meaning no static state can carry, reduced rather than removed.
- **Preferred response:** Read the preference once, expose it as state, and let transitions collapse
  to a cut.

### Content

#### TXT-001 - Placeholder copy shipped
- **Level:** HARD BAN
- **Rule:** No lorem, no `TODO`, no framework starter text, no `Jane Doe`.
- **Detection:** `grep -rniE 'lorem|jane doe|john doe|TODO|FIXME|example\.com' --include='*.xaml' --include='*.cs'`
- **Exception:** An explicitly requested prototype, listed in the handoff report.
- **Preferred response:** Real copy, or omit the element.

#### TXT-002 - A placeholder used as a label
- **Level:** HARD BAN
- **Rule:** A field keeps its name after the user types in it.
- **Failure signature:** `PlaceholderText="Email"` with no header. The label vanishes on first
  keystroke, and it was never announced to a screen reader as a label.
- **Detection:** `grep -rn 'PlaceholderText=' --include='*.xaml'` and check each for a header.
- **Exception:** A single-field search box whose purpose is unmistakable.
- **Preferred response:** A header above the field. Keep the placeholder for an example value, or
  drop it.

#### TXT-003 - A literal string where the project localises
- **Level:** CONTEXTUAL WARNING
- **Rule:** In a project with resource files, user-visible text comes from them.
- **Detection:** `grep -rn 'Text="[A-Z]' --include='*.xaml'` and compare against the resource files.
- **Exception:** A project that does not localise.
- **Preferred response:** Move the string to resources. Also verify the layout survives a string
  30% longer, which is what LAY-003 and LAY-004 fail on.

## 3. Automatic checks

Automate only what a search can decide. No grep knows whether a colour suits a brand.

```bash
# Colour, type, and template literals            THM-001 THM-002 THM-003
grep -rnE '(Background|Foreground|Fill|Stroke|BorderBrush)="(#|[A-Z][a-z])' --include='*.xaml' .
grep -rn 'FontSize=\|FontFamily=' --include='*.xaml' .
grep -rn '<ControlTemplate' --include='*.xaml' .

# Layout                                          LAY-001 LAY-002 LAY-003 LAY-005
grep -rn 'Margin="\|Padding="' --include='*.xaml' . | grep -v StaticResource
grep -rn '#if __\|OperatingSystem.Is' --include='*.cs' .
grep -rnE '(Width|Height)="[0-9]+"' --include='*.xaml' .
grep -rn 'SafeArea' --include='*.xaml' .

# Collections                                     LST-001
grep -rn '<ItemsControl' --include='*.xaml' .

# State and threading                             STA-004 STA-005 INT-003
grep -rn 'Task.Delay' --include='*.cs' .
grep -rn 'Opacity="0' --include='*.xaml' .
grep -rnE '\.Result\b|\.Wait\(\)|Thread\.Sleep' --include='*.cs' .

# Accessibility and content                       A11Y-002 TXT-001 TXT-002
grep -rn 'AutomationProperties' --include='*.xaml' .
grep -rniE 'lorem|jane doe|john doe|TODO|FIXME|example\.com' --include='*.xaml' --include='*.cs' .
grep -rn 'PlaceholderText=' --include='*.xaml' .
```

A hit is a question, never a verdict. Open the file and decide.

## 4. Visual checks

The rules a search cannot reach are settled by looking. Where the Uno App MCP is available, this is
evidence rather than assertion:

1. `uno_health` - confirm which workspace the server resolved, before anything else.
2. `uno_app_start`, then `uno_app_get_screenshot` at a narrow, a medium, and a wide width - LAY-006.
3. Screenshot again in the other theme - THM-004, A11Y-001.
4. `uno_app_visualtree_snapshot` - element depth per row for LST-002, and the realised item count
   for LST-001.
5. `uno_app_key_press` with Tab only, following focus across the screen - A11Y-003.
6. Drive the screen into its empty, loading, and error states and shoot each - STA-001, STA-002.

Where the MCP is not available, say so and review from the source, and mark the visual findings as
unverified rather than passing them silently.

## 5. Report

Report findings most severe first, each as: rule ID, file and line, what is wrong, what to do.
Confirm each in the file before reporting it - a grep hit is a hypothesis.

Close with **Still needed**: every placeholder left in place, every unwired interaction, every
missing asset, and every check that could not be run. A review that lists nothing outstanding is
claiming the screen is finished.
