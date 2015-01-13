---
layout: post
title: NAT TRAVERSAL （1）
description: "介绍了NAT穿透技术，目前在实现了UDP方式，TCP正在研究中"
category: sample-post
tags: [UDP, NAT, 渗透]
imagefeature: nat_1.png 
comments: true
share: true
---

Syntax highlighting is a feature that displays source code, in different colors and fonts according to the category of terms. This feature facilitates writing in a structured language such as a programming language or a markup language as both structures and syntax errors are visually distinct. Highlighting does not affect the meaning of the text itself; it is intended only for human readers.[^1]
<!--more-->

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>

### Pygments高亮代码块

#### C

{% highlight c %}
#include <stdio.h>
int main(void)
{
    printf("Hello World\n");
    return 0;
}
{% endhighlight %}

#### Java

{% highlight java linenos %}
class helloworld
{
    public static void main(String args[])
    {
        System.out.println("Hello World");
    }
}
{% endhighlight %}

### 标准代码块

#### Python

~~~ python
#!/usr/bin/python
printf("Hello World")
~~~

### 行内代码

#### Bash

`echo "Hello World"`
