# Django Tutorial Part 4: First Deployment

This is part 4 of the tutorial.    
Make sure you finished [Part 3](tutorial_part3.md)

**Quick reminder**: 

In parts 1-3 you created the polls app, added initial data with the admin, and the urls & views required to browse the polls app from a browser.

In this part, you will deploy, and see how it works on a real website.

## Test Localy with Apache/Nginx

So far we have tested the site with django development server. It's a great and easy way to work on the site when you code, but before deployment, you want to test that everything works on Apache/Nginx.

Restart Apache and Nginx, according to your dev environment.

OSX:

	you@dev-machine-osx: sudo apachectl restart
	you@dev-machine-osx: sudo nginx
	
*Note: if nginx is already running, use sudo nginx -s reload*
	
Ubuntu desktop:

	you@dev-machine-ubuntu: sudo service apache2 restart
	you@dev-machine-ubuntu: sudo service nginx restart
	
		
	
*Note:If you use the local apache and Nginx often, you don't have to restart. To reload the new django code to mod_wsgi just touch the `site_repo/wsgi.py` file. If Nginx configs didn't change there is no need to reload or restart it.    
The alias to reload the django code is: `$ site-reload`*

Browse to the local site at **127.0.0.1**.
This address  does **not** use :8000, which is used by the django development server.

Browse to the site:

Browse To | You Should See
----------| --------------
127.0.0.1| The "Hello World!!!" page
127.0.0.1/admin | The django admin
127.0.0.1/polls |  A list of the questions you entered in the admin
127.0.0.1/polls/42/ |  You're looking at question 42

OK, it works on the local Apache/Nginx, and the code is ready for deployment (in a more advanced project you will also run the test suite before deployment).

Current status:

	you@dev-machine: git-status
	
	# On branch polls-app
	nothing to commit (working directory clean)
	
Deployment is alway on master, so checkout:

	you@dev-machine: git checkout master
	
	Switched to branch 'master'
	
Merge the polls-app branch:

	 you@dev-machine: git merge polls-app
	 
	 Updating 5cb9549..172af59
	 Fast-forward
	 polls/admin.py                   |   17 +++++++++++++++++
	 polls/migrations/0001_initial.py |   34 ++++++++++++++++++++++++++++++++++
	 polls/models.py                  |   13 +++++++++++++
	 polls/tests.py                   |    3 +++
	 polls/urls.py                    |   14 ++++++++++++++
	 polls/views.py                   |   22 ++++++++++++++++++++++
	 settings.py                      |    3 ++-
	 urls.py                          |    1 +
	 8 files changed, 106 insertions(+), 1 deletion(-)
	 create mode 100644 polls/__init__.py
	 create mode 100644 polls/admin.py
	 create mode 100644 polls/migrations/0001_initial.py
	 create mode 100644 polls/migrations/__init__.py
	 create mode 100644 polls/models.py
	 create mode 100644 polls/tests.py
	 create mode 100644 polls/urls.py
	 create mode 100644 polls/views.py
	 
	 
Works.    
Check the master log:

	you@dev-machine: git log --oneline
	
	172af59 polls base views and urls
	9b678e8 polls admin
	09d6541 polls models and migrations
	7cdcb9a added polls app
	6d5d84c the color is blue
	5a04cb6 home page welcome message
	00bfc74 init site repository
	
All the polls-app commits were merged successfuly to the master.    
Status:

	you@dev-machine: git status
	
	# On branch master
	# Your branch is ahead of 'origin/master' by 4 commits.
	#
	nothing to commit (working directory clean)
	
After the merge, the local repository is ahead of the main project repository.    
Push the updates:

	you@dev-machine: git push
	
	Counting objects: 15, done.
	Compressing objects: 100% (10/10), done.
	Writing objects: 100% (10/10), 1.45 KiB, done.
	Total 10 (delta 5), reused 0 (delta 0)
	To django@45.33.93.118:/home/django/site_repo.git
	   5cb9549..172af59  master -> master
	   09d6541..172af59  polls-app -> polls-app
		
	
The main repository is up to date.

Reminder: the production website uses a separate repository, so the push to the main repo does not affect the website, it only saves the progress of your work.

