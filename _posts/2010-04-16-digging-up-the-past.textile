---
title: "Digging Up the Past"
layout: article
categories: articles
---

My former host, GoDaddy, is gone. I have nothing positive to say about my experience with them as a web host, right up until the moment I hit the cancel button. There is no way to pre-emptively cancel your account aside from conveniently providing a credit card that will expire before the next renewal, effectively making your hosting expire when renewal rolls around. Actively cancelling your account will terminate your hosting immediately. Regardless of whether or not you had time remaining before the renewal rolled around (one month in my case), your site is destroyed, your access is revoked, and no refunds are possible. Not very classy. What's the new solution? GitHub. Slightly more expensive, but I thoroughly enjoy the services GitHub provides and have no qualms about throwing them $7/month for them to honor a CNAME redirect.

I won't mention much about the new site, except to say thanks to the "bro":http://code.alexreisner.com for providing endless nudging to finally do this, and graciously letting me use his template as a basis. Web design isn't my forte, nor one of my major interests in the world of programming. The site itself is written in "Jekyll":http://github.com/mojombo/jekyll which means that I can write mostly in textile markup, and GitHub happily renders it into something gorgeous. I recommend reading the article "Blogging Like a Hacker":http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html if this is something that is appealing to you. It's written by the author of Jekyll and co-founder of GitHub, Tom Preston-Werner.

On a more exciting topic of history, shell history will make your life easier and more productive. As most references on the web focus on the ability to directly recall numbered items (available via the history command), I'll skip to more interesting things. Something I use on an insanely frequent basis is the !! operation, which recalls the last command in its entirety. Why is this so useful? Consider the following:

{% highlight bash %}
$ pacman -Syu
error: you cannot perform this operation unless you are root.
$ sudo !!
Password:
...
{% endhighlight %}

What if the last command isn't exactly what you want? Suppose you're bouncing between vim, gdb, valgrind, and gcc. What if you want to recall not the last command, but the last gcc command? Easily done:

{% highlight bash %}
$ !gc
gcc myproject.c -o .....
{% endhighlight %}

You can add to the command as well.

{% highlight bash %}
$ ls -la ~
# Oops, I meant to grep that...
$ !ls | grep foo
{% endhighlight %}

This only works, however, for commands *starting* with the pattern passed to !. If you want to match a comannd with a pattern found anywhere, you can modify the ! with ? wildcards as such:

{% highlight bash %}
$ !?pacm?
sudo pacman -Syu
{% endhighlight %}

Note that if you're matching on the end of a command, you can omit the trailing question mark.

History can help you correct spelling mistakes. Suppose you're cd'ing to a long directory path but make a mistake:

{% highlight bash %}
$ cd ~/doc/school/COMP233/hw4
cd: no such file or directory: ~/doc/school/COMP233/hw4
# Hmm, I meant to cd to COMP228, not COMP233
$ !!:s/233/228/
{% endhighlight %}

And if you haven't already guessed, that colon operator would work for modifying a recalled command like !gc as well.

Another way to fix typos if you call the wrong command, for example:

{% highlight bash %}
$ echo /var/log/pacman.log
# Well that's clearly not right, we meant to cat it
$ ^echo^cat
{% endhighlight %}

One final tip for you ZSH users out there. Adding the following to your .zshrc will greatly improve your experience with history:

{% highlight bash %}
export HISTIGNORE="&:ls:[bf]g:exit:reset:clear:cd*"
export HISTSIZE=25000
export HISTFILE=~/.zsh_history
export SAVEHIST=250
setopt APPEND_HISTORY
setopt INC_APPEND_HISTORY
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_IGNORE_SPACE
setopt HIST_REDUCE_BLANKS
setopt HIST_SAVE_NO_DUPS
setopt HIST_VERIFY
{% endhighlight %}

Many of the exports apply to Bash as well, but the setopts are ZSH only.

This is just the tip of the iceberg but I find that, for myself, these are my most commonly used history commands. You can, of course, check out m`man history` for a full onslaught of goodness.

