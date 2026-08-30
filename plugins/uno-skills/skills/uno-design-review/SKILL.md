---
name: uno-design-review
description: Use before shipping an Uno Platform screen, page, or XAML layout. Reviews what was built for hierarchy, theming discipline, adaptive layout, list virtualization, state completeness, form usability, navigation integrity, platform feel, visual consistency, accessibility, and placeholder content. Trigger on "review this screen", "is this page ready", "check this XAML", or when a screen is implemented and about to be committed. Also use when a screen looks generic, cramped, or unfinished and the reason is not obvious.
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

It is the counterpart of `uno-ux-design`, which makes the design decisions before the XAML exists.
A finding in this ledger is usually a design decision that was made in the code instead of there.

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

### Hierarchy

#### HIE-001 - Two competing primary actions
- **Level:** AVOID BY DEFAULT
- **Rule:** A screen has one primary action. A second element in the primary's visual register is
  a claim the screen cannot keep.
- **Failure signature:** Two filled, accent-background buttons side by side - Save and Delete,
  Buy and Compare - or three headings at the same size and weight in one viewport.
- **Detection:** Count the elements per page that carry the theme's primary style or the largest
  typography role. More than one is the finding.
- **Exception:** Two genuinely equal choices the product names as equal - Approve and Reject in
  an approval flow - stated as such in the design.
- **Preferred response:** One primary. Everything else secondary, quiet, or icon. The primary is
  where the eye lands first; when two elements both demand that landing, the screen has not said
  what it is for.

#### HIE-002 - A flat hierarchy
- **Level:** AVOID BY DEFAULT
- **Rule:** The eye needs a landing point. A screen where every element carries the same weight
  gives it none.
- **Failure signature:** One font size and one weight across a whole page; no element larger or
  stronger than any other; a card list in which title, detail, and timestamp all read as
  first-class.
- **Detection:** Count the distinct typography styles a page uses. Fewer than three across a
  screen that carries a title, body, and metadata is the signature.
- **Exception:** A screen that is a single role - a full-bleed list of same-shape rows.
- **Preferred response:** Primary, secondary, tertiary. Spend size, weight, and contrast on
  importance, and spend them only once.

#### HIE-003 - The primary action below the fold
- **Level:** CONTEXTUAL WARNING
- **Rule:** The first screenful is the screen. On a narrow width, the primary action is visible
  without scrolling.
- **Failure signature:** A phone-width screenshot in which the only filled button sits under
  three paragraphs of copy.
- **Detection:** Screenshot at a narrow width and look for the primary action before the scroll
  edge.
- **Exception:** A content screen whose action is the point of the reading - the comment box at
  the end of the article - stated as a deliberate design.
- **Preferred response:** Move the action up, into a command bar, or into the header. A warning
  with a cost: on desktop the fold is far, so state the width at which this was judged.

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

### Forms and input

#### FRM-001 - An error rendered away from its field
- **Level:** HARD BAN
- **Rule:** A field-level error renders beside the field, in the person's terms, and focus goes
  to it.
- **Failure signature:** A form validation fails and the only sign of it is a toast, a dialog, or
  a red banner at the top of the screen, while the invalid field sits unmarked and focus stays
  where it was. The person is asked to find which field failed.
- **Detection:** Read the validation path. For each field, ask where its error renders, what it
  says, and where focus lands on failure.
- **Exception:** A screen-level failure that belongs to no single field - the account is locked -
  which is a screen error, not a field error.
- **Preferred response:** Error text under or beside the field, a state the field visibly carries,
  and focus moved to the first invalid field. A summary is allowed as an addition, never as the
  whole of it.

#### FRM-002 - Input destroyed by a failed save
- **Level:** HARD BAN
- **Rule:** A failed write keeps what the person typed.
- **Failure signature:** A save fails on the server and the form reloads, resets, or clears the
  failed field - and the person types it again, not knowing which attempt failed.
- **Detection:** Read the error path of every write. Ask what happens to each field's value when
  the request returns an error.
- **Exception:** None. The write failed; the person's work is the only thing that must not.
- **Preferred response:** Keep the values, show the error beside the field or the form, and leave
  the retry where the person already is.

