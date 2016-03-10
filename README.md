# Django Tutorial with Deployment

A quick intro

## The Tutorial

This tutorial implements most of the django official polls tutorial, with one significant addtions: deployment. You will get a real feel how a django project gets a progress with git, deployment to a working server, and building a real website. 

When you finish, the polls app will run on your own production server!


## Before you start: Django in a nutshell

Django may seem complex at times. It's a mature framework, it has many many features, and a lot to learn.    
The good news are that if you need it to do something,  chances are that somebody already figured it out it before, and  the feature probably exists in django or a separate library.

However, remember that in a nutshell, django is quite simple.    
This is the crux of what django does:

1. Gets a URL request from the web server
2. Finds the correct Python function to run for that specific URL
3. Calls this function
4. Takes what this function returns, and sends it to the web server.

That's it.

So your job as a django developer, in a nutshell, is to write python functions that respond to URLs (in django these functions are called "views"), and hook these functions to specific URLs. It's not that hard!


## Tutorial Ref

The following is the implemntation of the django official tutorial in the one-click-django environment, with deployment.

Start with the [Playground](playground.md) , which let you experiment the environment.
   
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



