---
layout: post
title: Jeykll Blog Hosted by Bitbucket
---

After recently discovering that you can host a static site [on bitbucket](https://confluence.atlassian.com/display/BITBUCKET/Publishing+a+Website+on+Bitbucket) I decided to use it to host a [Jeykll](http://jekyllrb.com/) blog. This is a common use of [Github pages](https://pages.github.com/) but unfortunately we dont get the same behind the scenes magic to build Jekyll when using Bitbucket.
While there are a couple of old guides out there on google, I thought I would share my experiences of using a continuous deployment tool
[Codeship](https://codeship.com/projects) to get it working.

Hosting a site with Bitbucket
-----------------------------

If you make a repository called 'myusername.bitbucket.org', Bitbucket will host that repo using the repo name as the url - like this blog. It treats the repository root as the web root, which is where we run into problems with Jekyll. When you build your Jekyll project it does its stuff and produces your static site ready to be served inside the _site directory. _site is not the root directory, so Bitbucket will not serve your Jekyll site out of the box. The easy solution is to simply use two repositories, one that holds your Jekyll project and a second that holds a copy of the contents of the _site directory. While you could manually upload the files every time you update your site this seems like a process ripe for automation.

Automating the Deployment
-------------------------

There are any number of tools that could be used to automate this process but I decided to use [Codeship](https://codeship.com/projects) for two reasons, you can use it for free and the Bitbucket integration is painless. If you sign up for Codeship using your Bitbucket account you are greeted with an on-boarding flow where you select a repository and then configure a build for it. Our build is about as simple and they come, we just want to build the site with Jekyll, so we want to setup our build commands as follows

>bundle install
>jekyll build

At this point we had better go and add a Gemfile to the root of our Jekyll directory, if it has not already been created, so that Codeship is able to get the dependencies required to build the project. In my case I simply created a Gemfile in the repo root that contains the following

>source 'https://rubygems.org'
>gem 'jekyll' 

Now Codeship knows what dependencies we have and where to fetch them from so it can successfully build the blog. 
In addition when we imported the repo from Bitbucket Codeship configured a Post commit web-hook, so this build will get run whenever we commit to this repository.

Deploy _site to the second repository
-------------------------------------

Now that we have a successful build we need to add a deployment pipeline so that we can push the output of the build to the second repo. Since this can be accomplished using just Git commands we do not want to use one of the prebaked deployment recipes, instead we just want to use a custom script. A working deployment script will look something like this

>cd _site
>git init
>git remote add origin git@bitbucket.org:myAccountName/myAccountName.bitbucket.org.git
>git config --global user.email "robot@codeship.com"
>git config --global user.name "Codeship Deployment Script"
>git add .
>git commit -m "deploy from codeship"
>git push -f origin master

We change into the _site directory as initialize it as a git repository. Next we add a new remote called origin which is our Bitbucket repo that is set up for the static hosting. We next configure the details for the user that will be committing the changes. After this we stage all of the changes in the _site directory, commit them, and then force push them to origin.

It is worth a quick note here to say that you probably should not be using git push -f without understanding what you doing.  Normally a repository will refuse a push if we are updating a branch and missing commits are found in the remote. The flag skip that check and forces the remote to accept out commit. This means that what you have locally will be what ends up on the remote, so you can **permanently** lose work with a misplaced force push. In this instance it is safe as the contents of the repo we are pushing to is  always the output of a Jekyll build, so we can always rebuild an old commit if we want to rollback something on the blog.  

One last catch - SSH Keys
--------------------
The last thing required is to configure the SSH key used by CodeShip when accessing your Bitbucket repo, so we need to go into the repository setting for the repo that contains our Jekyll files. When we set up Codeship it automatically configured a deployment key for that repository, this is a key with read only permissions. As we are wanting Codeship to do more that just clone our repo we need to remove this deployment key and instead add it as an SSH key so that is has read and write permissions. So we want to copy the key, delete the deployment key, then go into manage account and then SSH keys. We now add the public key we copied so that codeship is able to push to our repositories. 

Once again a PSA is in order, we are giving Codeship full SSH access to our Bitbucket account so you may not want to do this if it contains anything sensitive, if you are security conscious you can simply create a second account to push your blog to. 

That is it. When you commit to your blog repository Codeship will now checkout your repo, run Jekyll build and then push the contents of _site to your magic Bitbucket repository where it will get served.

I hope I was able to explain how to get things working without too much effort, I had bit of pain working out the easiest way of getting Jeykll working nicely Bitbucket. If you found this post helpful I would love to hear from you in the comments :) 


