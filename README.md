# Django Tutorial with Deployment

**Django tutorial with git & deployment to a real website, based on the official polls tutorial**

## The Tutorial

This tutorial implements most of the django official polls tutorial, with one significant addtion: deployment. You will get a real feel how to build a a django and deploy it, using git and fabric.

When you finish, the polls app will run on your own production server, at `www.yourdomain.com/polls`.

To run this tutorial, you will need the one-click-django-server and the matching development local website. If you haven't installed it already, it's easy, the setup scripts auto-install (almost) everything:    

* Production website:[ Django site on Ubuntu 14.04 Server](https://github.com/Aviah/one-click-django-server)
* Develop & Deploy:  on [OSX El-Capitan](https://github.com/Aviah/one-click-django-dev-osx-el-capitan), or [Ubuntu 14.04 Trusty desktop](https://github.com/Aviah/one-click-django-dev-ubuntu-14-04-trusty)


*A note about the IDE: This is obviously a personal choice, what you feel that works for you. If you haven't selected an IDE yet, I recommend Wing IDE Pro which I use for years. It has the "Python Zen", just fits to the Python programmer, integrates with django, git, excellent debugging and great support* 



## Before you start: Django in a nutshell

Django may seem complex at times. It's a mature framework, has many features, and a lot to learn.    
The good news are that if you want it to do something,  chances are that somebody already figured it out it before, and  the feature you need probably exists in django or in a separate library.

However, in a nutshell, django is quite simple.    
This is the crux of what django does:

1. Gets a a URL with a request from the web server
2. Finds the correct Python function to run for that specific URL
3. Calls this function
4. Takes what this function returns, and sends it back to the web server.

That's it.

So your job as a django developer is to write python functions that respond to URLs (in django these functions are called "views"), and hook these functions to specific URLs. It's not that hard!


## Table of Contents

Start with the [Playground](playground.md) , which lets you play and experiment the environment.

[Part 1: Create the Polls App](tutorial_part1.md)    
[Part 2: The Django Admin](tutorial_part2.md)    
[Part 3: Views & URLs](tutorial_part3.md)    
[Part 4: First Deployment](tutorial_part4.md)    
[Part 5: Templates](tutorial_part5.md)    
[Part 6: Forms](tutorial_part6.md)    
[Part 7: A Brief Overview of Django's Base Components](tutorial_part6.md)    
[Part 8: What Next and some Resources](tutorial_part8.md)


Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|



