# Django Tutorial Part 5: Templates

This is part 4 of the tutorial.    
Make sure you finished [Part 4](tutorial_part4.md)

**Quick reminder**: 

In parts 1-4 you created the polls app and deployed it to production server with migrations and initial data.

In this part, you will add templates, which will enable your views to provide complete web pages.

## Reminder: how django works

In a nutshell, the django framework carries out the following process:

1. Gets a URL request from a web server
2. Decides what view function to call for that URL, and call this view
3. The view runs and returns a reposnse to django.
4. Django handles the response to the web server

Your job, as a django developer, is mostly to write views, and hook them to URLs. Django will do the rest. Although in time web applications can become very complex, this is still the crux: writing views and hook them to URLs.

## Two Types of Views

Generaly, there are two types of views:

1. **Views that generate a complete web page**: The view function generates a complete web page, with HTML and everything. When django calls the view, it is the view responsibility to create the **web page & the data**.

2. **Views that return only data**: The view does not need to render a page, it just returns some data. The client is responsible for the rendering. A mobile app needs just the data, and then the app already knows how to render the GUI, or activate the audio, or show the video. Same idea for a rich-client web application, where the client gets only the data, typically via AJAX, and uses the client-side javascript to render the page.


Both types of views typically involve some backend processing. But once this processing is done, the only-data view finishes. It returns the data as-is to django.    
The page generating view still needs, however, to **use** the data and prepare a page.


Here is a simple workflow of these views:


Step| View that generates a complete page | View that responds with data
----|--------------------------------------| -----------------------------
1 |Called by django | Called by django
2 |Gets the request object and additional arguments | Gets the request object and additional arguments
3 |Do something smart | Do something smart
4| Pack the results in a Python dictionary | Pack the results in a Python dictionary
5|A bit more work, have to create a web page | Done, just convert the dictionary to json and return it
6|Build HTML, render page with the data dictionary | 
7|Return the complete page|


Fortunately for the page generating views (and the developers that write them), django has **templates**. Once the data dictionary is ready (step 4), it's easy - just pass it to django with a template of an HTML page. Django will render the template, prepare the complete web page, and send it back to the web server.

## Rendering Templates in Django

