# Django Tutorial Part 7: A Brief Overview of Django's Base Components
This is the 7th part of using the official django tutorial in the one-click-django environment.    
Make sure you finished [Part 6](tutorial_part6.md)

## Django Components

Django is a mature and feature rich framework, and if you need to do something, chances are that this functionality is already available in django.

It helps to have a good grasp of the base components, so here is a quick recapץ

## Models

A django "Model" is a python class. It's the database facing part of django. You can think of a model as a "smart table", it has fields (a field is also a python class), and it can run a lot of code before sending something to the database, and after getting the data back, to prepare it for the django application. The models are defined with a python Metaclass: you provide the definition, and django creates your Model class in runtime.

The model provides the data as an "object": essentially, an object is a data row, encapsulated with python methods and properties.

Models are a great and easy way to work with data: you get both the data, and the functionality to manipulate it, in one python object. 

The limits of the models start to appear when you work with large sets of data. Sets are the task that SQL was designed to solve, while object oriented approach is somewhat limited in that regard.    
The django ORM (object-relational-mapping) can take you a long way with aggregates and sets, but it has it's limit. Luckily, django provides easy access to raw sql, when you need it.


## Forms
A django form is also a python class, with fields, not unlike to models. But where the model's primary job is to face the database, the form main task is to face the user input.    
So while the model's fields and behaviour are database oriented, the form's fields have properties, widgets and behaviour required for HTML forms and user input. 

The "ModelForm" is the bridge between the django models and django forms. The ModelForm connects the form's fields to the model's fields, and let you easily get and save data in one step from the user to the database. When the ModelForm doesn't fit, you will have to connect the form and the model in your code, get the data from the user in the form, and pass it to the model.


## Querysets

A django model is the django abstraction for a database table. A Queryset is a python class that abstracts a query. It's a essentially a list-like python class, with a list of rows that the database returned for a given query criteria.    
But since an object in django is of course much more than a "row", a queryset item is typically a complete python object with the data and functionality. 

Like any other python list (it's not a real list, but similar to one), you can loop through the queryset, the list of objects, use list comprehentions, etc. 

The confusing issue with Querysets is that a queryset object has more than one state: when you define a queryset, it just has the criteria for the sql query, like objects.filter(field1='foo',field2='baz). The queryset is defined, but without results - yet. Django will not run the query until it actually need it. Only when the code start to reference items for the query - loop the query, access the first item in the queryset, etc - django runs it. And when used incorrectly, the same queryset may run more than once. 

## Views

A view is a python function, or class, that gets an HTTP request, do something with it, and then sends back a response.    

Views are the core component of django: The framework main job is to get the request from the webserver, send it to the correct view, and then take the view result - the response - and send it back to the webserver. As a django developer, the development work is actually writing views: figure out what is the required response for each url, and write the correct view to provide this response.

** Class Based Views (CBV) **    
Django earlier versions used functions as views: the view function received the request as an argument, possibly with more arguments, and returned an HttpResponse. Simple and stright forward.    
Later, django introduced Class Based Views. Like many other django components, it's now possible to define a view by subclassing a django class.   
With the CBV, django introduced generic views, base classes that already do what most common web application views do: add a new item, delete item, edit and item, render a template, redirect, etc. 

The CBV, and the generic views, are great if you need the very common use cases (e.g.  show a simple list of items). But once you need some not-so-obvious functionality, it becomes incredibly complicated to use the correct subclass, mixins, methods and properties. To get an idea, see the [complete CBV reference](https://ccbv.co.uk/).
Another disadvantage of CBV is that the flow of execution is less clear than a simple function. It's harder to debug when things run somewhere in the inheritence chain.    
However, again, CBV are great to have a progeress fast, and much simpler code on the common things. It's effective when you need a common functionality that repeats  in many views (or to add such functionality to many views later in the project). In a way, class based views better fit django's general concept of exposing core components as classes (models, forms, handlers, etc), and let you subclass and customize what you need. 
   
So CBV are somewhere between an amazing toolbelt on the one hand, that let's you code these tedious repeating tasks in just a few lines (not unlike how django saves you the time of dealing with HTTP), or, on the other extreme, an insane [complex architecture](http://www.joelonsoftware.com/articles/fog0000000018.html).

At the end of the day, the CBV issue is the old OOP issue, applied to views. OOP has a lot of  pros, but also cons, and it's best to know both CBV and function based views, and choose what fits for your personal programming style, the project, and the specific view at hand.

Important to read:     
[The django docs about class based views](https://docs.djangoproject.com/en/1.8/topics/class-based-views/intro/)      
[Django's CBVs were a mistake](http://lukeplant.me.uk/blog/posts/djangos-cbvs-were-a-mistake/)    
[Django's CBVs are not a mistake (but deprecating FBVs might be)](http://www.curiousefficiency.org/posts/2012/05/djangos-cbvs-are-not-mistake-but.html)   
[My approach to Class Based Views](http://lukeplant.me.uk/blog/posts/my-approach-to-class-based-views/)


## Templates

Templates helps the view to generate a response: once the view has the finalized data for the response, it can pass to django the data as a dictionary, and a template. Django renders the template, and sends it as the response.

Templates has some logic, with django templates language, templates tags, and you can add your own tags and filters. It's not always clear where to put the logic: in the template, or in Python. This is a preference issue: sometimes it's easier to add things right in the template, sometimes template logic becomes complicated and it's easier to do it in Python. If some logic, rules or calculation repeats throughout the django app, or requires a lot of parameters and lines of code, better to put in Python.

The common pattern is to define a base template, the general structure of the page , the refernces, and then extend this template, so that each template defines just the specific function the view uses - the rest comes from the base template. If the application uses login and public areas, defince a base template for login pages, and another one for public pages. Both can extend the global base template.

## URL Resolvers
Django maps each url to a specific view. By resolving a url django can find the correct view by a url, and also the opposite: create a url for a given view, with reverse. 


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
Django has many other components, and the idea is usually the same: the developer can use the out-of-the-box django class, or subclass it and customize what is needed.



Continue to [Part 8: What Next and Some Resources](tutorial_part8.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|