#### FRM-003 - A field that validates too late
- **Level:** CONTEXTUAL WARNING
- **Rule:** A person is told what is wrong when they can still act on it, not after they have
  committed.
- **Failure signature:** Nothing happens while the person types, and the whole form turns red on
  submit; or a field is flagged empty on blur while the person is still in the middle of the form
  and did not ask.
- **Detection:** Fill a form deliberately wrong and note when the first sign of it appears, and
  how many times the person is told.
- **Exception:** A field whose validity only exists once the form is complete.
- **Preferred response:** Validate the field when it leaves the person's hand or on submit, show
  it there once, and do not repeat it louder. The first sign of an error is a correction, not an
  indictment.

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

### Navigation and flow

#### NAV-001 - A screen that builds its own navigation
- **Level:** HARD BAN
- **Rule:** The application has one navigation structure, and every screen lives inside it.
- **Failure signature:** A page that draws its own back button beside the shell's, its own tab
  bar, or its own header where the shell already carries one. Two backs on a screen are two
  applications arguing.
- **Detection:** `grep -rn 'NavigationView\|TabBar\|NavigationBar' --include='*.xaml'` and compare
  against the screens that sit inside the shell. A control at the leaf is the finding.
- **Exception:** A screen that is itself a navigation surface - a settings root that owns its own
  sub-navigation - stated as such.
- **Preferred response:** The shell carries the structure; the screen carries the content and its
  actions in the command bar or header the shell leaves for them.

#### NAV-002 - A back that does not go back
- **Level:** HARD BAN
- **Rule:** Back returns to where the person was, on every head, with the position they left.
- **Failure signature:** A hand-rolled back button that pops two pages, that resets a list to the
  top, that re-runs a query the person has already filtered, or that is missing on a head where
  the platform does not provide one.
- **Detection:** `grep -rn 'GoBack\|Pop(' --include='*.cs'` and walk each call's argument. Then
  drive the screen: filter, scroll, push, back - and ask whether the filter and the scroll
  survived.
- **Exception:** None for the system back. Where a head provides no back at all, the screen
  provides one; that is the rule's other half.
- **Preferred response:** Let the navigation stack own back, and keep state in the feed so a
  return restores it. A back button that re-runs the world is not a back button.

#### NAV-003 - A dead end
- **Level:** AVOID BY DEFAULT
- **Rule:** A screen either completes its action, leads somewhere, or says why it cannot go on.
- **Failure signature:** A detail view with no action and no path forward; an error screen with
  no retry; an empty state with no verb. The person arrives and the screen is finished with them.
- **Detection:** For every leaf screen, name its exit: the action it completes, the route it
  offers, or the reason it states. No name is the finding.
- **Exception:** A terminal screen the product means to be terminal - the receipt after a
  purchase, stated as such.
- **Preferred response:** Give the screen one honest exit. A retry on the error, a verb on the
  empty state, an action on the detail.

#### NAV-004 - A destructive path as short as the accidental one
- **Level:** HARD BAN
- **Rule:** What cannot be undone gets a step that a thumb cannot reach by accident.
- **Failure signature:** A Delete in a row's context menu that fires on one tap, a "Clear all"
  beside the items it clears, a send that a stray thumb can land on.
- **Detection:** Look for every irreversible action and count the taps between the first touch
  and the point of no return. One is the finding, unless it is guarded by an undo that actually
  undoes.
- **Exception:** An action with a working undo, where the undo is the guard.
- **Preferred response:** Confirmation for the truly irreversible, a confirm that names what will
  be lost; undo for the rest. The length of the path is the price of the mistake.

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

### Visual consistency

#### VIS-001 - A third variation
- **Level:** AVOID BY DEFAULT
- **Rule:** The same thing looks and behaves the same everywhere. A second way of doing a
  recurring thing is a finding; a third way is a broken system.
- **Failure signature:** One confirm is a `ContentDialog`, the next is an inline banner, the next
  is a popup of a different shape; one screen's empty state is an icon and a verb, the next is a
  bare "No items".
