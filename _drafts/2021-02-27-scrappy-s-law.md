---
layout: post
title: Scrappy's Law
---

Scrappy's Law (I made up the name and might change it if I
  think of a better one) goes like this:

> Functionality eventually gets used just for
> its effects, not its intended purpose.

I've now seen enough instances of this happening at many
different levels of scale that I think it's reasonable to
call it a "law". In this post I'll recount some
of the instances I've observed, and muse about how knowledge
of this law might affect our designs.

## Case Study 1: Github Stars

I just use 'em as bookmarks of things I might want to use
in the future. Most of my starred repos are tools I've never
used.

https://github.com/benchristel?tab=stars

Any interpretation of star count as a proxy for quality or
even popularity is likely to be misguided. Stars could just
mean hype.

## Case Study 2: Github Pull Request Reviews

Things that have been said during code reviews I've
participated in:

> Requesting changes to get this out of my queue.

> Requesting changes to reduce the risk that someone merges
> this too soon.

When is a change request not a change request? When it's
made in the context of a rigid, automated system that can't
account for the nuances of human communication!

## Case Study 3: Google Docs Comments

Comments can be used to represent sidenotes in the draft of
a document.

## Case Study 4: Excel

https://www.joelonsoftware.com/2012/01/06/how-trello-is-different/

> Over the next two weeks we visited dozens of Excel
> customers, and did not see anyone using Excel to actually
> perform what you would call “calculations.” Almost all of
> them were using Excel because it was a convenient way to
> create a table.
>
> —Joel Spolsky

People use Excel to make grids and tables.

The grid-ness of Excel is arguably valuable to more people
than its spreadsheet functionality is.

## Case Study 4: `constructUrlFromParts`

I've picked on UI and product designers enough. Now let's
look at how well programmers have grokked the
implications of Scrappy's law (spoiler alert: we haven't).



```javascript
function constructUrlFromParts(parts) {
  return parts.reduce((a, b) => a + b, "")
}
```

```javascript
tokenizedUrls.map(constructUrlFromParts)
  .forEach(url => console.log(url))
```

```javascript
tokenizedUrls.forEach((tokens, i) => {
  // force https
  if (tokens[0] === "http://") {
    tokens[0] = "https://"
  }
  // label each url
  if (labels[i]) {
    tokens.unshift(labels[i])
  }
})

// ...

tokenizedUrls.map(constructUrlFromParts)
  .forEach(url => console.log(url))
```

When I pointed out that the thing being passed to
`constructUrlFromParts` was not actually a tokenized URL, my
colleague just shrugged. The code worked. It passed the
tests. It was "readable", whatever that means—perhaps in the
same way that [mathematically garbled
advertising](https://xkcd.com/870/) is "readable" to someone
who is not thinking too carefully about it. For this way of
writing code has one advantage, and only one: it conveys
(redundantly) to the reader in a hurry that something
related to URLs is going on. To the careful reader, though,
it presents a confounding riddle, because, if we take the
name `constructUrlFromParts` at its word, this code looks
like it should not work.

```javascript
function concatStrings(strs) {
  strs.reduce(plus, "")
}
```

## Consequences

### Don't Remove Functionality

Even if you think some old feature is made obsolete by a new
one, chances are it doesn't actually accommodate all the
same use cases.

### Users Will Work Around Restrictions You Impose—So Don't Restrict Them

Users know their needs better than you do.

Restrictions may have a legitimate purpose (Twitter's
140-character limit was intended for SMS compatibility) and
in this case workarounds (like tweet threads) may be an
acceptable tradeoff, and perhaps future features can turn
those "workarounds" into first-class citizens (c.f.
"worse is better").

Also, more restrictions usually means more complicated code.
Why spend more money to build and test something that just
annoys users? Christin Gorman has a good talk on this, which
I unfortunately can't find on YouTube at the moment.

### Misuse is an Opportunity

So people aren't using your app the way you thought they
would. So what? They find it useful! Pivot away from your
original vision and build the features your users want, not
the features you first thought they'd need.

## Your Plan Is Not Important (to your users)

Scrappy's First Corollary:

> In any organization where following a plan takes priority
> over creating desired effects, the political reality is
> that the product—the thing being planned—isn't actually
> all that important. What's more important is the careers
> of the managers who want to be able to say they
> implemented the plans they wrote a year ago.

## The Effects of Planning

Scrappy's Second Corollary

>
