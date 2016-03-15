# Django Tutorial Part 6: Forms

This is part 6 of the tutorial.    
Make sure you finished [Part 5](tutorial_part5.md)

**Quick reminder**: 

In parts 1-5 you created and deployed the polls app, which now runs several web pages, with models, views and templates. You also entered initial data with the django admin.

In this part, you will add voting data via a standard web-page form. 



## Intro to Forms in Django

When you use forms, the browser sends the form's data as additional information. If it's a GET form, the form's arguments are sent in the url. When it's a POST form, the arguments are sent via HTTP:


Feature | GET | POST
-----|-----|------ 
URL|`www.yourdomain.com/polls?arg1=foo&arg2=baz`|`www.yourdomain.com/polls`
Form arguments are sent| In the URL | In the HTTP request
Available to the django view with | request.GET | request.POST
Form is translated by django to| Python dictionary | Python dictionary


Any view can accept forms, and any view that accepts forms can use the form's data via the python dictionary that django creates.

*Note: django's Class Based generic Views are really great when it comes to forms. The generic views hook everything together, and provide a ready and cleaned data*

*Note: POST forms are more secure, mainly because the GET urls with the form's arguments are saved in the browser history, and in the server logs. So even when the site uses https, there is a way to see the form data*


## Add a Form


First, to the development branch:

	you@dev-machine: git checkout polls-app
	

Edit the detail template and add the HTML form:

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

1. The HTML form **action**: The target url when the user submits the form, goes to the `polls:vote` url. Django uses the url's name, to provide the actual url's path.
2. The HTML form **method** is "post", so the view will read the form's data in `request.POST`.
3. A loop that adds a radio button for each choice
4. A `submit` button.
5. Django renders the form fields, but you define the `form` tag, and the `submit` button.


## View that Accepts a Form

Add the view code that will handle the (truth) form.
    
Edit the views file:

	you@dev-machine: nano polls/views.py
	
	
The `import` statements should look like this: 

	from django.core.urlresolvers import reverse
	from django.http import HttpResponse,HttpResponseRedirect, Http404
	from django.shortcuts import render,get_object_or_404

	from .models import Choice,Question 
	
This is the votes view:

	def vote(request, question_id):
		p = get_object_or_404(Question,pk=question_id)
    	try:
    		selected_choice = p.choice_set.get(pk=request.POST['choice'])
    	except (KeyError, Choice.DoesNotExist):
    		# Redisplay the question voting form.
        	return render(request, 'polls/detail.html',{
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

The view starts by checking if the question exists at all, otherwise it raises 404.

Then, when a form view accepts a request, it has to decide if:

1. The user just got to the form page, so it asks for a new empty form
2. Or, the user already filled the form and submited it with data

To decide, the view tries `request.POST['choice']`. If it doesn't exist, then the user asked for the form and the view redisplays it.   
Otherwise, it's a sbmitted form: the vote is incremented and saved, and the view redirects to the results url.    


## Django Forms

The tutorial example is simple, and it just uses `request.POST`, as is.     
The more common way - and more robust way - to use forms, is with a django `Form`. 

The django `Form` is a special django class that handles almost everything you need from a form. It has methods to clean the data, to provide the error messages, select the correct HTML widget for each form field, save the submitted data to the database, etc.

So instead of dealing with `request.POST`, you write a `Form` class, define this form fields (similar to the way you define the models fields), and then pass the `request.POST` data to this form - and let the Form do the rest.     
See part 7 of this tutorial about django components.

Django forms, and the integration of forms with models, is one of the most powerful and feature-rich parts of django. For a complete coverage of this topic, see the django docs.


## Redirect after successful submit

The results url is found with `reverse`, which takes a named url, and returns the url's path:

	 reverese('polls:results',1)
	 
Returns:

	 /polls/2/results    

Again, using `reverse` makes it easy to change the URLs path in the future, and it is a better and cleaner way to manage urls for a django project.

*Note: the `choice_set` is a method that django adds automaticaly for one-to-many relations, as an easy way for the one side object to get the related objects from the many side. Django knows about the questions-choices one-to-many relation from the `ForeignKey` field in the `Choice` model.* 

To see the results, edit the results view as well, so it looks like this:

	def results(request, question_id):
   		question = get_object_or_404(Question, pk=question_id)
   		return render(request, 'polls/results.html', {'question': question})
 
    		
Add the `results.html` template:

	you@dev-machine: nano templates/polls/results.html
	
After the edit, it should look like this:

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


Check at `127.0.0.1:8000/polls/1`.

If you entered choices for this question, the voting page should appear. Vote and submit: the results page should show the count for each vote.

Commit and Push:

	you@dev-machine: git status
	you@dev-machine: git add .
	you@dev-machine: git git commit -m "voting with a form"
	you@dev-machine: git push origin polls-app




# Deployment


Reload the site and test localy with Nginx & Apache:

	you@dev-machine: touch wsgi.py
	
Check at `127.0.0.1/polls/1`


Should work.
	

Merge to master:

	you@dev-machine: git checkout master
	you@dev-machine: git merge polls-app
	you@dev-machine: git push
	
	
We don't really need the `polls-app` branch now. When you will add more new code to the polls app, you can create a new temporary branch.

Delete `polls-app`:

	you@dev-machine: git branch -d polls-app
	Deleted branch polls-app (was 2feacb0).
	

OK! Ready for deployment.
    
It's only Python & HTML, so a simple deployment will do:

	you@dev-machine: fab deploy 
	

Finally, cast your vote at your site, at `www.yourdomain.com/polls`!



Great work!

You added a form view.

**Now you have a working polls web application on a real website. Congrats!**

This is the last step of the coding in this tutorial.
The next part is a recap of the "moving parts" of the framework, the common components that you work with as a django developer.    
Continue to [Part 7: A Brief Overview of Django's Base Components](tutorial_part7.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|

















	