Most of the views that render pages are built like this:

	from django.shortcuts import render
	...

	def show_details_view(request):
	
		# get the data by doing something smart
		# pack the data into a python dictionary, say Dcontext

		return render(request,'templatefile.html, Dcontext)
	
	
		
This tells django to render the `templatefile.html` with the data from Dcontext, and send the (rendered) complete page to the web server.

*Note: You can also define a django view as a class. See part 7 of this tutorial, about class based views (CBV). Both function views and class based views render a template in a similar way with the same django templates engine*

What is `render`?

It's a django provided function that means: **"Replace a placeholder in the template, with actual data from a dictionary, by key".**   

If the `templatefile.html` has a line like this :

	<h1> {{ title }} </h1> 
	
And the Dictionary `Dcontext` has a `"title"` key like this:

	{'title':'Hello World!!!'}
	
Then django's render replaces the `{{ title }}` placeholder in the template with the value of `Dcontext['title']`. The final HTML is:

	<h1> Hello World!!! </h1> 


## Adding Templates 

First checkout the polls-app branch.    
Reminder: master should be used only for the stable code, after it's ready for deployment.

From the `site_repo` dir:

	you@dev-machine: git checkout polls-app
	
	
Create the templates directory:

	you@dev-machine: mkdir templates/polls
	

Edit the first template:

	you@dev-machine: nano templates/polls/index.html
	
	
After the edit, the file should look like this:

	{% extends "base.html" %}

	{% block body %}
	
	    {% if latest_question_list %}
	        <ul>
	        {% for question in latest_question_list %}
	            <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
	        {% endfor %}
	        </ul>
	    {% else %}
	        <p>No polls are available.</p>
	    {% endif %}
	    
	{% endblock body %}


### A few hints about this template

The django templates are smart templates, and django has a **template language**. This language can handle some Python objects, basic flow commands, and you can even write your own templates functions (called template tags).    
In the above example, the template will iterate with a `for` loop over the `latest_questions_list`, and will add a line for each entry in the list.
If the list is empty, it will show "No polls are available".

Since template can handle some **Python objects & attributes**, django can render `{{ question.id }}` with the `id` attribute of the `question` object.

The template starts with `extends`. So you don't have to write the complete HTML code again and again. The shared parts of the site are saved in one or more **base templates**, and the rest of the templates **extend** the base.    
When the `base.html` has a global feature, like a favicon, there is no need to re-add it for all the other templates. Just include any global feature in the base template, then extend it.    
The `block` part, here `{% block body %}`, is designed to replace or add  elements of the global templates that are specific to the current template.    

*Note: Django templates have A LOT of features and options. See the django docs.* 

### More Templates

Add another template, for the detail view:

	you@dev-machine: touch templates/polls/detail.html

After the edit, the file should look like this:

	{% extends "base.html" %}

	{% block body %}
	
	    {% if latest_question_list %}
	        <ul>
	        {% for question in latest_question_list %}
	            <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
	        {% endfor %}
	        </ul>
	    {% else %}
	        <p>No polls are available.</p>
	    {% endif %}
	    
	{% endblock body %}
	
### Use the Templates in Views

Now edit the views, so it can use the templates:

	you@dev-machine: nano polls/views
	
	
After the edit, the index and detail views should look like this:

	from django.http import Http404

	def index(request):
   		latest_question_list = Question.objects.order_by('-pub_date')[:5]
    	Dcontext = {'latest_question_list': latest_question_list}
    	return render(request, 'polls/index.html', Dcontext)

	def detail(request, question_id):
   		try:
       		question = Question.objects.get(pk=question_id)
    	except Question.DoesNotExist:
        	raise Http404("Question does not exist")
    	return render(request, 'polls/detail.html', {'question': question})

Make sure you added the `from django.http import Http404` line.

Both views are now using templates, and both ask django to render the html template file with a data dictionary.

### The views also use the database

 
The **index** view queries for a list of the first 5 questions (`[:5]`), sorted decsending, by pub_date (`order_by('-pub_date')`).    

This is the common way to query with django: 

1. Use the model `objects` manager, and then the specific criteria.
2. The `objects` manager gets the "objects" using the criteria. Each "object" is essentially a row of data, but it's also a python object with many addtional methods and attributes (objects typically hold both data and functionality). 
3. The "objects" that the manager returns are packed in a `QuerySet` object, the Python obejct with the data, that the view can work with.

Similarly, the **detail** view uses the `Question` model's `objects` manager, but asks for just one record, by the questions primary key. The method for one record is `.get`, 

Check it! Run the django development server (assume you run it from the site_repo dir):

	you@dev-machine:  .././manage.py runserver
	 

Browse To | What You Shoud See
----------|--------------------
127.0.0.1:8000| The "Hello World!!!" page
127.0.0.1:8000/polls/ | The list of the questions you entered in the admin
Click on the first question link | The details of this question
127.0.0.1:8000/polls/1 | The details of the first question
127.0.0.1:8000/polls/42 | Http404 (unless you entered 42 questions)


## Remove Hardcoded URLs in Templates

When you develop a web application, it happens that you want to change the urls. But if the URLs are hardcoded in the templates, it can take a lot of time and a lot of mistakes.

Django allows to refer to a url by name: whenever you refer to this name, django will pick the correct url for this unique name in the urls.py file.

When you name a url, like "polls:detail", you can later decide to use:

* `www.yourdomain.com/polls/42`
* `www.yourdomain.com/polls/question/42`
* `www.yourdomain.com/polls/detail/42`
* `www.yourdomain.com/polls/42/detail`

And so on. 
    
The name of the url, **"polls:detail"**, didn't change. Any reference to "polls:detail" will be OK, and you can change the actual url in one place.


To add a named url, edit the template:

	you@dev-machine: nano templates/polls/index.html
	
Replace this line:

	li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
	
With this one, which uses the django `url` template tag:

	<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
	
	
Check it in the browser, everything should work the same. 

From a coding perspective, it will be much easier to change the URLs later.    
It's also easier and cleaner way to provide urls throughout your app.

**url namespace:** Note that the `polls:detail` comes from the main `site_repo/urls.py`, where the namespace `polls` is defined, and then the specific name for each url in the app urls file, at `site_repo/polls/urls.py`.

See what has changed:

	you@dev-machine: git status
	
	On branch polls-app
	# Changes not staged for commit:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   polls/views.py
	#
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	templates/polls/
	no changes added to commit (use "git add" and/or "git commit -a")
	
You added the templates, and modified the views.


*Note: the polls templates are saved in the main templates directory, in `templates/polls`. Django allows you to save templates in the app directory, it's a matter of the developer preferences and the specific app*

 
Commit and push:

	you@dev-machine: git add .
	you@dev-machine: git commit -m 'index,detail templates'
	you@dev-machine: git push origin polls-app


## Deployment

You had quiet a progress, everything works, so it's a good time to deploy.

Check on local Nginx/Apache. If you didn't stop Nginx/Apache after testing in Part 4, you just have to reload the site.    
From site_repo:
	
	you@dev-machine: touch wsgi.py
		
This will tell mod_wsgi to reload the code.

If you do need to restart, on OSX:

	you@dev-machine-osx: sudo apachectl restart
	you@dev-machine-osx: sudo nginx
	
Ubuntu:

	you@dev-machine-ubuntu: sudo service apache2 restart
	you@dev-machine-ubuntu: sudo service nginx restart
	
	
Check the local site at `127.0.0.1/polls`

	
*Note: HTML code changes do not require a mod-wsgi reload, and as soon as the template HTML changes django will pick the new file. This is why new HTML code should be deployed only in sync with a reload of the matching Python code. Otherwise django will use the newer HTML templates, with the older python code. It's not a problem if you changed style or markup, but if the template looks for an argument that is available only in the newer python code version, this may trigger an error. See [Deployment](https://github.com/Aviah/one-click-django-docs/blob/master/deployment.md)*

Make sure the polls-app status is OK:

	you@dev-machine: git status
	
	# On branch polls-app
	nothing to commit (working directory clean)

Merge to master:

	you@dev-machine: git checkout master
	you@dev-machine: git merge polls-app

Should be a simple fast forward:

	you@dev-machine: git log --oneline
	
	c286015 index,detail templates
	172af59 polls base views and urls
	9b678e8 polls admin
	09d6541 polls models and migrations
	7cdcb9a added polls app
	6d5d84c the color is blue
	5a04cb6 home page welcome message
	00bfc74 init site repository
	
	
Simple deployment, only Python code & HTML:

	you@dev-machine: fab deploy
	
	
Check it at `www.yourdomain.com/polls`. Should work!
	

Great Work!

You have added and deployed templates, to render some real web pages.

Until now you entered data in the admin, it would be nice to enter data from the website with a common web page form, wouldn't it?
Continue to [Part 6: Forms](tutorial_part6.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|


