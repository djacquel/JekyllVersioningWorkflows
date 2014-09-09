---
title: Jekyll versioning workflows
layout: page
---

&nbsp;

# Jekyll versioning workflows

You have two options when it comes to host a Jekyll site on Github pages :

 - simple versioning : your code is in one repository branch
 - multi-branch versioning : code and site in two separate branches

## Which versioning workflow suits my needs ?

You just have to ask few questions :

 - do I need plugins to render my site ?
 - do I need extra processing like Gulp task or Rake tasks ?
 - do I need to push in several locations (integration, continuous integration) ?

If the answer to any of this questions is **yes** then you should use multi-branch versioning. Otherwise, stay with the basic versioning.

## 1. Basic workflow

You host your site on Github pages and just have to push your code :

 - in **master branch** for user repository at **https://github.com/userName/userName.github.io** (url : http://userName.github.io) 
 - in **gh-pages branch** for project repositories at **https://github.com/userName/repositoryName** (url : userName.github.io/projectName).

Just need a `git push origin master` or `git push origin gh-pages` and Github will build you site just fine.

This is the simplest versioning worflow.

## 2. Complex workflow

  If you need some plugins (generator, tag, ...) or build tasks (gulp, grunt, ...) they will not run on gh-pages. So, you will have to build you site before pushing the resulting pages on github (or on any server that can serve static pages).

  For a **User/Organization** site, it will be **master branch** for the site build, and **sources branch** (or any name you like) for the code.
  It will be versioned at **github.com/userName/userName.github.io** and hosted at **https://userName.github.io**.

  For a **Project site**, it will be **gh-pages branch** for the site build, and **master branch** (or any name you like) for the code.
  A Project site will be versioned at **github.com/userName/projectName** and hosted at **https://userName.github.io/projectName**.

  Both repository can also have a [custom domain name](https://help.github.com/articles/about-custom-domains-for-github-pages-sites), but it's another story.


# How can I automate this setup and the deploy task ?

Check out the [Rakefile](https://github.com/djacquel/JekyllVersioningWorkflows/blob/master/Rakefile) !

It can do :

 - base install with multi-branch versionig
 - launch build tasks (rake, gulp)
 - deploy code to various locations (local, continuous integration, production)
 - order pizzas

# How does this Rakefile do the base install for me ?

In order to publish :

 - a User/Organization site : source code will be versioned in a **source** branch and site code in **master** branch.
 - a Project site : source code will be versioned in a **source** branch and site code in **gh-pages** branch.

Once you've set the parameters in the head of the Rakefile, you can launch the installation by doing :

 1. create a repository on github (eg: https://github.com/userName/userName.github.io)

 2. in the console do `cd pathTo/yourJekyllSource`

 3. then `wget https://raw.githubusercontent.com/djacquel/JekyllVersioningWorkflows/master/Rakefile` will save the Rakefile in your repository.

 4. edit the Rakefile and set parameters.

 5. in the console do : `rake setup`

 And your done !

## Setup explanation

In the background, and depending if we deal with a User/Organization (**UO**) site or a Project site (**P**), `rake setup` does :

 1. `git init`

 2. `git remote add origin git@github.com:userName/userName.github.io.git` (**UO**) or `git remote add origin git@github.com:userName/repositoryName.git` (**P**)

 3. `jekyll new .` creates your code base

 4. in **_config.yml**, set the baseurl parameter to **baseurl: ''** (**UO**) or **baseurl: '/repositoryName'** (**P**)

 5. in **.gitignore** add **_site**, it will be versioned in the other branch

 6. `jekyll build` will create the destination folder and build site.

 7. `git checkout -b sources` (**UO**) or `git checkout master` (**P**)

 8. `git add -A`

 9. `git commit -m "jekyll base sources"` commit your source code

 10. `git push origin sources` (**UO**) or `git push origin master` (**P**) push your sources in the appropriate branch

 11. `cd _site`

 12. `touch .nojekyll`, this file tells gh-pages that there is no need to build

 13. `git init` init the repository

 14. `git remote add origin git@github.com:userName/userName.github.io.git` (**UO**) or `git remote add origin git@github.com:userName/repositoryName.git` (**P**)

 15. `git checkout master` (**UO**) or `git checkout -b gh-pages` (**P**) put this repository on the appropriate branch

 16. `git add -A`

 17. `git commit -m "jekyll first build"` commit your site code

 18. `git push origin master` (**UO**) or `git push origin gh-pages` (**P**)

# Other tasks included

- rake build : Build Jekyll with production configuration
- rake deploy : Deploy code and site in appropriate branches on Github
- rake dev : Build Jekyll with development configuration
- rake list : list tasks
- rake setup : Do the base setup for a repository with complex workflow
- rake w : Build Jekyll with development configuration and watch

# Contributors welcome

Feel free to [fork my repository](https://github.com/djacquel/JekyllVersioningWorkflows) if you want to enrich the code.
