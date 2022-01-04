---
layout: post
title: What Every Software Engineer Should Expect From Their Tools in 2022
---

# What Every Software Engineer Should Expect From Their Tools in 2022

Nine years ago, I started my first job as a software
developer, writing Ruby on Rails code for Groupon. I brought
to the job an abundance of gumption, an over-fondness
for domain-specific lang&shy;uages, and my favorite text editor,
Komodo Edit, which I'd used throughout college.

We had an open-plan office and I sat across the desk from my
manager, Josh Krall. About a week into my tenure there, Josh
walked around to my side of the desk to chat about something
and happened to see me opening a file in Komodo Edit. This
is what I did: I hit `command+O` on the keyboard, which
brought up the Mac OS system dialog for choosing a file. I
then clicked through the folder structure, scrolled to the
file I wanted, and clicked "Open".

"Hang on a second," Josh said. "Does your editor not have
fuzzy search? Because if not, we really need to get you a
better editor!"

It turned out that Komodo Edit _did_ have fuzzy search: a
different hotkey would let me type part of a filename and
get a list of matches that filtered automatically as I
typed. Another keystroke would open the file under my
cursor. Bliss.

So that's the story of how, fourteen years after I wrote my
first program, I learned how to open a file.

Fast-forward to 2021. With COVID driving software teams
every&shy;where to distributed work, there's now a six-digit
number of first-year engineers who've never set foot
in an office. I was very lucky to have Josh as my manager,
and lucky that he walked by my desk when he did. But that kind
of chance encounter and immediate corrective feedback
doesn't happen automatically when work is remote. Who is
going to tell all these new engineers that there's a better
way to open files?

## Knowing is half the battle

There are two types of people in the world: those who have
never thought to type the words "fuzzy finder" into Google,
and those who have. Crucially, if you don't know that a term
exists, you can't search for it, and so unless
someone tells you it exists you can only stumble upon
it by pure luck.

The main reason for my
file-opening incompetence circa 2012 was that I
_simply did not know there was a better way_. The problem
wasn't that I didn't know the answer to my question, it was
that I didn't even know there was a question that could be asked.
The existence of fuzzy finders was, in Donald Rumsfeld's terms,
an "unknown unknown".

