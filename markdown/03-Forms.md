Forms processing in Lift
==================

This section gives examples of working with Lift forms in different ways.

Plain old form processing
========================

Problem
-------

You want to process form data in a regular old-fashioned, non-Ajax, way.

Solution
--------

Extract form values with `S.param`.   For example, we can show a form, process an input value, and echo the value back out.  Here we have `dumbForm.html` which contains a form wrapped in a snippet:

```html
<div class="lift:DumbForm">

<form  action="/dumbForm.html" method="post">
  <input type="text" name="it" value="something" >
  <input type="submit" value="Go" >
</form> 

<div id="result"> </div>

</div>
```

Pick out the values in the snippet:

```scala
package code.snippet

import net.liftweb.util._
import net.liftweb.common._
import Helpers._
import net.liftweb.http._

class DumbForm {

  val inputParam = for {
    r <- S.request if r.post_?  // restricting to POST requests
    v <- S.param("it")
  } yield v
  
  def render = inputParam match {
      case Full(x) => 
        println("Input is: "+x)
        "#result *" #> x
      
      case _ =>  
        println("No input present! Rendering input form HTML")
        PassThru  
  }
  
}
```

When you run this code you'll see `No input present! Rendering input form HTML` on the console, and when you press "Go" you'll see `Input is: something` and
the "something" will appear on the page.

Discussion
----------

The `DumbForm` snippet extracts the input parameter using a for comprehension.  The result will be of type `Box[String]` which we then match on to decide what to do next.

In this example, we are also checking that the request is a POST request before extracting the value of the `it` form parameter.

Screen or Wizard provide alternatives for form processing, but sometimes you just want to pull values from a request, as demonstrated in this recipe.


See Also
--------

