---
title: Line Numbers
parent: Code
layout: page
---

# Code Snippets with Line Numbers

{: .warning }
In prior versions of the docs, we provided "workarounds" to rendering issues arising from code snippets with line numbers. While these seemed to resolve visual layout issues, they did not resolve core issues with Jekyll generating invalid HTML. See [the detailed explanation](#detailed-error-explanation) for more information.

The default settings for HTML compression are incompatible with the HTML
produced by Jekyll for line numbers from highlighted code
-- both when using Kramdown code fences and when using Liquid highlight tags.

To avoid non-conforming HTML and unsatisfactory layout, HTML compression
can be turned off by using the following configuration option:

{% highlight yaml %}
compress_html:
  ignore:
    envs: all
{% endhighlight %}
