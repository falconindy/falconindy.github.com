---
title: "Saving Keystrokes With Bash"
layout: article
categories: articles
---

Bash is a wonderful fully featured shell that provides a multitude of ways to cut back on your keystrokes. One of my favorite features, which I don't see used often enough, is brace expansion. Simply put, brace expansion lets you specify multiple similar arguments without retyping the commonalities. Let's take a look at how this is accomplished.

With this handy feature, you can do a multitude of things. You can make a backup copy of a file:

{% highlight bash %}
cp /etc/mpd.conf{,~}  #Same as cp /etc/mpd.conf /etc/mpd.conf~
{% endhighlight %}

Then you can restore that file:

{% highlight bash %}
cp /etc/mpd.conf{~,}  #Same as cp /etc/mpd.conf~ /etc/mpd.conf
{% endhighlight %}

Obviously, it doesn't stop here. You can make a whole directory structure:

{% highlight bash %}
mkdir -p /squashed/{usr,opt}/{rw,ro}
#Same as mkdir -p /squashed/usr/rw /squashed/usr/ro /squashed/opt/rw /squashed/opt/ro
{% endhighlight %}
Bash will expand ranges as well. You can create a zero padded range like so:

{% highlight bash %}
echo {000..100}  #Will print 000 001 002 003 ... 097 098 099 100
{% endhighlight %}
Ranges aren't limited to being numerical. Better yet, expressions can be the preamble and postscript to each other.

{% highlight bash %}
echo {0..9}{A..Z}   #Prints 260 strings!
{% endhighlight %}

If you really use your noodle you can nest expressions, though I've personally not come across a situation where this has been needed (yet).
