
# Jekyll workflows

You have two options when it comes to host a Jekyll site on Github pages :

## 1. Basic workflow

If you don't need to run plugins or special tasks (grunt, gulp, ...) in order to get your site running.

You can keep the default configuration and simply push your sources to **master branch**  for user repository (userName.github.io) or to **gh-pages branch** for project repositories (userName.github.io/projectName).

Just need a `git push origin master` or `git push origin gh-pages` and Github will build you site just fine.

This is the simplest worflow.

## 2. Complex workflow

  If you need some plugins (generator, tag, ...) or build tasks (gulp, grunt, ...) that will not run on gh-pages.

  You will have to publish your Jekyll sources in a branch and your build result in a separate branch.

  For a User / Organization site, it will be **master** for the site build, and **sources** (or any name you like) for the code.
  A User / Organization site will be versioned at github.com/userName/userName.github.io and hosted at http://userName.github.io.

  For a Project site, it will be **gh-pages** for the site build, and **master** (or any name you like) for the code.
  A Project site will be versioned at github.com/userName/projectName and hosted at http://userName.github.io/projectName.

  Both repository can also have a [custom domain name](https://help.github.com/articles/about-custom-domains-for-github-pages-sites), but it's another story.

### User or organization repository

In order to publish a User/Organization site, source code will be versioned in a **source** branch and site code in **master** branch.

Complete setup will be :

 1. create a repository on github (eg: https://github.com/userName/userName.github.io)

 2. go to command line and `cd pathTo/yourJekyllSource`

 3. `git init && git remote add origin git@github.com:userName/userName.github.io.git`

 4. `jekyll new .` creates your code base

 5. in **_config.yml**, set the baseurl parameter to **baseurl: ''**

 6. in **.gitignore** add **_site**, it will be versioned in the other branch

 7. `jekyll build` will create the destination folder and build site.

 8. `git checkout -b sources`

 9. `git add * && git commit -m "jekyll base sources"` commit your source code

 10. `git push origin sources` push your sources in the **sources** branch

 11. `cd site`

 12. `touch .nojekyll`, this file tells gh-pages that there is no need to build

 13. `git init && git remote add origin git@github.com:userName/userName.github.io.git`

 14. `git checkout master`

 15. `git add * && git commit -m "jekyll first build"` commit your site code

 16. `git push origin master`


### Project repository

This is the way I do it for a project repository (a site living at **http://userName.github.io/repositoryName**) :

 1. create a repository on github (eg: https://github.com/userName/repositoryName)

 2. go to command line and `cd pathTo/yourJekyllSource`

 3. `git init && git remote add origin git@github.com:userName/repositoryName.git`

 4. `jekyll new .` creates your code base

 5. in **_config.yml**, set the baseurl parameter to **baseurl: '/repositoryName'**

 6. in **.gitignore** add **_site**, it will be versioned in the other branch

 7. `jekyll build` will create the destination folder and build site.

 8. `git checkout master`

 9. `git add * && git commit -m "jekyll base sources"` commit your source code

 10. `git push origin master` push your sources in the **master** branch

 11. `cd _site`

 12. `touch .nojekyll`, this file tells gh-pages that there is no need to build

 13. `git init && git remote add origin git@github.com:userName/repositoryName.git`

 14. `git checkout -b gh-pages`

 15. `git add * && git commit -m "jekyll first build"` commit your site code

 16. `git push origin gh-pages`

And your good to go !


## Automating

Check to Rakefile !

It can do :

 - base install with the described workflow
 - deploy code
 - build site
 - build site for development environment
 - order pizzas
