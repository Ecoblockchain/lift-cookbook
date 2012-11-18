Setting the page title
=======================

Problem
-------

You want to set the `<title>` of the page from a Lift snippet.

Solution
--------

Select all the elements of the `title` element and replace them with the text you want:

```scala
"title *" #> "I am different"
```

Assuming you have a `<title>` tag in your template, the above will result in:

```scala
<title>I am different</title>
```

Discussion
----------

It is also possible to set the page title from the contents of `SiteMap, meaning the title used will be the title you've assigned to the page in the site map:

```scala
<title class="lift:Menu.title"></title>
```

The `lift:Menu.title` code appends to any existing text in the title.  This means the following will have the phrase "Site Title - " in the title followed by the page title:

```scala
<title class="lift:Menu.title">Site Title - </title>
```

If you need more control, you can of course bind on title using a regular snippet. This example uses a custom snippet to put the site title after the page title:

```scala
<title class="lift:MyTitle"></title>

object MyTitle {
  def render = <title><lift:Menu.title /> - Site Title</title>
}
```


See Also
--------

* [CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html#toc-Section-7.10) in _Simply Lift_.
* [Wiki page for SiteMap](http://www.assembla.com/spaces/liftweb/wiki/SiteMap)
* [Using <lift:Menu>](http://exploring.liftweb.net/master/index-7.html#toc-Subsection-7.2.3) in _Exploring Lift_.
* Mailing list discussion of [dynamic titles on sitemap](http://groups.google.com/group/liftweb/browse_thread/thread/e19bd2dda2b3159d).


