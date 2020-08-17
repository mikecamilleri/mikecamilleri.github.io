---
title: "A Test Post from the Beginning of Time"
date: 1970-01-01
license: cc-by-sa
---

This is a test post intended to incorporate all of the [GitHub Flavored Markdown](https://guides.github.com/pdfs/markdown-cheatsheet-online.pdf) that I may use in blog posts here.

## Headings (`<h2>`)

The highest level of heading that may be used within a post is `<h2>`. `<h1>` is reserved for the title. 

### Subheadings (`<h3>`)

The lowest level of heading that may be used is `<h3>`. Two levels should be enough for even the most complex blog posts. If more are necessary, the post may be better suited to its own website. 

## Inline Features

**Bold** and _Italic_ text are supported. Although `*` and `_` are interchangeable in Markdown, I always use `**` for **bold** and `_` for _italic_. [Hyperlinks](https://mikecamilleri.com) are supported. I won't demonstrate images here, but the basic syntax is `![alt text](url)`. Back-ticks are used for `code`. `\` can be used to escape \*\*special\*\* characters. 

"Quotes" and apostrophies such as in the word _it's_ should be converted to the so-called "smart" versions--em-dashes should work too!

## Block Features

### Blockquotes

Blockquotes with citations are a mess in both Markdown and HTML. This needs to be worked on. 

One quote following another...

> I must die, must I? If at once, then I am dying: if soon, I dine now, as it is time for dinner, and afterwards when the time comes I will die. 
> 
> --Epictetus, _Discourses_

> Remembering that you are going to die is the best way I know to avoid the trap of thinking you have something to lose. You are already naked. There is no reason not to follow your heart.
> 
> --Steve Jobs

A poem...

> this should be
> on four lines
> like a poem
> fingers crossed

And a multi-paragraph quote with citation...

>I went to the woods because I wished to live deliberately, to front only the essential facts of life, and see if I could not learn what it had to teach, and not, when I came to die, discover that I had not lived...
>
> ...I wanted to live deep and suck out all the marrow of life, to live so sturdily and Spartan-like as to put to rout all that was not life, to cut a broad swath and shave close, to drive life into a corner, and reduce it to its lowest terms...
>
> --Thoreau, _Walden_

### Lists

An ordered list with nested unordered list...

1. one
2. two
   - a
   - b
3. three

An unordered list with nested ordered list...

- a
- b
  1. one
  2. two
- c

### Code Blocks

Code blocks support syntax highlighting. This is in Go...

```go
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```
