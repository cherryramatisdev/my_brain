---
title: How to create your own completion for vim
description: How to create your own completion for vim
tags:
  - vim
  - neovim
  - article
cover_image: ""
canonical_url: 
published: true
---

Have you ever thought about defining your own completion for particular things like emails, contact names or for something complex like tailwind classes or github usernames? Well, my goal on this post is to show you how to do it in a simple and concise way!

## A demo first

[![demo video](https://asciinema.org/a/593194.svg)](https://asciinema.org/a/593194)
As you can see on the video above it's quite a simple mechanism to define your
own completion, first you create a function that return an array of strings and
then you assigned to `completefunc` option that accept a function reference and
after that you can trigger the completion by pressing `Ctrl+x Ctrl+u` when on
insert mode. Now let's understand the structure of this function and write a
more exciting one!

## Understanding the completion function structure

Here is the function used on the video above for example:

```vim
function! MyCompletion(findstart, base) abort
  return ['something', 'to', 'complete', 'devto']
endfunction
```

The first argument `findstart` it's a numeric argument and vim will call it with value `1` on the first execution of completion to find the column position of the current word (to position the completion popup for example), then on a second moment vim call this function again with a value `0` where it expects an actual lists of items.

> That's why on the video I got an error saying `E745: Using a List as a Number`, because vim was expecting the column number at that point of the completion cycle.

The second argument `base` is a string with all the previous completion matches grouped together, of course it's empty on the first execution and gradually will be incremented if the user keep pressing the completion keybind.

## Writing the simple version with filter

Now that we understand the basic structure for a completion function let's
improve it a little bit with a filter! Basically we'll detect the substring
that the user typed and return the correct option first so it's easy to select
it.

Here is an example:

```vim
function! MyCompletion(findstart, base) abort
  if a:findstart
    return 0
  endif

  let s:matches = ['something', 'to', 'complete', 'devto']

  if a:base->len() == 0
    return s:matches
  endif

  return s:matches->matchfuzzy(a:base)
endfunction
```

The `matchfuzzy` on this context is a default function of vimscript and help us get an fuzzy matching algorithm that give our simple completion function a **lot** of power, as shown below:

[![demo video about completion with fuzzy](https://asciinema.org/a/593196.svg)](https://asciinema.org/a/593196)

## Last improvement on the basic example

If you mess with this function on your own setup you'll observe that exists a bug, to clarify and keep us on the same page i'll show the bug below:

[![demo video about completion bug](https://asciinema.org/a/593197.svg)](https://asciinema.org/a/593197)

As you can see on the video we can only complete the first word and then everything stop working, that's because we didn't return a proper integer value for findstart! that way vim can't find the start of the current word and provide correct `base` value, this is quite simple to solve and we can increment it like this:

```vim
function! MyCompletion(findstart, base) abort
  if a:findstart
    let s:startcol = col('.') - 1
    while s:startcol > 0 && getline('.')[s:startcol - 1] =~ '\a'
      let s:startcol -= 1
    endwhile
    return s:startcol
  endif

  let s:matches = ['something', 'to', 'complete', 'devto']

  if a:base->len() == 0
    return s:matches
  endif

  return s:matches->matchfuzzy(a:base)
endfunction
```

On this version we're effective walking through the line and updating the `startcol` variable accordingly, that way vim will always find our current incompleted word and provide correct arguments.

## Writing a more exciting completion function

Well at this point I think you understand the structure and power of a custom completion function in vim, so let's spice things up and give a more complex example.

1. Let's assume we have a list of emails on `/tmp/emails` like this:

```txt
cherry@gmail.com
mel@gmail.com
morgana@outlook.com
yaya@heart.com
huelder@insiide.com.br
daniel@heart.com
```

2. We'll write a function that get the content of this file and return the emails using the fuzzy matching algorithm, this can be done with the following function:

```vim
function! EmailCompletion(findstart, base) abort
  if a:findstart
    let s:startcol = col('.') - 1
    while s:startcol > 0 && getline('.')[s:startcol - 1] =~ '\a'
      let s:startcol -= 1
    endwhile
    return s:startcol
  endif

  let s:emails = expand('/tmp/emails')->readfile()

  if a:base->len() == 0
    return s:emails
  endif

  return s:emails->matchfuzzy(a:base)
endfunction
```

3. And voila! let's see it working?

[![final demo with email completion](https://asciinema.org/a/593198.svg)](https://asciinema.org/a/593198)
