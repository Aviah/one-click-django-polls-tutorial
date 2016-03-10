# Django Tutorial Part 6: Forms

This is the 6th part of using the official django tutorial in the one-click-django environment.    
Make sure you finished [Part 5](tutorial_part5.md)

**Quick reminder**: 

In parts 1-5 you have created the polls app, and added initial data with the django admin. The website already serves a few pages with the app urls, views, and templates.


## Intro to Forms in Django


Until now, the browser requests from the django site was a simple URL. Django had to matched the URL with the urls list in the urls.py files, and once a match was found, django called the correct view for that URL.

When you use forms, the client (the browser) sends additional information . If it's a GET form, the form arguments are sent in the url. When it's a post form, via HTTP:


Feature | GET | POST
-----|-----|------ 
URL|www.yourdomain.com/polls?arg1=foo&arg2=baz|www.yourdomain.com/polls
Form arguments are sent| In the URL | In the HTTP request
Available to the django view with | request.GET | request.POST
Form is translated by django to| Python dictionary | Python dictionary


So, in a nutshell, any view can accept forms, and any view that accepts forms can use the python dictionary that django creates from the form data 


*Note: Class based generic views are really great when it comes to forms, the generic views hook all the forms-views steps together and provide the ready cleaned data*

*Note: POST is more secure, mainly because URLS are saved in the browser history and in the server logs, so even when the site uses https there is a way to see the form data*


## Add a Form


First, to the development branch:

	you@dev-machine: git checkout polls-app
	

Edit the detail form:

	you@dev-machine: nano templates/polls/detail.html
	
	
After you edit the file, it should look like this: 

	{% extends "base.html" %}


	{% block body %}
	
	    <h1>{{ question.question_text }}</h1>
	    
	    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
	    
	    <form action="{% url 'polls:vote' question.id %}" method="post">
		    {% csrf_token %}
		    {% for choice in question.choice_set.all %}
		        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
		        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
		    {% endfor %}
		    <input type="submit" value="Vote" />
	    </form>
	
	{% endblock body %}

Quite a few new things here:

1. The HTML form **action**: The target URL when the user submits, goes to the polls:vote url. Django uses the url name to provide the actual url.
2. The HTML form **method** is "post", so the view will read the data in request.POST
3. A loop that adds a radio button for each choice
4. A **submit** button.
5. Django renders the form fields, but the you define the **form tag** and the **submit button**.


Add the view code that will handle the (truth) form. Edit the views file:

	you@dev-machine: nano polls/views.py
	
	
The imports statements should look like this: 

	from django.core.urlresolvers import reverse
	from django.http import HttpResponse,HttpResponseRedirect, Http404
	from django.shortcuts import render,get_object_or_404

	from .models import Choice,Question 
	
This is the votes view:

	def vote(request, question_id):
    p = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = p.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': p,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(p.id,)))


Like any view that accepts a form, when called, the view has to decide if:   

1. The user asked for a new empty form
2. Or, the user filled the form and submited it

So if the question exists at all (otherwise the view raises 404), the view tries POST['choice'], and redisplay the form if it doesn't exists. Otherwise, the vote is increamented and saved, and the view redirects to the results url.    

The results url is found with **reverse**, which takes a named url, and return the path for it:

	 reverese('polls:results',1)
	 
will return:

	 /polls/2/results    

Again, using reverse makes it easy to change the URLs path in the future, and it a better and cleaner way to manage urls for django development.

*Note: the **choice_set** is a method that django adds automaticaly for one-to-many relations, as an easy way for the one side object to get the related many side objects. Django knows about the questions-choices one-many relation from the ForeignKey in the polls/Choices model.* 

To see the results, edit the results view as well, so it looks like this:

	def results(request, question_id):
   		question = get_object_or_404(Question, pk=question_id)
   		return render(request, 'polls/results.html', {'question': question})
 

    		
Add the results.html template:

	you@dev-machine: nano templates/polls/results.html
	
After the edit it should look like this:

	{% extends "base.html" %}

		{% block body %}
	
	   		<h1>{{ question.question_text }}</h1>
	    
	    	<ul>
	    		{% for choice in question.choice_set.all %}
	        		<li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
	    		{% endfor %}
	    	</ul>
	    
	    	<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
	    
		{% endblock body %}
		
		
Run the django development server:

	you@dev-machine: .././manage.py runserver


Check at **127.0.0.1:8000/polls/1**

If you entered choices for this question, the voting page should appear. vote and submit - and the results page shows.

Commit and Push:

	you@dev-machine: git status
	you@dev-machine: git add .
	you@dev-machine: git git commit -m "voting with a form"
	you@dev-machine: git push origin polls-app




# Deployment


Reload the site and test localy with Apache/Nginx:

	you@dev-machine: touch wsgi.py
	
Check at **127.0.0.1/polls/1**


Should work.
	

Merge to master:

	you@dev-machine: git checkout master
	you@dev-machine: git merge polls-app
	you@dev-machine: git push
	
	
We don't really need the polls-app branch now. When you will add more new code, you  will create a new temporary branch based on master.    
Delete:

	you@dev-machine: git branch -d polls-app
	
	Deleted branch polls-app (was 2feacb0).
	

OK! Ready for deployment!    
It's only Python & HTML, so simple deployment will do:

	you@dev-machine: fab deploy 
	


Cast your vote at your site, at **www.yourdomain.com/polls**



Great work!

You added a form view, and now have a more streamlined web application. 

This is the last step of this tutorial. You will probably want to advance toward building a full web application, so
Continue to [Part 7: What Next and Some Resources](tutorial_part7.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|

















	
