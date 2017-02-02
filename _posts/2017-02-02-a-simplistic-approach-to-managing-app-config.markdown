---
layout: post
title:  "A simplistic approach to managing app config"
date:   2017-02-02 00:00:00
---
I've recently been involved in the transformation of the processes used to deliver change from being heavily manual, time consuming and error prone to fully automated, efficient and reliable.

To drive the new automated process without human interaction, other than maybe clicking a single button, we knew that we would need a way of managing application configuration that allowed the process to simply pick up the config values during its execution. Our automated deployment process was driven by bash scripts so compatibility with bash was desirable.

There were two types of config we had to deal with:

* Deployment - which components go where etc.
* Application - runtime parameters e.g. JDBC connection params

An additional complication to the management of config was due to the nature of our business with us hosting multiple deployments of the same product for different customers, up to around 20. This meant that the following dimensions had to be considered when deciding how to manage the config params:

* Product
* Customer
* Environment e.g. development (DEV), integration testing (IT), user acceptance testing (UAT), production (PROD)
* Version

We also decided that it was important to have an audit trail of changes to the config to help with the analysis and debug of any issues that may arise.

This would all be tied together through a Jenkins job where any preparation of the config could be performed and an artifact created to ease consumption by other Jenkins jobs.

So to summarise, our requirements were:

* Compatibility with automated deployment process i.e. consumable by shell scripts (Bash)
* Support multiple customers, products, environments and versions
* Audit of changes to config

## The Solution

The only tools you'll need for this solution are a shell environment (i.e. Bash), your favourite file editor and Git.

### Shell compatibility

Starting with the first requirement, the need for the config to be consumable by shell scripts, we opted to define a set of key/value pairs in the form of Linux environment variables in a shell script called `params.sh`, for example:

{% highlight bash %}
MY_PARAM1=value1
MY_PARAM2=value2
{% endhighlight %}

This provides a very simple way to inject parameter values into the process performing the application deployment. This could be via a Jenkins job or simply by sourcing the params file into another shell script e.g.

{% highlight bash %}
source params.sh
{% endhighlight %}

### Support for multiple customers, products, environments and versions

Addressing the second requirement, to support the provision of config files for multiple customers, products, environments and versions, we had to find a way of organising the config is such a way to make it easily accessible.

The options boiled down to preferring the config to be accessible first by customer then product or by product then customer. To illustrate this the directory structures for storing the config of each option would be:

{% highlight bash %}
# Customer -> Product -> Environment -> Version
CUST1
../PRODUCT1
..../UAT
....../1.0.0
....../1.1.0
......../params.sh
..../PROD
....../1.0.0
......../params.sh
../PRODUCT2
CUST2
...

# Product -> Customer -> Environment -> Version
PRODUCT1
../CUST1
..../UAT
....../1.0.0
....../1.1.0
......../params.sh
..../PROD
....../1.0.0
......../params.sh
../CUST2
PRODUCT2
...
{% endhighlight %}

To be honest it was six of one, half a dozen of the other as to which structure would be better. We went with the first option, customer then product, mainly due to the fact that then for a single customer we would have all of their config grouped together. As mentioned though there are arguments just as strong the other way.

So now we have the ability to source a parameter file into a Bash script that will be held in a directory structure such as `./CUST1/PRODUCT1/UAT/1.1.0/params.sh`.

### Auditing

Finally, to meet the requirement of having an audit log of all config related changes we simply host the above file structure and parameter files in a Git repo.

Whenever we need create or update create a new Git commit and push the change to the central Git repo. A Jenkins job will then clone the repo and create an artifact (i.e. tar file) containing all config that can then be consumed by other processes (i.e. other Jenkins jobs).

As we are using Git we also include an audit log in the artifact to make it easier to reference the changes made to the config wherever they are being used. This is achieved with via the `git log` command with some special formatting:

{% highlight bash %}
git log --pretty=format:"%h - %an (%ae), %ad : %s" > commit.log
{% endhighlight %}

This produces output such as:

{% highlight bash %}
e3e0728 - Nick Ebbitt (someone@gmail.com), Mon Jan 30 20:34:11 2017 +0000 : Fix grammatical error
81826b4 - Nick Ebbitt (someone@gmail.com), Wed Jan 25 13:58:48 2017 +0000 : Reword introduction
af7a186 - Nick Ebbitt (someone@gmail.com), Wed Jan 4 21:03:44 2017 +0000 : Add Amazon Linux to tech skills
167244c - Nick Ebbitt (someone@gmail.com), Wed Jan 4 10:04:22 2017 +0000 : Update footer to 2017
61ef2f7 - Nick Ebbitt (someone@gmail.com), Tue Jan 3 21:06:32 2017 +0000 : Update photo on main page
5fa1a83 - Nick Ebbitt (someone@gmail.com), Tue Jan 3 19:19:07 2017 +0000 : Update technical skills on CV
{% endhighlight %}

## Final thoughts...

So that's it. As in the title, this is a very simplistic approach that may not stand the test of the time but it works quite well for our basic needs right now.

Currently this solution doesn't provide a way for app components to discover their config or receive config changes at runtime however without too much effort this solution could be reworked to play nicely with [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/) or an equivalent tool.
