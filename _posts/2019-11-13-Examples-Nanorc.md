---
layout: post
title: "Examples Nanorc"
date: 2019-10-13 20:00:06 -0700
comments: true
---

## Syntax Files

<img src="/images/posts/nanorc.png" width="40%" alt="nanorc" />

Paths nanorc syntax files
- `sudo nano /usr/share/nano/markdown.nanorc`
- `sudo nano /usr/local/share/nano/markdown.nanorc`

Example sysntax for Markdown
```
syntax "markdown" "\.md$" "\.markdown$"

## Quotations
color cyan "^>.*"

## Emphasis
color green "_[^_]*_"
color green "\*[^\*]*\*"

## Strong emphasis
color brightgreen "\*\*[^\*]*\*\*"
color brightgreen "__[\_]*__"

## Underline headers
color brightblue "^====(=*)"
color brightblue "^----(-*)"

## Hash headers
color brightred "^#.*"

## Linkified URLs (and inline html tags)
color brightmagenta start="<" end=">"

## Links
color brightmagenta "\[.*\](\([^\)]*\))?"

## Link id's:
color brightmagenta "^\[.*\]:( )+.*"

## Code spans
color yellow "`[^`]*`"

## Multi-line code span
color yellow start="^```$" end="^```$"

## Links and inline images
color brightmagenta start="!\[" end="\]"
color brightmagenta start="\[" end="\]"

## Lists
color yellow "^( )*(\*|\+|\-|[0-9]+\.) "

```

Include yours syntax files
`nano ~/.nanorc`
```
include /usr/local/share/nano/*   
```