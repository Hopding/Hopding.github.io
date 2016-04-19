---
layout: page
title: CSS Demo
---

Let us start with a demo of the horizontal rule and the 6 header sizes:

---

# <h1>

## <h2>

### <h3>

#### <h4>

##### <h5>

###### <h6>

Well, that was *nice*, wasn't it? Okay, let's move on to inline code and code blocks & highlighting.

## Code
Here's some `inline code`. Cool, right?

And then we can do an unhighlighted code *block*:

```
<h1>HTML</h1>
<p>Some html junk</p>
```
But we can also <font color='pink'><i>highlight</i></font> a code block!
{% highlight java %}
public class Hopding {
  public static void main(String[] args) {
    /*Recursively print to STDOUT*/
    for(int i = 0; i < someVal; i++) {
	     System.out.println("Hopding!"); //print "Hopding!"
	  }
  }
}
{% endhighlight %}

## Lists
1. This
1. is
1. a
1. ordered
1. list

* This
  * is
    * a
      * unordered
        * sublist

Fun.

## Text Decorations
We can do *emphasis*/_italics_, **strong emphasis**/__bold__, and even **_combine them!_**
We can also ~~strikethrough~~ text, which is nice.

## You Are Here
[hopding.com](http://hopding.com/)

## The Almighty Blockquote
> We can't forget the powerful blockquote!!

> Here's a super duper long piece of text that will wrap properly within my blockquote after it is compiled by jekyll and loaded into my browser

>```
 Code
 in
 a
 blockquote
 anybody?
```
