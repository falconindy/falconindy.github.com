---
title: "Don't Mock Me"
layout: article
categories: articles
---

Due to some (favorable) changes in my day job, I've been writing less code outside of work. Without going too much into the reasons, I've become accustomed to having tests available for any code I write. Tests give you a nice baseline for making changes and refactoring. It's easy to know that you've broken something right away. Knowing that you need to write tests also makes you structure your code in such a way that it's reasonable to actually write tests for it. Conversely, making changes to longstanding code that doesn't have tests becomes a chore, especially for more invasive changes. Suddenly, what used to be a weekend hack because I loved getting into the guts of a program starts to look like a real pain in the ass.

It's not that I have an aversion to writing tests, but finding a framework in the open source world that doesn't suck and embracing it can be difficult.  This is especially true given the languages I tend to deal with outside of work, namely shell (Bash, in particular). I'm aware of projects like "shunit":http://sourceforge.net/projects/shunit/ and some of the "derivatives":http://code.google.com/p/shunit2/, but they don't solve some of the real problems that one generally encounters with testing shell.

Let's look at what mkinitcpio does. When you boil it down, it builds CPIO archives. That's really it. You're talking about a lot of raw calls to the filesystem to read/write modules, binaries, and configuration files. Some of the code in its current state is relatively simple to test. Some of it, with a little bit of massaging, would be a lot easier to test. Some of the functions -- namely, some of the important ones like add_module, are currently not at all possible to test due to calls to external binaries like modinfo or modprobe. So what's the solution?

In "real" languages, you have libraries, such as "gmock":http://code.google.com/p/googlemock/, to mock side effects. For shell, I've started writing something I'm calling "Apron":https://github.com/falconindy/apron/.

I'll say it up front: Apron is a hack. But, I've found that everything "good" in shell feels that way. Apron leverages two notable features of Bash:

* The lookup order for command execution favors functions over external binaries.
* The existance of the @command_not_found_handle@ function.

With these 2 things in mind, you can intercept any call which isn't a function, but which also depends on a PATH lookup. By setting @PATH@ to a bogus value (due to a bug going back at least to bash 3.2 you can't simply unset it), you force all these external calls to trigger your @command_not_found_handle@ function. Now, everything you call which isn't a function can do your bidding. You're in a *relatively* safer environment than you used to be, but of course you can still call a binary by its absolute path and avoid the PATH lookup. I'll ignore the fact that Bash will let you define a function called /bin/rm, but you can certainly leverage that, if you know to expect the call. This might tie in to writing your code in a way that it facilitates being tested.

This still isn't very interesting, since causing all your external commands to do nothing will probably also cause your actual code to fail. So, simply define functions with the same name as the binaries you call. Now you can set the behavior for those functions which really matter and simply leave Apron to automatically stub out the rest.

With a little extra bookkeeping, Apron tracks what functions you've defined as mocks and lets you suspend and restore the mocked environment. I've written a simple "example":https://github.com/falconindy/apron/blob/master/mock-test as a test of the framework in action.

It's likely going to take a fair bit of effort, but I'm already quite pleased with how Apron is working (even at under 100 lines of code), and I'm fairly excited about making mkinitcpio more testable on a unit level. Perhaps, in the process, someone else finds this useful too.

Update: Apron supports expectations now, too. The latest "README":https://github.com/falconindy/apron/blob/master/README.md documents how to use it, and there's of course also an "example":https://github.com/falconindy/apron/blob/master/expect-test to go along with it.