It may seem like I'm making a mountain out of a molehill
here, but the point I'm trying to make is not about fuzzy finders, but about tools in general. I really believe that one of the core problems
facing new engineers is a lack of facility with the
staggeringly huge ecosystem of dev tools that surrounds them.
[I wish](https://benchristel.github.io/blog/2018/04/16/verse-not-just-for-poets/) programming were as simple as [_Structure and Interpretation
of Computer Programs_](https://web.mit.edu/alexmv/6.037/sicp.pdf) makes it out to be, where once you've
learned to match parentheses it's pure algorithms and datastructures and metacircular evaluation, but that's
not the world anyone I know actually lives in. Here in the real world we use
Bash and Git and terminal emulators and IDEs and Vim and
debuggers and test frameworks and `ag` and `awk` and the
browser dev tools and the list goes on and on.

## Trying to learn all these things is overwhelming

I don't have a solution. But I can suggest a direction: a
philosophy to adopt. It is this: **_Expect more from your tools_**. Nay, _demand_ more from your tools.

I suppose that's not very helpful advice, on its own. _What_
exactly should you expect and demand? Well, I made a list,
which is what the rest of this post is.

Some of the following tools may be available in your editor or
environment already. For others, you may have to install
plugins. I trust that, once you know that your tools
_can and should_ do these things for you, you can figure out
a solution that works for you and your setup.

### You can navigate and select text with the keyboard

On macOS, holding the `option` key while using the arrow keys
will move word by word, while holding the command key will
fling the cursor as far as possible, to the beginning or end
of the line or the top or bottom of the file. Adding `shift`
to any of these gestures selects text as the cursor moves
over it. You can use these key combos to perform most editing
operations without taking your hands off the keyboard.

### You can switch between apps and windows with the keyboard

On macOS, `command+Tab` rotates through your open apps.
`command+shift+Tab` does the same
in reverse order. `command+~` and `command+shift+~`
switch windows within the current app.

### You can move and tile windows with the keyboard

Tools like [ShiftIt](https://github.com/fikovnik/ShiftIt)
and [Hammerspoon](http://www.hammerspoon.org/) give you an
upgrade from clicking and dragging: with a single key combo,
you can maximize windows, teleport them between monitors, or
snap them to the left or right half of your screen.

### You can install or update any software by typing its name

[Homebrew](https://brew.sh/) is a universal installer for
macOS programs and apps. When I was working at Groupon,
installing and configuring a Postgres database system for my
dev environment was a multi-day effort. Now it's `brew
install postgresql`.

### You can jump to the definition of any variable or function with a gesture

In IntelliJ and related editors, you can hold down the
command key (on macOS) and click on any variable or function
name to jump to its definition. Similar gestures exist
for other editors.

### You can easily find all usages of a variable or a function

In IntelliJ on Mac the hotkey is `option+F7`. Other editors
have similar features. The point is, you should have a way
to discover all the places a function is called that's more
reliable than simple text search (which often turns up many
false positives).

### You can re-run tests automatically when you save a file

Four hundred milliseconds. That is the optimal amount of
time between changing code and seeing test results. It's
fast enough that you don't feel you're waiting, and
leisurely enough that you have time to finish
your thought before the computer shoves a test failure in
your face. Gary Bernhardt talks about this 400 millisecond
threshold in ["Fast Test, Slow Test"](https://youtu.be/RAxiiRPHS9k).
In the UX world, it's known as the
[Doherty Threshold](https://lawsofux.com/doherty-threshold/).

If all the tests you care about run in 400 milliseconds,
you can run them every time you pause to think. Even better,
you can configure your IDE to run them automatically, so you
don't even have to hit a hotkey. If you do that, the test results
become an ever-present source of information like the syntax
highlighting in your editor. Not a tool you have to decide
when to use, but a simple and reliable indicator of status.

Now, some of you may be objecting that a test suite that runs in 400 milliseconds is
just not realistic for any project of even medium size. Let
me clarify that 400 milliseconds equates, in my
calculations, to about 4000 test cases, and I think it's a
rare project in which one team is responsible for more than
4000 test cases.

Now, I _will_ acknowledge that it is actually impossible to
run 4000 test cases in 400 milliseconds using the test
frameworks popular today. The reason is that the test
frameworks are simply too slow. The JavaScript test
framework Jest, for example, takes about 4 seconds to run
3000 trivial tests on my machine, and about 600 milliseconds
to run a single test. All that time is just overhead incurred
by the test framework. But the overhead is not essential.
The reason Jest is slow is that it's doing extra work to
accommodate complicated tests for hard-to-test code. A test
framework that assumed simple tests for well-designed code
could be much faster.

You should expect, and demand, faster tests and test
frameworks that can give you feedback at the speed of
thought. If you're interested in pursuing this,
you might find my test framework
[Taste](https://github.com/benchristel/taste) worthwhile.

### You can comprehend the output of a failing test

The output of your test suite should be terse. When the tests
pass, they should output next to nothing—a green dot for each
test is the most I care to see. If a test fails, the output
should state succinctly what went wrong, and highlight it
in a visible color so you can spot the problem quickly.

Too often, though, I see test suites that spew all kinds of
garbage to the terminal—log messages and exception
stacktraces and who knows what. The failure message for the
actual test is very hard to spot among all the noise. This
isn't really a tooling problem as much as it is a test
design problem—you need to set up your tests so that log
output that would be printed in production is suppressed or
redirected to a file. In the meantime, IDEs can help work
around noisy output, by parsing the test results and
displaying the failures in a neat list.

### You can run the tests in just one file

IntelliJ lets you do this by placing your cursor in a test
file and hitting `control+shift+R`.

### You can run a specific test case

Similarly to the above, you can run a single test in IntelliJ
by putting your cursor in that test and hitting
`control+shift+R`.

### You can debug running code from your editor

Many editors have debugger integration. You can debug code
that's running as part of a test, or in your development
environment.

### You can see the commit history for any file and understand why changes were made

IDEs will show you this information in a nice GUI; in Git
the command is `git log <file path>`. Even if you can view
the commit history, though, you still need to understand what
changed and why. You should expect and demand that your
coworkers write good commit messages!

### You can find the commit that introduced a bug in O(log(n)) time

The tool for doing this is `git bisect`.

### You can see a graph of all your local git commits

This is another feature that IDEs often have; there are also
specialized tools like Git-Gui and `gitk` (which comes with Git,
so you probably already have it installed!) Or you can run
`git log --graph --decorate --oneline --all` which I've aliased
to `git lola` by putting this in my `~/.gitconfig`:

```
[alias]
  lola = log --graph --decorate --oneline --all
```

### You can create aliases for shell commands

Speaking of aliases, did you know you can create aliases
for shell commands you run often? It's as simple as putting
something like the below in your `~/.bash_profile` (on Mac) and
opening a new terminal window.

```sh
alias gs="git status"
```

### You can edit multiple lines at once

One of my must-have editor features is multi-select: the
ability to progressively select occurrences of a word or
phrase and then edit all of them at once. In Atom and VSCode
on Mac, select some text and hit `cmd+D` repeatedly to
select following occurrences. In IntelliJ, the shortcut is
`ctrl+G`.

## You are encouraged to cheat

There is a certain type of programmer machismo (not unique
to men, by the way) that drives people to "do it the hard
way". For the sake of yourself, your peers, and your
company, please don't. A former coworker of mine once
expressed that test-driven development made software
development so reliably straightforward that it "felt like
cheating". You are encouraged to cheat. Seek out the ways of
doing things that make your work quick, easy, and reliable,
and you'll go home happier at the end of each day.
Don't worry, there will still be
plenty of crunchy problems to solve,
plenty of mountains to climb. Don't mistake a slog through
the muck for the ascent to the summit.

## You don't have to learn all these things at once

Learning is hard. It's
hard for _everyone_. The cognitive resources humans have
available for learning and other intellectual tasks are
extremely easily depleted and can only really be restored by
sleep (though they can be briefly boosted by sipping a
sugary beverage like fruit juice). So don't beat yourself up
for what you don't know. You will learn most effectively if
you can focus on one thing at a time and learn just that
thing until it's mastered.

For tips on how to do that, watch [Kathy Sierra's talk
"Making Badass Developers"](https://youtu.be/FKTxC9pl-WM).

## Pairing is doubly essential when everyone's remote

If everyone is working from home, moments like my conversation
with Josh can't happen in physical space. They _won't_ happen
at all, unless you intentionally create opportunities for
real-time collaboration. That means remote pairing.

Remote pairing is difficult, and even more exhausting than
in-person pairing, but I think the learning that results is
well worth it. Pair for an hour each day, and see what people
say about it at the next retrospective. And remember, even
Kent Beck can only pair for 5 hours a day :)

My current favorite tool for remote pairing is [Pop](https://pop.com/).

## Your pair will help with anything you're not trying to learn right now

One of the many benefits of regular pair programming is that
it works around the problem of feeling like you have to know
_everything_ to get anything done. If you're pairing with
someone who's an expert in the tools you're using, you can
lean on them to do all the tool related nonsense that you're
_not_ interested in learning at the moment. Then when it
comes time to use the tool you _do_ want to learn, you can
take the driver's seat and they can stand by to answer your
questions.

And if you're that expert, it's _your responsibility_ to
accommodate this learning time. Be kind, be patient, and resist the urge
to grab the keyboard. It may feel like the coding would go
faster if you drove, and maybe that's so, but the learning
would go slower, and it's learning, not coding, that's the
bottleneck in software development.

Of course, people are not neatly partitioned into novices
and experts. People are skilled at different things and
struggle with different things. Everyone is interested in
learning different things. So at the beginning of your
pairing session, consider discussing with your pair what
each of you would like to learn, and what you're _not_
interested in learning right now.

## Conclusions

I feel like this post is ridiculously long for a glorified
top-whatever list of my favorite editor tricks, but I also
feel like I've barely scratched the surface. I'm sure your
teammates will recommend tools and tricks I've never heard
of, and that's kind of the point! My goal was not to write
the authoritative list of Things You Must Know, but to
provide a starting point that you can use to build a set of
techniques that works for your team.
