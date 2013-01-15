---
layout: post
tags: [regex, regular expressions, java, tutorial, eclipse]
description: Refactoring using Regex in Eclipse
tagline: Refactoring using Regex in Eclipse
---

{% include JB/setup %}

The project I having been working on for the last year or so was first written
in around 2004 meaning it was written for Java 1.4, it had a lot of things wrong
with it. I want to show you several of the techniques I used in refactoring this
project using Eclipse.

These tutorials will come in several parts, this first part addresses using
[regular expressions](http://en.wikipedia.org/wiki/Regular_expression) to
refactor multiline logging.

I will be using Eclipse Juno for this but I believe almost  all of the
techniques I use are available on 3.x. You will need a basic understanding of
regular expressions, if you don't checkout out [this helpful
guide](http://www.codeproject.com/Articles/9099/The-30-Minute-Regex-Tutorial),
it covers .NET but Java uses exactly the same regex syntax as .NET so don't
worry.

This shouldn't need to be said, but if your doing large refactorings like this
one you should always have [Revision
Control](http://en.wikipedia.org/wiki/Revision_control) system like Git in
place. It enables you to quickly roll back any changes if unneeded or bad and
track the changes that you make.

The logging was in a bad way. It was a sort of roll your type of logging that
required a Singleton to access the logging which then used the Java logging
methods and it really wasn't very good or useful.  This was also one of the
hardest tasks in my refactoring.  After  researching logging implementations I
decided upon using [SLF4J - Simple Logging Facade for
Java](http://www.slf4j.org/), it creates a nice decoupling between the logging
which gets invoked and the implementing code.

Anyway, to use [SLF4J](http://www.slf4j.org/) you create a logger object for
each class or for a collection of classes. 

{% highlight java %}
	Logger logger = LoggerFactory.getLogger(ClassToRefactor.class);
{% endhighlight %}

Now we need this logger field in every class that needs logging. Considering
there were about 1000 classes in total and about but 600 use logging, this would
be no simple task to do by hand. My first few attempts were with using
[sed](http://en.wikipedia.org/wiki/Sed), however they weren't very successful,
however you could use the regular expressions I use here and modify them for
sed. It was also a lot safer to use Eclipse, as I could roll back all my changes
with one undo!

This is the first regex (regular expression) I used to find all my class
definitions in all my Java files.

	public (abstract)? class (\w+)\s\{\n

This regex found any public or abstract classes, we can also expand
this to cover private classes.

	p(ublic|rivate) (abstract)? class (\w+)\s\{\n

This regex will not cover any declarations with an extends or implements keyword
in though, we need to expand to cover those case too.

	p(ublic|rivate) (abstract)? class (\w+) (.*)?\s*\{\n

The simple addition of `(.*)?` covers any possible declarations after the class
name. 

<div class='row-fluid'>
	<ul class='thumbnails'>
		<li class='span6'>
			<a href='/images/search-file-menu.png' class="thumbnail">
				<img src='/images/search-file-menu.png'></img>
			</a>
		</li>
		<li class='span6'>
			<a href='/images/search-regex.png' class="thumbnail">
				<img src='/images/search-regex.png'></img>
			</a>
		</li>
	</ul>
</div>

We then can use this class definition and *tack on* a field definition. We then
use Eclipse's replace function using this regex. Make sure you have ticked the
box 'Regular Expression', then click replace.  My replace expression was:

	\0\n\tprivate static final Logger logger =
	LoggerFactory.getLogger(\2.class);

<div class='row-fluid'>
	<ul class='thumbnails'>
		<li class='span6'>
			<a href='/images/replace-regex.png' class="thumbnail">
				<img src='/images/replace-regex.png'></img>
			</a>
		</li>
	</ul>
</div>

The `\0` is the entire match, so we first want to put back the class definition,
then add a new line and a tab `\n\t`. Next is the field declaration of the
logger, and we want to put back the class name as the parameter for
`getLogger()`, the class name has been captured in group 2 `\2`.

<div class='row-fluid'>
	<ul class='thumbnails'>
		<li class='span6'>
			<a href='/images/organize-import-file-menu.png' class="thumbnail">
				<img src='/images/organize-import-file-menu.png'></img>
			</a>
		</li>
	</ul>
</div>

Once we hit replace, most of our classes should now have this
field declaration in and if you give Eclipse a chance to refresh it should show
us an error on every single class we added the field to! This is expected, as we
haven't added the imports for `Logger` or `LoggerFactory`. There are two ways we
could do this through Eclipse.

* Use Eclipse's `Organize Imports` to go through all the Java files and try and
  automatically fix the definitions. This may cause issues if you have multiple
  libraries using the same class names, and Eclipse provides a nice pop-up to
  solve this.

* Another regular expression! This time we want to match the top of the file and
  add our imports. This can be done by matching the package declaration and
  appending below it. This will avoid the clashing class name on organize
  imports completely.
  
The search regex:
	
	^package .*;

The replace regex:

	\0\n\nimport org.slf4j.Logger;\nimport org.slf4.LoggerFactory;

This regex is similar to the last one where we search for the class definition
and replaced it with the line we matched. This time we match the package
declaration: `package com.yourcompany....` and replace it with itself and our
import with a new line between. Refresh Eclipse and all your errors should
disappear!

*Note: This won't work if you have a default package for your project, ie you
have no package declaration*

Next we need to update the old logging functions to use the new logging
function. Most of the logging calls were wrapped around a horrible logging if
statement to check if logging was active for that level and then writing to the
log if the log level was active. There was a check for different levels which
was I could utilise to create calls to my new logging levels. 

This is what the old logging calls looked like:

{% highlight java %}
	public void method() {
		if (StaticLogger.isLevel(LogLevel.CUSTOMER)) {
			StaticLogger.log("Starting method");
		}

		int ans = otherMethod();

		if ( StaticLogger.isLevel(LogLevel.WARNING) ) {
			StaticLogger.log("Finishing method got: " + ans);
		}
	}
{% endhighlight %}

The regex we need to match this is fairly complex, as we need to capture the log
message and the log level.

	if\s*\(\s*StaticLogger\.isLevel\(\s*StaticLogger\.WARNs*\)\s*\) \{\n\s*StaticLogger\.log\(\s*([^)]*)\);\n\s*\}

We can see the structure in the regex but we will replace `LEVEL` with our
desired level to replace. Make sure you note the **excessive** use of `\s*` the
optional character class for any whitespace, any number of times. This is
littering our regex to make sure that every invocation of the logging statement
is captured with varying levels of whitespace. There are a few `\n?` optional
new lines character clases in the regex, these are in case our call has been
split on to multiple lines.

We can actually get rid of most of these *just in case* calls by formatting our
all of our code correctly first. To do this you must first go into the option
and creating a formatting profile. Eclipse already as a built in formatting
profile which we can use, however I like to make my formatting very strict and
we can do this by setting the options in the formatting profile. A new profile
must be created if you want to generate your own formatting profile, you can't
just override the default one unfortunately.

<div class='row-fluid'>
	<ul class='thumbnails'>
		<li class='span6'>
			<a href='/images/organize-imports.png' class="thumbnail">
				<img src='/images/organize-imports.png'> </img>
			</a>
		</li>
		<li class='span6'>
			<a href='/images/formatter-edit.png' class="thumbnail">
				<img src='/images/formatter-edit.png'> </img>
			</a>
		</li>
	</ul>
</div>

Now we can call for a project wide update of formatting. This will restructure
and correctly indent, add or remove whitespace etc and do all of the other
formatting constraints you set up in your profile.

<div class='row-fluid'>
	<ul class='thumbnails'>
		<li class='span6'>
			<a href='/images/format-file-menu.png' class="thumbnail">
				<img src='/images/format-file-menu.png'></img>
			</a>
		</li>
	</ul>
</div>

Once we have formatted all of our source code correctly we can now be more
certain of our regex and can probably remove some of the unnecessary whitespace
character classes such as `\s*` and `\n?`. We easily test our our regex using
the same Eclipse file search feature, just don't click replace click search
instead!

Once we are certain of our regex we can execute the replace. Our replacement
regex will look nice and simple:

	logger.warn(\1);

Where we can replace 'warn' with the level we are replacing. 

Brilliant, now that horrible logging method:

		if ( StaticLogger.isLevel(LogLevel.WARNING) ) {
			StaticLogger.log("Finishing method got: " + ans);
		}

Will be transformed into:

	logger.warn("Finishing method got: " + ans);

Once we have gone through all the different log levels in the old code we can
remove the singleton that was being used for logging, this will show all the
places where the regex has missed cleaning up the old logging methods. There
were certainly a few missed cases when I tried this on my legacy project some
cases, but these were easy to find now as Eclipse gave a nice list of the
problems. It enabled me to either refine my regex and use the same process again
or just go in and edit the code myself.

Part two legacy code refactoring will be available in the next few weeks.

Tom

