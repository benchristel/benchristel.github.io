---
layout: post
title: Design's Wrong Turn
---

## What am I criticizing, exactly?

Examples:
- scroll bar that isn't
- GitHub vs Phabricator
- Big Sur
- Wasted screen real estate
- Indistinct icons
- functional distinction without visual distinction
  - buttons are flat. buttons look the same as inputs
  - everything has rounded corners with the same radius
    (or else everything has square corners, as in Windows)
  - everything is the same color
  - clickable areas are not demarcated—you have to guess
    where they are.
    - sometimes you guess wrong: e.g. Jira input
    - fitt's law?
- "Focus on content"
  e.g. big sur "New design for apps makes it easier to focus on your content" https://apps.apple.com/us/app/macos-big-sur/id1526878132?mt=12 "a new design for apps makes it easier for users to stay focused on their content and interact with apps. Buttons and controls appear when needed and recede when they are not — reducing visual complexity and bringing the most relevant content to the forefront."
- reliance on optical illusions
  e.g. github diffs

## Why might these developments in design be good?

- less visual clutter
  - e.g. optical illusions, e.g. github diff gutter
  - direction of attention
    - attention is a scarce resource
- simpler interface for novice/casual users
- touch-friendliness
- accessibility
  - maybe flat design is needed to support dark mode!
    - you can't have shadows on a black background!
      - you _can_ go to almost black.
- componentization
  - these changes in design trends coincided with an
    increase in the development and use of component
    libraries and explicit design languages for the Web—e.g.
    Google's Material Design guidelines. These libraries and
    languages help designers and developers work faster and
    (ostensibly) produce better output. Maybe the changes in
    design style are necessary for these languages and
    libraries to exist.

## Why are they actually bad?

- optical illusions
  - fail when necessary context is obscured
  - sometimes exact boundaries are important!
    - code diffs: indentation level matters—don't hide the margin.
  - fitt's law—how long does it take to acquire a target
    when you don't know how big the target is?
- less visual clutter
  - Tog: "software does not have to look like a tractor. It can look like a Porsche. It can't look like a Porsche that's missing its steering wheel, driver's seat, and brake pedal."
  - cues should be apparent if you direct attention to them
  - apps now have _less_ of a distinction between chrome and
    content
  - the problem with the "let users focus on the content"
    guideline is that it can be applied recursively
- simpler interface for novices
  - you can't learn what you can't perceive
  - you _can_ be frustrated by what you don't understand
  - creates learned helplessness
    - analogy: Rails and HTTP
- accessibility
  - using grayscale icons ensures that icons will work for
    color-blind users.
  - but of course colorful icons with sufficient contrast in
    other dimensions will work for color-blind users too.
    Flat gray icons are just a way of saving _developers'_
    and designers' time because they don't have to remember
    to switch their displays to grayscale mode to test
    accessibility.
  - lack of color contrast makes the UI _less_ accessible for people with
    other visual impairments (e.g. nearsightedness).
  - while we're on the subject, let's talk about the appalling
    lack of contrast in many apps.
- the mirror of the self test

## What real problems in design need addressing?

- deep hierarchy
  - desktops, apps, windows, tabs, tab stacks (?), tabs within a site?
    - it's too much to keep track of
    - everyone has their own favorite way of organizing things
- peak attention
  - c.f. "peak oil"
  - it's possible that the amount of attention software can
    extract from its users will soon peak or has already
    peaked.
- selling software
  - Tog proposes that software should have a showroom mode
- lifestyle-shaming
  - Don't sell to users by lifestyle-shaming them.
- tiny screens vs. fat fingers
  - phones can't get much bigger. Our fingers aren't getting
    any smaller. There's only so much we can cram onto a
    screen.

## What opportunities do we have?

- high-res screens with great color depth and contrast
  - icons could be beautiful!
- beyond dark mode
  - dark mode isn't actually sufficient. For late-night
    reading, a black-text-on-white page that's color-shifted
    toward the red end of the spectrum is more legible and
    reduces eyestrain. Dark mode doesn't affect the colors
    of videos and images, which can mess with circadian
    rhythm.
  - dark mode doesn't mean "invert all color values"
    - content can still be lighter than context, while
      leaving plenty of headroom for contrast with the text.
- customizable themes
  - more and more UIs are using CSS (e.g. Firefox, and even
    Windows!) CSS makes it easy to apply whatever theme you
    want to a site. GitHub has several modes you can choose
    from (light/dark/high-contrast), which they make
    technically feasible by using CSS variables. The barrier
    to enabling custom color schemes on GitHub would be
    pretty low.

## What are the risks of change?

- designs with too much personality, whimsy, or flimflam are
  usually unwelcome. Users don't care about the product,
  brand, or the designer; they want to accomplish a goal and
  get on with their lives. Apple's iPod is a great example of
  a design whose brand strength was _structural_. Rather than
  design an iPod that constantly reminded you via gratuitous
  glitz how cool it was, Apple "just" made a revolutionarily
  useful and usable product that made _people_ feel cool.

## What are some examples of great design?

- 2nd gen iPod Nano
- Notational Velocity
- Phabricator
  - Separates content from context/controls with distinct
    background colors: content gets a white background (and
    therefore the highest contrast) and context/controls
    (like line numbers) get a dimmer background.
    - Github does not do this. Everything is white.
  - shows more context lines by default, conserves space
    by using a smaller font size overall. Phabricator is
    almost exactly the same shape as GitHub at 90% zoom.
  - has a table of contents sidebar for each diff
    - which is resizable (!)
    - and includes a keyboard shortcut reference (?!)
      - WHY DOESN'T GITHUB DO THIS? GitHub's most powerful
        features are completely invisible and nearly
        undiscoverable.
  - does Phabricator's UI appear more cluttered or complex
    than GitHub's? Certainly. But that's because it is
    addressing a difficult, complex problem. It's
    exactly as complex as it needs to be to solve the
    problem. It doesn't sugarcoat the solution or hide
    necessary details.
- IntelliJ
- Pivotal Tracker
  - even though they make the scrollbar mistake...

## What should change

- Explicitly differentiate between "chrome"/context and
  content, ideally with different background colors. Content
  should be given the highest contrast.
- Give different UI widgets different appearances. Buttons
  should appear raised, inputs recessed. Cards, windows, app icons, and
  other movable or dismissible elements should seem to float
  above the background. The old pattern of an input with
  fully rounded ends meaning "search" should be brought
  back.
- Use large (36px?) high-contrast icons. The tap target for
  touchscreens needs to be at least that big anyway, so you
  might as well use it all for the icon. Differentiate icons
  by shape, color, _and_ position for maximum accessibility.

## But what you're proposing will make software uglier!

No, it won't. What I'm proposing is a renaissance of
design languages and elements that were once considered
beautiful. What has changed since then, I think, is that
people have
experienced a large amount of software using that design
language that was awful for reasons that had nothing to do
with visual design, and now those design elements leave a
bad taste in their mouth. No design language is immune from
this effect, of course—bad design will always be with us.

If my proposals seem regressive, consider that
a) I'm not a designer. Actual designers will probably have
  more creative ways of solving the problems I've described.
  All I've done is articulate the benefits of particular
  solutions—but other solutions might have the same or
  greater benefits.
b) Current design trends are _also_ regressive. "Modern"
  flat UI resembles nothing so much as Apple's System 6 (from the 1980s).
