# Django Tutorial Part 7: A Brief Overview of Django's Base Components

This is part 7 of the tutorial.    
Make sure you finished [Part 6](tutorial_part6.md)


**Quick reminder**: 

In parts 1-6 you created and deployed the polls app.

This part is not a coding tutorial, but rather a summary of the framework main components. 

The django "essense" is a set of core components, that the django developer uses, subclasses, extends and customize. It's important to have a good grasp of these components, and their roles in the django application.

So this part is a quick recap.
 
[Models](#models)    
[Forms](#forms)    
[Querysets](#querysets)    
[Views](#views)    
[Templates](#templates)     
[URL Resolvers](#url-resolvers)    
[Context Processors](#context-processors)    
[Django Auth](#django-auth)    
[Middleware](#middleware)    
[Signals](#signals)    
[Cache](#cache)    


## Models

A django `Model` is a python class. It's the database facing part of django.     
You can think of a model as a "smart table": it has fields (a field is also a python class), it runs a lot of code before it sends something to the database, and after getting the data back.

Django runs the models with a python metaclass: you provide the definitions, and django's metaclass creates your Model class in runtime.

Models are a great and easy way to work with data: you get both the data and the functionality to manipulate it, in one python object. The model provides the data as an "object": essentially, a model object is a data row, encapsulated with python methods and properties.

**SQL:** The limits of the models start to appear when you work with large sets of data. Sets are the task that SQL was designed to solve, while object oriented approach is somewhat limited with large sets.    
The django ORM (object-relational-mapping) can take you a long way with aggregates and sets, but it has it's limit. Luckily, django provides easy access to raw sql, when you need it.


## Forms
A django `Form` is also a python class, with fields, not unlike models. But where the model's primary job is to face the database, the form's main task is to face the user input.    
While the model's fields and behaviour are database oriented, the form's fields have properties, widgets and behaviour that is required for HTML forms and **user input**. 

The `ModelForm` is the bridge between the django models and the django forms. The `ModelForm` connects the form's fields to the model's fields, and let you easily get and save data in one step, from the user to the database. 

When the `ModelForm` doesn't fit, you will have to manually connect the form and the model, get the user's submiited data in the form, and save it to the model.


## Querysets

If a django `Model` is the django abstraction to a database table, then the `Queryset` is a python class that abstracts a query. 

A `Queryset` is a list-like python class, with a list of rows that the database returned for a given query criteria. However, in django, you get an "object" that is much more than a "row". A queryset item is typically a complete python object with both data and functionality. 

Like any other python list (it's not a real list, but similar to one), you can loop through the queryset and use list comprehentions. 

The confusing issue with `Queryset` is that a queryset object has more than one state: 

* Without data: When you define a queryset, it has only the criteria for the SQL query, like `objects.filter(field1='foo',field2='baz)`. The queryset is defined, but has no data - yet. 
* With data: After django runs the underlying SQL query and populate query, it has the data. 

Django will not run the query to the database until it actually needs it. Only when the application code starts to reference items of the `Queryset`, e.g. to loop the results or access the first item, django runs the SQL, gets the results from the database, and populates the `Queryset`. When used incorrectly, the same queryset may run more than once. 

## Views

A view is a python function, or class, that gets an HTTP request, do something with it, and then sends back a response.    

Views are the core component of django: The framework's main job is to get the request from the web server, send it to the correct view, and then get the view result - the response - and send it back to the web server.
As a django developer, the development work is actually writing views: figure out what is the required response for each url, and write the correct view to provide this response.

**Class Based Views (CBV)**    
Django earlier versions used functions as views: the view function received the request as an argument, possibly with more arguments, and returned an `HttpResponse`. Simple and straightforward.    

Later, django introduced Class Based Views. Like many other django components, it's now possible to define a view by subclassing a django core class.   

With CBV, django also introduced **generic views**, a set of base classes that already do what most common web application views need: add a new item, delete item, edit an item, render a template, redirect, etc. 

The CBV - and the generic views -  are great when you need the very common use cases (e.g.  show a simple list of items). But once you need some not-so-obvious functionality, it becomes incredibly complicated to use the correct subclass, mixins, methods and properties. To get an idea, see the [complete CBV reference](https://ccbv.co.uk/).

Another disadvantage of CBV is that the flow of execution is less clear than a simple view function. It's harder to debug when things run somewhere in the inheritence chain.
    
However, again, CBV are great to get a fast development progeress, and a much simpler code base for the common tasks. It's effective when you need a common functionality that repeats  in many views (or to add such functionality to many views later in the project).   
   
So CBV are somewhere between an amazing toolbelt on the one hand, that let's you code these tedious repeating tasks in just a few lines of code (not unlike how django saves you the time of dealing with HTTP), or - on the other extreme - yet another insane [complex objects architecture](http://www.joelonsoftware.com/articles/fog0000000018.html).

At the end of the day, the CBV issue is similar to the old OOP issue, applied to views. OOP has a lot of  pros, but also cons, and it's best to know both CBV and function based views, and choose what fits for your personal programming style, the project, and the specific view at hand.    
In a way, class based views are a better fit to django's general concept of exposing the framework's core components as classes (models, forms, handlers, etc), and then let the developer subclass and customize.

Important to read:     
[The django docs about class based views](https://docs.djangoproject.com/en/1.8/topics/class-based-views/intro/)      
[Django's CBVs were a mistake](http://lukeplant.me.uk/blog/posts/djangos-cbvs-were-a-mistake/)    
[Django's CBVs are not a mistake (but deprecating FBVs might be)](http://www.curiousefficiency.org/posts/2012/05/djangos-cbvs-are-not-mistake-but.html)   
[My approach to Class Based Views](http://lukeplant.me.uk/blog/posts/my-approach-to-class-based-views/)


## Templates

Templates are typicallly used to generate a web page: once the view has the  data for the response, it can pass this data with a template to django. Django renders the template, and sends the complete page to the web server.

**Logic**: Templates have some logic, with the django templates language and templates tags, and templates can also handle some Python objects. You can even add your own tags and filters.   
It's not always clear where to put the logic: in the template? or in Python, in views, forms, models? This is mainly a developer preference issue: sometimes it's easier to add things right in the template, sometimes the template logic becomes complicated and it's easier to implement it in Python.    
As a rule of thumb, if some logic, rules or calculation repeat throughout the django app, or require a lot of parameters, better to implment it in Python, and pass the results to the template in the context dictionary.

**Hierarchy:** The common pattern is to define a base template, with the general structure of the page and the globaly available features, with the references to the external libraries (jQuery etc). Then extend the  base template to other templates. Each template defines just the specific blocks it needs to change or add to in the base template. All the other features come from the base template as is.

## URL Resolvers
Django maps each url to a specific view. By resolving a url django can find the correct view by a url, and also the opposite: create a url for a given view, with `reverse` in python, or a `url` tag in forms.


## Context Processors
The template rendering engine gets the data from a specific view, but often you need some globaly available data that is used everywhere. Context Processor provide this data, and let you define data dictionaries once, and then use it in all templates, and all the views.

## Django Auth
Django comes with a full authentication system: register, sign in, reset password, change password. This component includes views, urls and forms. Very easy to have a user login authentication fast, but probably requires some modifications later.
The core model used by django is the User model.

## Middleware
Python classes that django calls before the request gets to the view, and after the view return the response. Usefull for code that should run, well, before the view starts or after it finishes. Typically for site global functionality.


## Signals
Signals, like middleware, provide site-global functionality. However, middleware is always called, and alwats runs, for every request. A signal is a conditional call that runs only "when something happens", and you define what this "something" is.  The signal calls another function, which may not be related to the specific view and request at hand.    
When used for models and data, signals provide functionality that is similar to a database trigger. 

## Cache
Django has a cache framework that lets you cache anything from specific data items to entire rendered pages. For storage, as usual, django provides common backends, and you can customize your own. The simplest cache is the file system: django will store the cached data as files, and there is nothing to install. You just have to tell django what directory to use for the cache.

Django  works great, out-of-the-box, without caching. A decent webserver and database will do. As you scale, or need some repeating expensive queries or rendering, cache is useful.

 
## More Components & Customize Django
Django has many other components, and the idea is usually the same: the django developer can use the out-of-the-box django class, or subclass it and customize what is needed.

This is the final step of the tutorial.    
Now build an awesome web application, and dive into advanced advanced in-depth resources!
Continue to: [Part 8: What's Next and Advanced Resources](tutorial_part8.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|






