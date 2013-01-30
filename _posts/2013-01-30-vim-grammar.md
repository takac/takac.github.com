---
layout: post
tags: [vim, grammar, beginner, tutorial]
description: An Introduction to Vim's Grammar
---
## Intro to Vim's Grammar ##

<br /> 
#### Update: Vim Grammar diagram used during my talk at Vim London available [here](/assets/grammar-slide.html)

<br /> 

A grammar is a set of formation rules for strings in a formal language. The
rules describe **how to form strings from the language's alphabet that are
valid according to the language's syntax and structure**. A grammar does not
describe the meaning of the strings or what can be done with them in whatever
context, only their form.

*Paraphrased from Wikipedia*

### Alphabet

Vim's alphabet is formed of all the keys and meta-keys on the keyboard. This
includes combinations of keys pressed together, e.g. `Ctrl-A` is its own entry in
the alphabet as opposed to only `a` and `Ctrl` being defined in the alphabet.

Lets define some of our alphabet mappings:

### Operators

Operators function as the doing words in our grammar. They define how are text will
be processed.

Main Operators:

	d - delete
	y - yank
	c - change

There are around 8 other useful operators, however these 3 comprise most of
operations we will ever do in Vim.

### Motions

Motions are the keys associated with moving around in Vim, whether thats to move
to the end of word or back to the start of the document, these are all motions.

There are **LOTS** of motions and I couldn't hope to cover them all here,
however here is a some subset for our example alphabet.

Common Motions:
	
	h,j,k,l - left, up, down, right
	w,W - to start of next word or WORD
	b,B - to start of previous word or WORD
	e,E - to end of word or WORD
	$   - to end of line
	^   - to start of line

### First Rule

With this simple starting alphabet we can from some simple commands using a
combination of `operator` and `motion`.

	d$ - Delete to end of the line
	yW - copy till end of WORD
	cE - delete till end of the word and go to insert mode

That is our first rule!
	
	rule = operator motion

The 3 words we see above are referred to as symbols in a grammar. The left hand
side can be replaced with the right. We can now add more functionality to our
language by adding more operators or motions.

The rule definitions are similar to regular expressions. Our rules have to match
a certain pattern of characters. In our rules we will also specify where we can
have repetitions of characters similar to regex `+`, `{x}` and `*`, which we will see
later.

### Expanded First Rule

Lets expand our alphabet with another symbol.

Repetition. If we want to repeat our rule above a certain number of times we add
a number in front of the operator.

What form can this take:
	
	repetition = 1..9 { 0..9 }

This formal definition says a repetition starts with a character between 1 and 9,
and can have any number (including zero) more characters form 0 to 9. In other
words the numbers greater than zero.

THEY ARE CHARACTERS, NOT NUMBERS. This is because we are defining our symbols
from raw character strings.

Lets expand on our original rule:

	rule = [repetition] operator motion

The `[]` square brackets in this notation mean that this symbol is optional. So we
can optionally have a repetition as the first symbol of the rule.

	3dw - delete a word 3 times. 

We can however also say:

	d3w - delete forward for 3 words.

Lets add this definition to our rule:

	rule = [repetition] operator [repetition] motion

We could even have:

	3d5w - delete forwards 5 words 3 times

Make sure you understand where the repetition is being applied. This last
command is `3 times delete 5 words forward` and the first repetition applies to
the whole command, it is more like - *do this next operation THIS many times*.

This rule is already looks a bit messy. Lets move the repetition from the
rule definition to the operator and motion definition.

	rule     = operator motion

	operator = [repetition] ( d | c | y )
	motion   = [repetition] ( h | l | j | k | gg | G | ... )

### Text Objects

Grammar definition:

	text-obj = modifier object

Lets define what a modifier is:

	modifier = 'a' | 'i'

A modified is defined the keys `a` or `i`. The bar `|` is our choice operator
here, we have to choose at least one of the symbols in the choice group.

	a - generally means around our object including some whitespace or
		surrounding symbols.

	i - usually means inside of the object, usually excluding whitespace and
	    surrounding symbols.

First object:
	
	w,W - word object, not to be confused with the movement!

In context this has to be used with the `a` or `i` modifiers, just like any
other text object, and it will effect the whole word/WORD regardless of cursor
placement.

When `daw` is used and the cursor is in the middle of a word, it effects the
whole word an and the leading whitespace. This is different and must not be
confused with the behaviour of `dw` which is **delete till.. end of
movement**.

More Objects:

	s - sentence. This is a conitiguous set of words which are terminated by a
	    sentence terminator, generally a fullstop or two line breaks. Vim
		handles multi-line sentences well.

	p - paragraph. New line seperated block of text.

	[ ] { } ( ) " ' ` < > 
		All of these characters work in the same way. These characters specify text
		objects which are wrapped by the corresponding character.

If you wanted to delete your function arguments inside of `function("first", "second",
"third")` and your cursor was somewhere in the middle you can call:

`di(` or `di)`

This leaves us with `function()` with our cursor inside of the brackets. The
*angle* or direction of the bracket does not matter.

`da(` or `da)`

This will leave us with `function` with our cursor at the end of the word.

If we replaced the round brackets with square or curly braces, we can swap
the object specifying character, ie `(` to `{`.

	function()
	{
		var x = "Y";
		// Vim will even work on multiline objects!!
		// Great!
		var y = "X";
		return x+y;
	}

Typing `di{` or `di}` or even `diB` will leave us with a nice empty function
with our cursor on the bottom bracket.

	function()
	{
	}

### Second Rule

We can combine our text objects using one of these two modifiers and an operator.
This is our next rule.

	rule = operator modifer object

Ideally we need to encapsulate this idea into another rule.

	text-object = modifer object

We need to define these symbols more formally.

	object      = w | W | p | s | ( | ) | { | } | [ | ] | " | ' | ` 
	modifer     = a | i

Lets add an optional repetition:

	text-object = [repetition] modifer object

This gives us our simplified rule:
	
	rule = operator text-object

### Back to One Rule

We can combine this with our other rule:

	rule = operator ( motion | text-object )

Our composite rule now covers both text object manipulation and manipulation
using motions.

### More Rules

To create a full grammar we would need to specify all of the commands and there
are lot of corner cases.

We also need to mention a special case in our Vim grammar. When a operator is
called twice it produces its effects on the entire line.

	dd - delete whole line 
	yy - yank whole line
	cc - delete whole line and go to insert mode

New rule for this:
	
	rule = duplicate-op

We can repeat this command too:

	rule = [repetition] duplicate-op

Simple. Says what it means.

	rule = motion

We can also have a general purpose rule for movement. We can see how we are now
building up our grammar for Vim and how this can define all our commands we will
ever use.

### Other rules

	selection-manipulation = selection { motion | text-object } ( selection-operator | operator )

Curly braces specify an optional repetition, similar to the `*` operator in regex, it
allows any number of matches including zero.
 
	
	[repetition] insert-mode {chars} exit-insert
	[repetition] colon {chars} exit-command
	[repetition] operate-special
	[repetition] selection-operator
	[repetition] search-specify