**Interim summary:**

1. The polls app is working localy both on the django development server, and the local Nginx/Apache. 
2. The polls-app branch was merged to master
3. The main project repository is up to date


## Deployment

Looking at the interim summary, it seems that everything is ready for deployment:

	you@dev-machine: git status
	
	# On branch master
	nothing to commit (working directory clean)


Since we changed only Python code, a simple deployment will do:

	you@dev-machine: fab deploy
	
	[django@45.33.93.118] Executing task 'deploy'
	[localhost] local: git --git-dir=/home/aviah/my-django-projects/my-site/site_repo/.git push production master
	Counting objects: 24, done.
	Compressing objects: 100% (21/21), done.
	Writing objects: 100% (21/21), 3.03 KiB, done.
	Total 21 (delta 8), reused 0 (delta 0)
	To django@45.33.93.118:/home/django/my-site/site_repo/
	   5cb9549..172af59  master -> master
	[django@45.33.93.118] run: git reset --hard
	[django@45.33.93.118] out: HEAD is now at 172af59 polls base views and urls
	
	[django@45.33.93.118] run: mv secrets.py secrets.txt
	[django@45.33.93.118] run: rm *.py
	[django@45.33.93.118] run: mv secrets.txt secrets.py
	[django@45.33.93.118] run: touch __init__.py
	[django@45.33.93.118] run: cp ~/my_site/site_repo/settings_production.py ~/my_site/site_config/
	[django@45.33.93.118] run: touch ~/my_site/site_repo/wsgi.py
	
	Done.
	Disconnecting from django@45.33.93.118... done. 


The fabric log tells us that:

1. Master was pushed to production
2. Production working directory updated with `git reset --hard`
3. The `settings_production.py` was re-copied to site_config
4. Mod_wsgi should reload the new code, after the `touch wsgi.py`

