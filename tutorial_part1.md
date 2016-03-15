# Django Tutorial Part 1: Create the Polls App


This is the 1st part of the tutorial. Before you start make sure you read the [intro](README.md)

## Start the Django Polls Tutorial

When you add new code, like new apps, features, bugfixes etc, it's better to use a separate branch, and keep the master branch stable. When the master branch is stable, deployment is just pushing it to production. As long as the master is stable, you can experiment and develop freely on other branches, and safely deploy with the master branch.

So, to add the polls app, start with a new branch.

**Make sure you are in the `site_repo` directory:**

	you@dev-machine: git branch polls-app
	you@dev-machine: git checkout polls-app
	
	
Create the polls app:

	you@dev-machine: django-admin startapp polls
	
What happened?

	you@dev-machine: git status
	
	# On branch polls-app
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	polls/
	nothing added to commit but untracked files present (use "git add" to track)track)	
	
Django created the app directory, polls.
The directory contains several files, based on the default app template:

	you@dev-machine: ls polls
	admin.py  __init__.py  migrations  models.py  tests.py  views.py
		
Add the files, and commit:

	you@dev-machine: git add .
	you@dev-machine: git commit -m 'added polls app'
	
Now we have to tell django to use the polls app. It's not enough to create the app directory, we also have to declate it in the `settings.py` file:

	you@dev-machine: nano settings.py
	
Then look for the `INSTALLED_APPS` key, and edit, so it will look like this:

	INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'site_repo.home',
    'site_repo.polls'
    )
   
The basic app files are in place, we can start to write the app.   
Edit the app models:

	you@dev-machine: nano polls/models.py
	
After the edit, the file should look like this:

	from django.db import models

	class Question(models.Model):
   	    question_text = models.CharField(max_length=200)
   	    pub_date = models.DateTimeField('date published')

	class Choice(models.Model):
   		question = models.ForeignKey(Question)
    	choice_text = models.CharField(max_length=200)
    	votes = models.IntegerField(default=0)


A model is a python class that django uses to access the database table. The django "models" are an abstraction layer, of the underlying database tables, with additional functionality (provided by django). When you code, you interact with models, and then django interact with the database behind the scenes.

Since the app's models are **not** database tables, once you define the model class, django needs to create the actual database table. But first, we tell django that we have updated the models. 
   
In the `site_repo` directory:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py makemigrations polls
	
	Migrations for 'polls':
	0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
    
    
See what happened:
	
	you@dev-machine: cd site_repo
	you@dev-machine: git status
	
	
	# On branch polls-app
	# Changes not staged for commit:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   polls/models.py
	#	modified:   settings.py
	#
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	polls/migrations/0001_initial.py

	
Note the changes of two tracked files: `settings.py`, and the `polls/models.py`.

And there is a new file: `polls/migrations/0001_initial.py`. 
Django added a migrations file, with specific instructions how to change the database to match the models. 

It's time to tell django to apply these changes to the database. Django handles the SQL schema updates to the database:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py migrate
	
	Operations to perform:
	  Synchronize unmigrated apps: staticfiles, messages
	  Apply all migrations: admin, contenttypes, polls, auth, sessions
	Synchronizing apps without migrations:
	  Creating tables...
	    Running deferred SQL...
	  Installing custom SQL...
	Running migrations:
	  Rendering model states... DONE
	  Applying polls.0001_initial... OK
	  
OK! Django just applied the `0001_initial` migration to the database.    
Check it:

	you@dev-machine: mysql -uroot -p
	mysql > use django_db;
	
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A

	Database changed
	
	mysql> show_tables;
	
	+----------------------------+
	| Tables_in_django_db        |
	+----------------------------+
	| auth_group                 |
	| auth_group_permissions     |
	| auth_permission            |
	| auth_user                  |
	| auth_user_groups           |
	| auth_user_user_permissions |
	| django_admin_log           |
	| django_content_type        |
	| django_migrations          |
	| django_session             |
	| polls_choice               |
	| polls_question             |
	+----------------------------+
	12 rows in set (0.00 sec)

The database contains (for now) 12 tables. 10 tables are used by django, and the last two tables are the new polls app tables.

By default, django names the tables as "app-name_model-name".    
To see the tables' fields:


	mysql > describe polls_question;
	
	+---------------+--------------+------+-----+---------+----------------+
	| Field         | Type         | Null | Key | Default | Extra          |
	+---------------+--------------+------+-----+---------+----------------+
	| id            | int(11)      | NO   | PRI | NULL    | auto_increment |
	| question_text | varchar(200) | NO   |     | NULL    |                |
	| pub_date      | datetime     | NO   |     | NULL    |                |
	+---------------+--------------+------+-----+---------+----------------+
	3 rows in set (0.00 sec)
	

	mysql > describe polls_choice;
	
	+-------------+--------------+------+-----+---------+----------------+
	| Field       | Type         | Null | Key | Default | Extra          |
	+-------------+--------------+------+-----+---------+----------------+
	| id          | int(11)      | NO   | PRI | NULL    | auto_increment |
	| choice_text | varchar(200) | NO   |     | NULL    |                |
	| votes       | int(11)      | NO   |     | NULL    |                |
	| question_id | int(11)      | NO   | MUL | NULL    |                |
	+-------------+--------------+------+-----+---------+----------------+
	4 rows in set (0.02 sec)
	
	


The django migration converted the models' definitions in python to SQL, and created the actual database tables.    
By default, django adds an "id" auto_increment field to each table.

*Note: The question_id field in the polls_choices table is a MUL index, a multiple index, which django will use for the "many" side of the one-to-many relation question-choices (in the `polls/models.py` file, there is a `ForeignKey` field for the choices model)*


Most of the time you will work with django models, but once in a while you will need to use direct sql to run some complex queries or improve preformance. For now, exit MySQL.


**Interim Summary:**

1. You created the polls app
2. You added the polls app to the site `INSTALLED_APPS`, in `settings.py`
3. You added polls models
4.  You created a migration
5.  You applied the migration to the **local** database


Seems like a progress, so it's a good time to commit.    
From the `site_repo` directory:

	you@dev-machine: git add .
	you@dev-machine: git status
	# On branch polls-app

	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   polls/migrations/0001_initial.py
	#	modified:   polls/models.py
	#	modified:   settings.py
	#
	
All the changes are staged.    

Commit:

	you@dev-machine: git commit -m "polls models and migrations"
	
And push:

	you@dev-machine:  git push origin polls-app
	
	Counting objects: 13, done.
	Compressing objects: 100% (11/11), done.
	Writing objects: 100% (11/11), 1.69 KiB, done.
	Total 11 (delta 3), reused 0 (delta 0)
	To django@45.33.93.118:/home/django/site_repo.git
	 * [new branch]      polls-app -> polls-app
	 
	 
Everything so far is on the polls-app branch. The master branch didn't change at all. We will merge the polls-app to master only when it's tested, stable, and ready for deployment.

Great Work!

In the next tutorial, you will enter some manual data with the django admin.    
Continue to [Part 2: The Django Admin](tutorial_part2.md)


Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|


		

	

	



	
	
	


