---
layout: post
title: Your Velocity Is Broken
---

# Your Velocity Is Broken

Or at least, it's a broken metaphor. Let me explain:

- Velocity (in XP and similar methodologies) is supposed to
  measure how fast you're delivering value to the end user.
  The "to the end user" part is key—it's what distinguishes
  velocity from mere speed. Speed is just the amount of work
  you can do in a given time. Velocity cares about your
  direction—i.e. whether that work is actually valuable to
  the people you're working for.
- Velocity is measured in story points.
- Story points are a measure of effort, complexity, and risk,
  not value.
- Therefore, velocity does not measure what it purports to measure. Q.E.D.

If velocity isn't measuring how fast you're delivering
value, what *is* it measuring? Well, roughly, it's measuring
how much *effort* you expend on delivering value. That is a
very different thing.

A team that takes on technical debt and never refactors may
soon find themselves in a quagmire of code, where even the
simplest stories are difficult. As a result, they'll start
pointing stories higher. Their ability to deliver value
declines; the team is suffering, but their velocity remains
high. Low velocity is supposed to be the signal that the
team's process or technical approach needs adjustment, but
in my experience that signal rarely appears, or it gets
lost among the noise of all the other things that affect
velocity.

## The Fix

So how do we make velocity mean what it's really supposed to
mean? I have a few (totally untested) ideas, which I will
gladly and irresponsibly share with you.

### First Estimate Effort, Then Direction

In the flavor of XP my team is currently using, estimation
of stories happens during a weekly planning meeting with
the whole team. The PM leads the team down the backlog and
the developers discuss and then "point" each story. Pointing
is done by everyone simultaneously throwing up a number of
fingers to indicate how many points they think the story is
worth. This prevents people from biasing each others' votes.
If there is wide variation in the number of points people
assign to the story, it's a sign that more discussion is
needed. More rounds of discussion and voting are held until
the points converge.

It wouldn't be too hard to add a concept of "direction" to
the estimate for each story. This maps nicely onto the
velocity metaphor: points tell the PM how much effort we'll
need to expend, and the direction tells her how aligned that
effort is with the user's objectives. Developers can indicate
their estimate of direction by pointing with their arm:
straight up means "100% aligned with delivering value";
45 degrees from vertical means "about half the effort will
be spent on refactoring and tooling". A horizontal arm means
the story is impossible; a negative
slope means you think the story will actually make the
users' lives worse.

If a team starts running into problems with tech debt,
their direction estimates will droop toward horizontal.
By recording a rolling average of direction each iteration,
the team can get a more complete picture of their velocity,
which they can use to inform their refactoring efforts.


### Other Teams Estimate Your Stories For You

If the goal of velocity is not really to measure how much
value is being delivered, but simply to
find out whether a team is suffering from blockers and slowdowns,
it might be worthwhile to have another team with no knowledge
of their codebase or development environment point their stories.
The catch is, the other team has to understand the *product* well,
and the stories must spell out the acceptance criteria very
explicitly, or the team will miss edge cases that add complexity.
I doubt this will work well on many real projects,
but it's an interesting thought experiment to consider how
someone else would view your project. How hard would an outsider
consider this story to be?

One compromise might be to have such an outsider join each planning
meeting and participate in the estimation. But again, this
person would have to know the product well enough to follow
the discussion.
