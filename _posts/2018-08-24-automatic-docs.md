---
layout: post
title:  "Automatic online documentation for GitHub projects using Doxygen, Travis CI and GitHub pages"
date:   2018-08-24 16:47:00 +0200
categories: ci documentation continuous-integration github makefile
---
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
For a couple of weeks I've been putting my evenings into developing a neural network framework in C++.
As the project has grown I've started thinking about how useful it would be to have an accessible documentation at hand.
This immediately got me thinking about doxygen, a really nice tool for generating documentation in multiple different formats directly from comments in source code.
By combining this with the Continuous Integration tool Travis CI and web hosting from GitHub pages I set up a system that keeps an online documentation up to date with the master branch of the project.
This setup can quite easily be expanded and generalised to any kind of project hosted on GitHub.

The main idea behind the system is:
- We have a set of source files with descriptive comments
- html documentation for these is generated through Doxygen
- Travis CI runs Doxygen on every push
- Documentation is pushed to separate repository/branch
- Repository/branch is served using GitHub pages

An example of this setup can be found in my [NNetCpp](https://github.com/joelnir/NNetCpp) project, with the documentation pushed to the [NNetDocs repository](https://github.com/joelnir/NNetDocs).
This can then be reached through a [GitHub pages website](https://joelnir.github.io/NNetDocs/).

### Source files
Although this setup will help with an easy deployment of the documentation, the quality of the documentation is fully dependent on writing informative comments.
Doxygen allows for many different ways to comment your code.
Everything from just a short sentence describing a function to entire constructs using keywords like \param and \return are supported.
Take a look at the [Doxygen documentation](https://www.stack.nl/~dimitri/doxygen/manual/docblocks.html) for how to comment your code to allow for generating documentation.

### Doxygen
Doxygen is easy to configure and run.
Downloads can be found on the [website](https://www.stack.nl/~dimitri/doxygen/download.html).
On Debian-based linux distributions it can simply be installed with

{% highlight Bash %}
sudo apt-get install doxygen
{% endhighlight %}

To use Doxygen with a project the only thing needed is a configuration file.
In the project folder, run

{% highlight Bash %}
doxygen -g <config-file-name>
{% endhighlight %}

to generate a default one.
This is a pretty big file and might seem frightening at first, but luckily we don't have to change that much in it.
It is also very extensively documented, making it easy to understand the different options.

This is a good time to consider the directory structure.
This is the structure I've been using for my project.

{% highlight Text %}

ProjectFolder/
    +- doxygen.conf             Doxygen config file
    +- Makefile                 Makefile for project
    +- src/                     Source file directory
    +- documentation/           Documentation directory
        +- docs/                Directory for html documentation
        +- (Documentation.pdf)  (pdf documentation, if generated)

{% endhighlight %}

With this setup the documentation is kept in a completely separate directory.
This is beneficial to keep our project top folder clean.
It is also the `documentation` directory that we eventually want to serve using GitHub pages.
Other project structures can of course also be used by adjusting some of the paths in upcoming steps.

We need to adjust some settings in our Doxygen config.
These options are spread out throughout the file, so the easiest way to find them is to search for them.

{% highlight Text %}
PROJECT_NAME           = "My Cool Project"
{% endhighlight %}
The name of the project. Used for headers in the documentation.

{% highlight Text %}
PROJECT_BRIEF          = "A short description of my project"
{% endhighlight %}
Brief description of project. Used as subheader in documentation.

{% highlight Text %}
OUTPUT_DIRECTORY       = "documentation"
{% endhighlight %}
Directory to output all documentation into. Adjust this if you want a different directory structure.

{% highlight Text %}
INPUT                  = src
{% endhighlight %}
Directories containing source files and direct paths to source files.
All files to be included in the documentation needs to included here.

{% highlight Text %}
RECURSIVE              = YES
{% endhighlight %}
If you have subdirectories in your source folders that you want to be included in the documentation set this to YES.

{% highlight Text %}
GENERATE_HTML          = YES
{% endhighlight %}
This tells Doxygen to create a html documentation. (YES is already default)

{% highlight Text %}
HTML_OUTPUT            = docs
{% endhighlight %}
Output folder of HTML documentation. The name `docs` later allows us to serve this folder specifically through GitHub pages.

{% highlight Text %}
GENERATE_LATEX         = NO
{% endhighlight %}
For HTML documentation we don't need this.
See the chapter further down on how to extend this setup to also create a PDF documentation document.

{% highlight Text %}
HAVE_DOT               = NO
{% endhighlight %}
Doxygen uses the dot tool to generate class diagrams.
If you have it installed you can set this to YES.
I did not feel like installing it, so I set it to NO to generate more simple diagrams.

There are plenty more options in the Doxygen config file that can be used to customize the documentation.
Most can be understood by just reading the comments above them. For more in depth information see the [Doxygen documentation](https://www.stack.nl/~dimitri/doxygen/manual/index.html).

To finally generate the documentation we simply run

{% highlight Bash %}
doxygen doxygen.conf
{% endhighlight %}
`doxygen.conf`is the name of the Doxygen config file.
This should create html documentation in the `documentation/html` directory.
To look at it locally simply open `documentation/html/index.html` in a browser.

Since I have been using a Makefile for the rest of my project I decided to add the generation of documentation as a command in it:
{% highlight Makefile %}
.PHONY: docs

docs:
	doxygen doxygen.conf

{% endhighlight %}
With this we can simply run `make docs` to generate the documentation.

### Travis CI
Travis CI is a Continuous Integration tool that can be used freely with open source projects on GitHub.
It is mainly used for testing and deploying builds, but can be set up to do almost anything after a push.
In this setup we will use it to generate the documentation and then push it to a separate repository/branch.

I prefer having a completely separate GitHub repository for the documentation.
This allows for better separation between code and docs.
Another benefit is that since the name in the GitHub pages URL is the same as the name of the repository we can then choose it freely on the documentation repository.
You might prefer to deploy the documentation to the same project repository on the gh-pages branch.
This can easily be achieved by changing some values in the Travis configuration.
Note however that you then have to make sure that the html documentation is in the root folder for GitHub to serve it correctly.

To use Travis with your project you need to authorize it for use with your project repository.
Follow the [Getting Started](https://docs.travis-ci.com/user/getting-started/) page of the Travis CI documentation to get set up.

Travis is configured through a `.travis.yml` file in the repository.
Travis runs on a virtual machine that pulls the repository and then runs multiple commands.
In this file we need to prepare this virtual machine, generate the documentation and deploy it to our decided destination.

The entire `.travis.yml` file looks like this

{% highlight YAML %}
language: cpp

branches:
    only:
        - master

sudo: enabled

before_install:
      - sudo apt-get update
      - sudo apt-get install -y doxygen

script:
    - make docs

deploy:
    provider: pages
    skip-cleanup: true
    github-token: $GITHUB_TOKEN
    keep-history: true
    on:
        branch: master
    local-dir: documentation
    repo: <github-username>/<documentation-repository>
    target-branch: master
{% endhighlight %}

Here's what the different commands mean:

{% highlight YAML %}
language: cpp
{% endhighlight %}
The programming language used in the project. Here it is C++.

{% highlight YAML %}
branches:
    only:
        - master
{% endhighlight %}
Only deploy documentation on pushes to master branch. Change this to fit with your branch workflow.

{% highlight YAML %}
sudo: enabled
{% endhighlight %}
Allow us to use sudo for installing packages.

{% highlight YAML %}
before_install:
      - sudo apt-get update
      - sudo apt-get install -y doxygen
{% endhighlight %}
This is where we customize the virtual machine to support our scripts.
Update package infrastructure and install Doxygen.
`-y` is for automatically answering yes to the install prompt.

{% highlight YAML %}
script:
    - make docs
{% endhighlight %}
Our script.
Since we put the command in the Makefile it is enough to run `make docs`.
If not using a Makefile you could easily put it in a bash-script or simply type out `doxygen doxygen.conf` in the `.travis.yml`.

{% highlight YAML %}
deploy:
    provider: pages
{% endhighlight %}
This tells Travis that we want to deploy using GitHub pages.
Travis has built in support for this, but it requires some setup from our side.
For more information about deploying to GitHub pages see the [Travis docs](https://docs.travis-ci.com/user/deployment/pages/).

{% highlight YAML %}
    skip-cleanup: true
{% endhighlight %}
Do not remove the files that we want to upload.

{% highlight YAML %}
    github-token: $GITHUB_TOKEN
{% endhighlight %}
In order to allow Travis to commit to our repositories we need to give it a personal access token.
The way to do this in a secure way is to add the token as a hidden environment variable in the Travis repository settings.

First go to your GitHub settings page. Under developer settings generate a new personal access token.
If the repository you want to deploy to is public you only need the *public_repo* scope. If it is private you need *repo*.


<div style="text-align: center; margin: 20px 0px;">
    <img src="/assets/access_token.png" style="margin: auto">
    <p>Generating a new personal access token</p>
</div>

Copy this token and go to the Travis CI repository settings for you project repository.
Create a new environment variable named `GITHUB_TOKEN` and set the value to your personal access token. Make sure the *Display value in build log* slider is set to off so your token stays hidden.

<div style="text-align: center; margin: 20px 0px;">
    <img src="/assets/travis_var.png" style="margin: auto">
    <p>GITHUB_TOKEN environment variable in Travis repository settings</p>
</div>

Now Travis should be allowed to commit to our documentation repository.

{% highlight YAML %}
    keep-history: true
{% endhighlight %}
Tells Travis to not force push, but keep the history of the repository.

{% highlight YAML %}
    on:
        branch: master
{% endhighlight %}
Deploy when the push is to the master branch.

{% highlight YAML %}
    local-dir: documentation
{% endhighlight %}
Select what directory to push to the documentation repository.
With the directory structure set up as described earlier we simply want to tell Travis to push the entire `documentation` directory.

{% highlight YAML %}
    repo: <github-username>/<documentation-repository>
{% endhighlight %}
The repository to push the documentation to. Written on the form *username/reponame*.

{% highlight YAML %}
    target-branch: master
{% endhighlight %}
The branch to push to on the documentation repository.

With this set up we can then push to our master branch and Travis will start a new job.
Travis will generate the documentation and then push it to the documentation repository.
If the Travis build would fail make sure to check the log file.
From the log it should be clear what went wrong.

### GitHub Pages
Now we have our documentation in a repository ready to go.
We only have to tell GitHub to start serving it using GitHub pages.

In the documentation repository, go to the settings tab.
Scroll down to GitHub pages and select source *master branch /docs folder*.
This is the reason why we chose the name `docs` for the html documentation directory in the Doxygen config.

<div style="text-align: center; margin: 20px 0px;">
    <img src="/assets/gh_pages.png" style="margin: auto">
    <p>GitHub pages set up to serve to /docs directory from the master branch</p>
</div>

Wait for a second and your documentation should be available at `<github-username>.github.io/<repository-name>`.
Now we have a finished system that will update the documentation after each push to the master branch.

Everything should work as expected at this point, but once everything is set up we can expand on the system in different ways.

### PDF from $$\LaTeX$$
One thing that could be nice is to also include the documentation as a pdf file in the documentation repository.
This can be achieved by configuring Doxygen to output in LaTeX format and then using a LaTeX compiler to create a pdf file.

The only change needed in the Doxygen config is:
{% highlight Text %}
GENERATE_LATEX         = YES
{% endhighlight %}
This tells Doxygen to generate a LaTeX documentation in the `latex` directory.

For compiling this documentation we need a LaTeX compiler and some extra packages.
A common one is TeX Live, easiest installed on debian systems with
{% highlight Bash %}
sudo apt-get install texlive texlive-latex-extra
{% endhighlight %}

We can now look at the pdf documentation by running
{% highlight Bash %}
doxygen doxygen.conf
cd documentation/latex
make
{% endhighlight %}
This should create the file `refman.pdf` in the `latex` folder.

To automate this process and make sure that the pdf is pushed to our documentation repository we need to put this in a script.
Once we are done with the LaTeX files they are unnecessary and should be deleted.
I edited my Makefile to create the pdf, move it out of the latex folder and delete it.
{% highlight Makefile %}
.PHONY: docs

docs:
	doxygen doxygen.conf
	cd documentation/latex && make
	mv documentation/latex/refman.pdf documentation/Documentation.pdf
	rm -r documentation/latex
{% endhighlight %}

With this in the same Makefile command as before we don't have to adjust the Travis config much.
The only necessary change to `.travis.yml` is the addition of the LaTeX packages.

{% highlight YAML %}
before_install:
      - sudo apt-get update
      - sudo apt-get install -y doxygen texlive texlive-latex-extra
{% endhighlight %}

Now Travis will also create a pdf documentation for you.
The file `Documentation.pdf` can be found in the root of the documentation repository.

There are many other ways to adapt and expand on this setup for the needs of different projects.
If you have any questions, ideas for improvements or cool use-cases feel free to reach out on [twitter](https://www.twitter.com/joel_oskarsson) or [email](mailto:joel.oskarsson@outlook.com).
