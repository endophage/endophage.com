+++
title = "Style Doesn't Matter"
date = "2017-01-30T19:59:57-08:00"

+++

OK, so the title leaves a bit out. Coding style absolutely does matter. What
I really want to tell you is that _your_ coding style, specifically, doesn't 
matter. I don't care if you prefer spaces to tabs, like your braces on the same
line or the next line, or prefer underscores vs camelCase.

When you work as part of a team, the purpose of coding style is to create a 
uniformity to the code, such that any member of the team not only knows what
to expect, but knows what is expected of them. You should be able to open any
file in a team project you're working on and, beyond anyone that likes to pick
quirky variable names, or that Brit who keeps putting u's where they belong, 
it should be impossible to tell which member of your team wrote it.

## Picking a style

When it comes to agreeing on a coding style, engineers have been known to hold
strong opinions. Even worse, a lot of us are really bad at acknowledging it's
only an opinion. This is why I've come to love Go. The tooling provided by the 
Go team is, in my opinion, second to none when it comes to enforcing a uniform
style. Furthermore, because the tooling is easy to use, and some is even integrated
into the core `go` binary, it's widely used. This has created a uniformity of 
style not just within a team, or company, but across the entire ecosystem.

Where possible, it's obvious that an officially sanctioned coding style should 
be preferred, i.e. PEP8 for Python. In the absence of an official standard,
some larger companies have publicised their coding standards for various 
languages. Remember, your preferences on coding style don't matter, you just
want everyone on your team to write code that looks and feels the same.

## Automate it

If you're going to have a coding style (and you should), start by following
Go's example and use tools to enforce it automatically as part of your CI system.
There are tools for many languages that enforce official standards where they
exist, and where they don't, there are often tools available for popular 
de-facto standards. Choose strict tooling, the goal is uniformity, not
personal comfort. Your preferences on coding style _don't matter_!

## Let it go

Don't be selfish. There are no winners in arguments on style. Style doesn't
matter; as long as everyone on a project uses the same style.