* _Simply Lift_, section 4.1, [Old Fashioned Dumb Forms](http://simply.liftweb.net/index-4.1.html#toc-Section-4.1) and 4.9 [But sometimes Old Fashioned is good](http://stable.simply.liftweb.net/#toc-Section-4.9).
* [Simple forms](https://github.com/marekzebrowski/lift-basics), example Lift project.


Ajax form processing
====================

Problem
-------

You want to process a form on the server via Ajax, without reloading the whole page.

Solution
--------

Mark your form as an Ajax form with `lift:form.ajax` and supply a function to run on the server when the form is submitted.  Given this form...

```html
<div class="lift:AjaxExample">
 <form class="lift:form.ajax">
  <input type="text" name="info" value="" />
  <input type="submit" name="sb" value="go!" />
 </form>
 <div id="result"></div>
</div>
```

...we use the following snippet to accept the text field input of `info` and send back a JavasScript command to update the `result` div with the value sent to us:

```scala
package code.snippet

import net.liftweb.util.Helpers._
import net.liftweb.http.SHtml
import net.liftweb.http.js._
import JsCmds._

import scala.xml.Text

class AjaxExample {
  
  var inputVal = "default"

  def process(): JsCmd = {
    println("Received: "+inputVal)
    SetHtml("result", Text(inputVal))
  }

  def render = {
    "name=info" #> ( 
        SHtml.text(inputVal, inputVal = _) ++ 
        SHtml.hidden(process) )
  }

}
```

Discussion
----------

The form's `info` input is bound to a `SHtml.text` box which will set the local `inputVal` variable to the value submitted by the form.

The hidden field instructs Lift to call the `() => Any` function (`process`, in this example) when the form is submitted.  The end result is the text entered is echoed back by setting the HTML node `result`. There are many other `JsCmd`s you could send, including `Noop` if you decide to send nothing.

In `SHtml` you will see functions starting with "ajax" (e.g., `ajaxText`).  These are great for field-level Ajax interactions, such as triggering actions on input or selection changes.


See Also
--------

* _Simply Lift_, chapter 4.8 [Ajax](http://stable.simply.liftweb.net/#toc-Section-4.8).
* Example [simple forms](https://github.com/marekzebrowski/lift-basics) Lift project.
* [Server side function order](http://www.assembla.com/spaces/liftweb/wiki/cool_tips) on the Lift Cool Tips Wiki page.
* [SHtml Scala Doc](http://scala-tools.org/mvnsites/liftweb-2.4/net/liftweb/http/SHtml.html).
* Lift's [Ajax Demo page](http://demo.liftweb.net/ajax).

Ajax JSON form processing
========================

Problem
-------

You want to process a form via Ajax, sending the data in JSON format.

Solution
--------

Make use of Lift's `jlift.js` Javascript and `JsonHandler` code. Consider this HTML, which is not in a form, but includes `jlift.js`:

```html
<div class="lift:JsonForm" >

 <!--  required for JSON forms processing -->
 <script src="/classpath/jlift.js" class="lift:tail"></script>

 <!--  placeholder script required to process the form -->
 <script id="jsonFormScript" class="lift:tail"></script>

 <div id="formToJson" name="formToJson">
  <input type="text" name="name" value="Royal Society" />
  <input type="text" name="motto" value="Nullius in verba" />
  <input type="submit" name="sb" value="go!" />
 </div>
 <div id="result"></div>
</div>
```

The server-side code to accept the input as JSON would be as follows:

```scala
package code.snippet

import net.liftweb.util._
import Helpers._

import net.liftweb.http._
import net.liftweb.http.js._
import JsCmds._

import scala.xml._

class JsonForm {

  def render = 
     "#formToJson" #> ((ns:NodeSeq) => SHtml.jsonForm(jsonHandler, ns)) &
     "#jsonFormScript" #> Script(jsonHandler.jsCmd)   
    
    object jsonHandler extends JsonHandler {
      
      def apply(in: Any): JsCmd = in match {
          case JsonCmd("processForm", target, params: Map[String, _], all) => 
            val name = params.getOrElse("name", "No Name")
            val motto = params.getOrElse("motto", "No Motto")
            SetHtml("result", 
                Text("The motto of %s is %s".format(name,motto)) )      
          
          case _ => 
            SetHtml("result",Text("Unknown command"))
      }

    }
}
```

If you click the go button and observe the network traffic, you'll see the following sent to the server:

```json
{ "command":"processForm",
  "params":{"name":"Royal Society","motto":"Nullius in verba"} }
```

The server will send back JavaScript to update the `results` div with "The motto of the Royal Society is Nullius in verba".

Discussion
----------

The key components in the example are:

1. `jlift.js` script that makes various JSON functions available; and

2. generated JavaScript code (`jsonHandler.jsCmd`) that is included on the page to perform the actual submission.

In the binding, `SHtml.jsonForm` takes the `jsonHandler` object which will process the form elements, and wraps your template, `ns`, with a `<form>` tag.  We also bind the JavasScript required to the  `jsonFormScript` placeholder.

When the form is submitted, the `JsonHandler.apply` allows us to pattern match on the input and extract the values we need from a `Map`. Note that compiling this code will produce a warning as `Map[String,_]` will be "unchecked since it is eliminated by erasure".

If you are implementing a REST service to process JSON, consider using Rest helpers in Lift to do that.


See Also
--------

* [Using JSON forms with AJAX in Lift Framework](http://www.javabeat.net/2011/05/using-json-forms-with-ajax-in-lift-framework/).
* _Lift in Action_, section 9.1.4 "Using JSON forms with AJAX".
* Example Lift application demonstrating [Simple form](https://github.com/marekzebrowski/lift-basics) processing.
* Section 10.4, JSON, in [Exploring Lift](http://exploring.liftweb.net/master/index-10.html).
* [Nullius in verba](http://en.wikipedia.org/wiki/Nullius_in_verba).

Conditionally disable a checkbox
=================================

Problem
-------

You want to add the `disabled` attribute to a `SHtml.checkbox` based on a conditional check.

Solution
--------

Create a CSS selector transform to add the disabled attribute, and apply it to your checkbox transform.  For example, suppose you have a simple checkbox:

```scala
class Likes {
  var likeTurtles = false
  def checkbox = "*" #> SHtml.checkbox(likeTurtles, likeTurtles = _ )
}
```

Further suppose you want to disable it roughly 50% of the time:

```scala
def disabler = if (math.random > 0.5d)
  "* [disabled]" #> "disabled"
else
  PassThru

def conditionallyDisabledCheckbox = 
  "*" #> disabler( SHtml.checkbox(likeTurtles, likeTurtles = _ ) )
```

Using `lift:Likes.conditionallyDisabledCheckbox` the checkbox would be disabled half the time.

Discussion
----------

The `disabler` method returns a `NodeSeq=>NodeSeq` function, meaning when we apply it in `conditionallyDisabledCheckbox` we need to give it a `NodeSeq`, which is exactly what `SHtml.checkbox` provides.

The `[disabled]` part of the CSS selector is selecting the disabled attribute and replacing it with the value on the right of the `#>`, which is "disabled" in this example.

What this combination means is that half the time the disabled attribute will be set on the checkbox, and half the time the checkbox `NodeSeq` will be left untouched because `PassThru` does not change the `NodeSeq`.


The example above separates the test from the checkbox only to make it easier to write this discussion section.  You can of course in-line the test, as is done in the mailing list post referenced below.


See Also
--------

* Mailing list question regarding [how to conditionally mark a SHtml.checkbox as disabled](https://groups.google.com/d/topic/liftweb/KBVhkuM1NQQ/discussion).
* _Simply Lift_ [7.10 CSS Selector Transforms](http://simply.liftweb.net/index-7.10.html).

Use a select box with multiple options
======================================

Problem
-------

You want to show the user a number of options in a select box, and allow them to select multiple values.

Solution
--------

Use `SHtml.multiSelect`:

```scala
class MySnippet {
  def multi = {
    case class Item(id: String, name: String)
    val inventory = Item("a", "Coffee") :: Item("b", "Milk") :: 
       Item("c", "Sugar") :: Nil
    
     val options : List[(String,String)] = 
       inventory.map(i => (i.id -> i.name))
     
     val default = inventory.head.id :: Nil
     
     "#opts *" #> 
       SHtml.multiSelect(options, default, xs => println("Selected: "+xs))
  }
}
```

The corresponding template would be:

```html
<div class="lift:MySnippet.multi?form=post">
  <p>What can I getcha?</p>
  <div id="opts">options go here</div>
  <input type="submit" value="Submit" />
</div>
```

This will render as something like:

```hmtl
<form action="/" method="post"><div>
  <p>What can I getcha?</p>
  <div id="opts">
   <select name="F25749422319ALP1BW" multiple="true">
     <option value="a" selected="selected">Coffee</option>
     <option value="b">Milk</option>
     <option value="c">Sugar</option>
   </select>
  </div>
  <input value="Submit" type="submit">
</form>
```

Discussion
----------

Recall that an HTML select consists of a set of options, each of which has a value and a name. To reflect this, the above examples takes our `inventory` of objects and turns it into a list of (value,name) string pairs, called `options`.

The function given to `multiSelect` will receive the values (ids), not the names, of the options.  That is, if you ran the above code, and selected "Coffee" and "Milk" the function would see `List("a", "b")`.


### Selected no options ###


Be aware if no options are selected at all, your handling function is not called. This is described in ticket 1139. One way to work around this to to add a hidden function to reset the list.  For example, we could modify the above code to be a stateful snippet and remember the values we selected:

```scala
class MySnippet extends StatefulSnippet {

  def dispatch = {
    case "multi" => multi
  }
  
  case class Item(id: String, name: String)
  val inventory = Item("a", "Coffee") :: Item("b", "Milk") :: 
    Item("c", "Sugar") :: Nil
    
  val options : List[(String,String)] = inventory.map(i => (i.id -> i.name))
    
  var current = inventory.head.id :: Nil
  
  def multi = "#opts *" #> (
    SHtml.hidden( () => current = Nil) ++ 
    SHtml.multiSelect(options, current, current = _)
  )
}
```

Each time the form is submited the `current` list of IDs is set to whatever you have selected in the browser.  But note that we have started with a hidden function that resets `current` to the empty list, meaning that if the receiving function in `multiSelect` is never called, that would mean you have nothing selected. That may be useful, depending on what behaviour you need in your application. 

### Type-safe options ###

If you don't want to work in terms of `String` values for an option, you can use `multiSelectObj`.  In this variation the list of options still provides a text name, but the value is in terms of a class. Likewise, the list of default values will be a list of class instances:

```scala
val options : List[(Item,String)] = inventory.map(i => (i -> i.name))
val current = inventory.head :: Nil
```

The call to generate the multi-select from this data is similar, but note that the function receives a list of `Item`:

```scala
"#opts *" #> SHtml.multiSelectObj(options, current, 
  (xs: List[Item]) => println("Got "+xs) )
```


### Enumerations ###

You can use `multiSelectObj` with enumerations:

```scala
object Item extends Enumeration {
  type Item = Value
  val Coffee, Milk, Sugar = Value
}

import Item._
  
val options : List[(Item,String)] = 
  Item.values.toList.map(i => (i -> i.toString))
    
var current = Item.Coffee :: Nil
  
def multi = "#opts *" #> SHtml.multiSelectObj[Item](options, current, 
  xs => println("Got "+xs) )
```


See Also
--------

* _Exploring Lift_, Chapter 6, [Forms in Lift](http://exploring.liftweb.net/master/index-6.html).
* [Ticket 1139](https://www.assembla.com/spaces/liftweb/tickets/1139), Cannot clear out multiselect.