- **Detection:** `grep -rn 'ContentDialog' --include='*.xaml' --include='*.cs'`, and for each
  recurring job - confirm, toast, empty state, loading - list the variations the app actually
  carries. Two or more is the finding.
- **Exception:** A variation the content genuinely forces - a confirm that must show a list,
  where the small confirm cannot.
- **Preferred response:** One pattern per job, in a shared control or style the screens reference.
  Flag the inconsistency; do not invent the third variation to fix it.

#### VIS-002 - A treatment applied to some and not all
- **Level:** AVOID BY DEFAULT
- **Rule:** A visual decision made once is a decision made everywhere, or not at all.
- **Failure signature:** Corner radius on the cards in one screen and square ones in the next;
  dividers between rows here and none there; shadows on one surface and not its neighbours.
- **Detection:** For each visual treatment in the app - radius, divider, shadow, uppercase
  label - name the screens that carry it and the ones that do not, and ask whether the difference
  is a decision or an accident.
- **Exception:** A difference that tracks a real difference - a group header is a header.
- **Preferred response:** The treatment, applied or removed. A mixed treatment is not an
  aesthetic; it is a decision that was not finished.

#### VIS-003 - An icon from two sets
- **Level:** AVOID BY DEFAULT
- **Rule:** A screen carries one icon language: one set, one weight, one size.
- **Failure signature:** A sharp filled glyph beside a thin outlined one; a 16px icon beside a
  24px one doing the same job; an emoji where the set has a glyph.
- **Detection:** Collect the icons a screen uses - asset names and sizes - and compare their
  weight and size. Two families, or one job at two sizes, is the finding.
- **Exception:** None. A brand's own icon set is one set, which is the rule's point.
- **Preferred response:** One set for the app, sized by role. Where an icon is missing from the
  set, it is missing; a borrowed glyph is what reads as unfinished.

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

# Hierarchy and forms                             HIE-001 FRM-002
grep -rn 'FilledButtonStyle\|ContainedButtonStyle\|AccentButtonStyle' --include='*.xaml' .
grep -rn 'MessageBox\|ContentDialog' --include='*.cs' .

# Navigation                                      NAV-001 NAV-002
grep -rn 'NavigationView\|TabBar\|NavigationBar' --include='*.xaml' .
grep -rn 'GoBack\|Pop(' --include='*.cs' .

# Consistency                                     VIS-001
grep -rn 'ContentDialog' --include='*.xaml' --include='*.cs' .
```

A hit is a question, never a verdict. Open the file and decide.

## 4. Visual checks

The rules a search cannot reach are settled by looking. Where the Uno App MCP is available, this is
evidence rather than assertion:

1. `uno_health` - confirm which workspace the server resolved, before anything else.
2. `uno_app_start`, then `uno_app_get_screenshot` at a narrow, a medium, and a wide width - LAY-006.
   On the narrow shot, name where the eye lands first and where the primary action sits - HIE-001,
   HIE-002, HIE-003.
3. Screenshot again in the other theme - THM-004, A11Y-001.
4. `uno_app_visualtree_snapshot` - element depth per row for LST-002, and the realised item count
   for LST-001.
5. `uno_app_key_press` with Tab only, following focus across the screen - A11Y-003.
6. Drive the screen into its empty, loading, and error states and shoot each - STA-001, STA-002.
7. Fill a form deliberately wrong, submit, and watch where the first error lands and where focus
   goes - FRM-001, FRM-003. Repeat the save with the network cut and watch what happens to the
   typed values - FRM-002.
8. Push a screen, filter or scroll it, and press back on each head: did the position and the
   filter survive, and was there one back, not two - NAV-001, NAV-002.
9. Walk three screens that carry the same job - a confirm, a toast, an empty state - and name
   their variations - VIS-001, VIS-002, VIS-003.

Where the MCP is not available, say so and review from the source, and mark the visual findings as
unverified rather than passing them silently.

## 5. Report

Report findings most severe first, each as: rule ID, file and line, what is wrong, what to do.
Confirm each in the file before reporting it - a grep hit is a hypothesis.

Close with **Still needed**: every placeholder left in place, every unwired interaction, every
missing asset, and every check that could not be run. A review that lists nothing outstanding is
claiming the screen is finished.