Reminder: `settings_production.py` is maintained **in the repo**, but the actual file loaded to mod_wsgi is the one in the `site_config` directory, **outside** the repo. So you keep all the settings files in the repo, and use only one of them in each environment. Fabric also keeps the secrets.py file. See the project reference: [Production & Development Settings](https://github.com/Aviah/one-click-django-docs/blob/master/project_ref.md#production--development-settings).

Deployment seems to work. Time to check the website. 
Go to the website with your browser, at yourdomain.com.   
Browse to the new polls app, at `www.yourdomain.com/polls`

Ooops! 500 Serrver Error?    
That's annoying. Everything worked perfectly well localy!    
What happened?


## Basic Troubleshooting

You have to check the server:

	you@dev-machine: ssh IP.IP.IP.IP
	
	
And take a look at the logs:

	you@my-django-server: tail-logs
	
You should now see the tail of the following logs:

1. The django `main.log`, that logs everything you log in code with `logging.getLogger('main').info` or higher logging level
2. The django `debug.log`, that logs everything you log in code with `logging.debug`, and only in dev environment
3. The Apache error log
4. The Nginx error log


*Note: tail-logs will only work if you ssh with your user, because it requires sudo to Apache and Nginx error logs. The django user on the server does not have sudo permissions. For more details about logging see [Coding Reference](https://github.com/Aviah/one-click-django-docs/blob/master/coding_ref.md)*


Look closely at `main.log`. There is a line that says:

	ProgrammingError: (1146, "Table 'django_db.polls_question' doesn't exist")
	
OK, everything is clear now. The database table for the polls app is missing.    
We deployed the code - but didn't run migrations!    

Reminder: migrations tell django **how** to apply the models' changes to the database, but we still have to explicitly **run** these migrations.    
It worked localy, because we applied the migrations file to the local database. On the server, we just pushed the migrations file, but didn't run it. The database schema is still as it was before we added the polls app.
    
The solution is **deployment with migrations**.    

Exit the server, back to the local machine.   
Before any migration, it's' better to backup the current database:

	you@dev-machine: fab backup
	
	[django@45.33.93.118] Executing task 'backup'
	[django@45.33.93.118] run: python manage.py dumpdata > db_backup/2016-01-22--01-40-12
	[django@45.33.93.118] out: CommandError: Unable to serialize database: (1146, "Table 'django_db.polls_question' doesn't exist")
	
	
	Fatal error: run() encountered an error (return code 1) while executing 'python manage.py dumpdata > db_backup/2016-01-22--01-40-12'

	Aborting.
	
Oh, shoot! Django can't complete the dumpdata, it requires the polls models (it actually did backed up what's available).

One solution would be to go to the server, git checkout a previous commit, run manage.py dumpdata, and then checkout master again (when manage.py runs dumpdata, it will use the previous commit).   

However, a simpler solution is to dump with MySQL. This will dump everything as-is, in SQL.    
MySQL obviously doesn't care about the python code, and doesn't check the models on the django side. So it will backup everything as is.

SSH  to the server again:

	you@dev-machine: ssh django@PUB.IP.IP.IP
	
And dump the data

	you@my-django-server: cd mysite/db_backup/
	you@my-django-server: mysqldump -u django -p --databases django_db --add-drop-database > before_polls_mig0001.sql
	
	
OK, now we are really ready to deploy. Exit the server.


## Deployment with Migrations

It's safer to run database changes when the site is  in maintenance:


	you@dev-machine: fab site_maintenance
	
	[aviah@45.33.93.118] Executing task 'site_maintenance'
	[aviah@45.33.93.118] run: sudo /usr/local/bin/./site-maintenance.sh
	[aviah@45.33.93.118] out: [sudo] password for aviah: 
	[aviah@45.33.93.118] out: Site will go maintenance in 1 minute
	[aviah@45.33.93.118] out:  * Restarting nginx nginx
	[aviah@45.33.93.118] out:    ...done.
	
	Done.
	Disconnecting from 45.33.93.118... done.

	
Check:

	you@dev-machine: curl www.yourdomain.com
	
	<!DOCTYPE html>
	<html>	
	    <head>
	    <title> Maintenance </title>
	    </head>	
	    <body>	    
	        <h1> Maintenance </h1>	    
	    </body>	
	</html>    
	
	
The site is in maintenance, Nginx just returns a static page.

Finally, the migration:

	you@dev-machine: fab migrate
	
	[django@45.33.93.118] Executing task 'migrate'
	[django@45.33.93.118] run: python ~/my-site/manage.py migrate
	[django@45.33.93.118] out: Operations to perform:
	[django@45.33.93.118] out:   Synchronize unmigrated apps: staticfiles, messages
	[django@45.33.93.118] out:   Apply all migrations: admin, contenttypes, polls, auth, sessions
	[django@45.33.93.118] out: Synchronizing apps without migrations:
	[django@45.33.93.118] out:   Creating tables...
	[django@45.33.93.118] out:     Running deferred SQL...
	[django@45.33.93.118] out:   Installing custom SQL...
	[django@45.33.93.118] out: Running migrations:
	[django@45.33.93.118] out:   Rendering model states... DONE
	[django@45.33.93.118] out:   Applying polls.0001_initial... OK
	
	
	Done.
	Disconnecting from django@45.33.93.118... done.

Now it worked. Django applied the `polls.0001_initial` migration to the databse.    
You can SSH to the server, login to MySQL, and verify that the new polls tables are already in the django_db database.

Check the site at `www.yourdomain.com/polls`.   
Ahmmm…Page not found? 404?    
Ah, the site is still in maintenance.
Restart:

	you@dev-machine: fab site_up
	

Check again at `www.yourdomain.com/polls`    
Good news, the 500 error is gone.
But there is nothing else either.
   
Obviously there is nothing to see, since there are no questions in the **server's** database - yet.   
Go to the admin, on the server, at `www.yourdomain.com/admin`.    
Login, and add some questions, similarly to how you entered questions localy.

*Note: use the django superuser's username & passowrd that you provided when installing one-click-django-server.*

After adding a question or two, go to `www.yourdomain.com/polls`.    
And here are your questions.

Great work!

You deployed a migration, and handled some issues with server troubleshooting.
	
Now that everything works, there must be a way to make these pages look a bit better? a bit more useful?    
Continue to [Part 5: Templates](tutorial_part5.md)


Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|



	











	
