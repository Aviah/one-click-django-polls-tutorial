# Django Tutorial Part 2: The Django Admin


This is the second part of using the official django tutorial in the one-click-django environment.    
Make sure you finished [Part 1: Create the Polls App](tutorial_part1.md)

**Quick reminder**: 

In part 1 you

1. Created the polls app: the app directory & files, and included the app in the django global INSTALLED_APPS settings.
2. Added two app models: Question & Choice. Models are the django represntation and functionality for the underlying tables.
3. You have also applied these two models to the **local** database, and told django to create actual database table for each model (with migrations)

Everything was written in the polls-app branch of the site repository, at the site_repo directory.

Just checingk, from the site_repo directory:

	you@dev-machine: git status
	
	# On branch polls-app
	nothing to commit (working directory clean)
	
We are good to go.

## The Django Admin

Now that the models are defined, and the databases tables have been created, you can add some initial data. We will start with manual data entry via the django built-in admin app.    
But first we need to tell the admin app to hook to the polls app.


	you@dev-machine: nano polls/admin.py
	
After the edit, the file should look like this:

	from django.contrib import admin

	from .models import Question

	admin.site.register(Question)
	
	
Run the django development server:

	you@dev-machine: .././manage.py runserver
	
	
And browse to the admin site, at **127.0.0.1:8000/admin**.
The admin login page should appear: 

1. login with site root username and password you provided when installing one-click-django-dev.
2. The next page is "Site Administration", you should see the "Questions" row, under "Polls". Click "Questions"
3. Click "Add question"
4. For "Question Text" write "What's Up?"
5. Click on "today" and "now" (django is aware that these are date and time fields)
4. Click Save    

You have just added the first question.      
Check it in the database:

	you@dev-machine: mysql -uroot -p
	mysql> SELECT * from django_db.polls_question;
	+----+---------------+---------------------+
	| id | question_text | pub_date            |
	+----+---------------+---------------------+
	|  1 | What's Up?    | 2016-01-21 06:40:46 |
	+----+---------------+---------------------+
	1 row in set (0.00 sec)
	
*Note: There are other ways to enter data into a database, or directly with SQL, but the admin enters the data **via django**, so it's a good way to test to how real data entry works in the web application, and verify that all that additional calls like signals or custom save methods run and create a coherent data.*

## Customize the Admin

The next step is to add choice for each question.    
The polls app already has two models: Question & Choice, and the Choice uses a foriegn key to the Question model.

Edit polls/admin again:

	you@dev-machine: nano polls/admin
	
	
So it will look like this:

	from django.contrib import admin
	from .models import Choice, Question
	
	
	class ChoiceInline(admin.TabularInline):
	    model = Choice
	    extra = 3
	
	
	class QuestionAdmin(admin.ModelAdmin):
	    fieldsets = [
	        (None,               {'fields': ['question_text']}),
	        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
	    ]
	    inlines = [ChoiceInline]
	
	admin.site.register(Question, QuestionAdmin)	
And run the django development server:

	you@dev-machine: .././manage.py runserver

Browse to the admin at 127.0.0.1:8000, and add a question.


The admin should show a much cleaner form: The question title, a date information details with show/hide, and a table of 3 available choices related to the question.    
Type your question, date, time, and a few choices, then save.   

The edited polls/admin.py defines all these layout options to the admin app:


1. Include the Question, Choice models
2. Show the choices with the table, 3 choices per question (extra=3), in a tabulr form. Django uses the ForiegnKey to match choices with a specific question.
3. Split the question for to fieldsets, one of these is the "Date Information" fieldset that you can collapse.

*Note: If you are going to use the admin a lot, there are many more options to customize it. See the django tutorial about [Customize the admin form](https://docs.djangoproject.com/en/1.8/intro/tutorial02/#customize-the-admin-form) for many more details and examples*
	

We have done with the admin for now, it seems to work fine: 

	you@dev-machine: git status
	
	# On branch polls-app
	# Changes not staged for commit:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   polls/admin.py
	#
	no changes added to commit (use "git add" and/or "git commit -a")
	
Stage and commit:

	you@dev-machine: git add .
	you@dev-machine: git commit -m 'polls admin'
	
	
Great Work!

In the next tutorial, you will add some urls and the functions that let you actually browse the polls app from a browser.    
Continue to [Part 3: Views & URLs](tutorial_part3.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




	
	
		
	
	
