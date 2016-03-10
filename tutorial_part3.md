# Django Tutorial Part 3: Views & URLs

This is the 3rd part of using the official django tutorial in the one-click-django environment.    
Make sure you finished [Part 2](tutorial_part2.md)

**Quick reminder**: 

In parts 1 & 2 you have created the polls app, and added some initial data with the django admin.

Now we need the views, the django functions that render the webpages and let you browse to the app.


## Views
Form the site_repo directory:

	you@dev-machine: nano polls/views.py
	
After the edit, the file should look like this:

	from django.http import HttpResponse
	from django.shortcuts import render
	
	from .models import Question
	
	def index(request):
	    latest_question_list = Question.objects.order_by('-pub_date')[:5]
	    output = ', '.join([p.question_text for p in latest_question_list])
	    return HttpResponse(output)
	
	def detail(request, question_id):
	    return HttpResponse("You're looking at question %s." % question_id)
	
	def results(request, question_id):
	    response = "You're looking at the results of question %s."
	    return HttpResponse(response % question_id)
	
	def vote(request, question_id):
	    return HttpResponse("You're voting on question %s." % question_id)

	
	
The polls app has 4 views: index,detail,results,and vote. Each view will render a different web page.

Of course these are not real HTML files that the web server sends to the browser, but rather python functions. A django view function dynamicaly creates the web page (in django: an HttpResponse object), which django handles and passes to the webserver via mod_wsgi.

Note that each view accepts a "request" argument, which is a django HttpRequest object. This is the python class that django builds from the HTTP request it got from the server. The HttpRequest object has a lot of data about the actual request, and the view uses this data to render the response page.

What is these views role in the django workflow? The steps to run these views is as follows:

1. Django gets the URL from the webserver
2. Django decides which view is responsible to create a page for this URL, and calls this view
3. The called view runs, and returns an HttpResponse object to django.
4. Django returns the final response to mod_wsgi
5. mod_wsgi sends it to Apache, Apache to Nginx
6. Nginx, which is the server that faces the internet, sends the response to the user's browser. 
7. 

This is exactly the nice thing in a web framework, and django in particular: Your main job is, essentialy,  to define URLS and write the functions that respond to those urls. Then (almost) everything else is handled by the framework.

*Note: When the client asks for a real file (image, css file, javascript file), Nginx does **not** need to go to Apache, and directly sends the file. This is much faster.*   


*Note: Regarding Apache & Nginx. Once Apache handled the response to Nginx, the Apache-mod-wsgi-django thread is free to run another request, since Nginx now takes care of the response and sending it via the actual connection to the client.    
The heavy lifting is the code that runs in django-mod_wsgi-Apache. This is why it's better to free resources ASAP, let django-mod_wsgi-Apache to deliver the response localy, to Nginx, rather than wait until Apache-mod_wsgi deliveres it to the client over the web. Nginx is much better handling the the clients, and the delivery over slow connections, disconnections etc, while django-mod_wsgi threads are free to run the next request*


## URLs

To recap, the flow of request-response is:

URL to django >> django calls the correct View >> the view returns an HttpResponse to django >> django to mod_wsgi >> to Apache >> to Nginx >> Internet >> User's Browser.

We already have the **views** in place.

Now we have to define **urls**, to tell django how to decide which view to call for each URL.    
We will add a urls.py file for the polls app (note that each app can have it's own urls.py file):

	you@dev-machine: nano polls/urls.py
	
After the edit, the file should look like this:


	from django.conf.urls import url

	from . import views
	
	urlpatterns = [
	    # ex: /polls/
	    url(r'^$', views.index, name='index'),
	    # ex: /polls/5/
	    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
	    # ex: /polls/5/results/
	    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
	    # ex: /polls/5/vote/
	    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
	]

	

The urls defenitions are maybe the most confusing part of django. In a nutshell, djago has a list of possible urls, and a specific view to call for each url.
The confusing part is that django matches urls with regex, and it also extracts arguments from the url and pass these argument to the view (the view is a function, so it accepts arguments).

**But the basic idea is simple: django gets the URL that the user requested, match it to one of the urls in the list, and call the correct view for this matched url.**

By default, django looks only at the main urls.py file, at site_repo/urls.py).   
So we need to tell django to try to match urls also from poll/urls.py.    

From the site_repo directory, edit the **main** urls.py file:

	you@dev-machine: nano urls.py
	
After the edit, the urlpatterns should looks like this:

	urlpatterns = [
	    url(r'^polls/', include('site_repo.polls.urls', namespace="polls")),
	    url(r'^admin/', include(admin.site.urls)),
	    url(r'^$', home_page, name='hello-world'),
	]
	
The include function will let django match urls from the polls app.
	
Test it! Run the django development server:

	you@dev-machine: ../.manage.py runserver
	
And browse:

Browse To | You Should See
----------| --------------
127.0.0.1:8000/polls |  A list of the questions you entered in the admin
127.0.0.1:8000/polls/42/ |  You're looking at question 42.
127.0.0.1:8000/polls/42/results |  You're looking at the results of question 42.
127.0.0.1:8000/polls/42/vote |  You're voting on question 42.

It's clear that the only first view is actually using the database. In the case of the other views, django just parses the "42" argument from the url, passes it to the view, and the view renders the response with the argument.

You have successfuly added views and urls to the polls app:

	you@dev-machine: git status
	
	# On branch polls-app
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   polls/urls.py
	#
	# Changes not staged for commit:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   polls/urls.py
	#	modified:   polls/views.py
	#	modified:   urls.py

Commit:

	you@dev-machine: git add .
	you@dev-machine: git commit -m 'polls base views and urls"


Great Work!
	
	
It seems that you have something to see on a real website, so the next tutorial will be the first deployment.    
Continue to [Part 4: First Deployment](tutorial_part4.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|



	











	
