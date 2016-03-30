---
layout: post
title: "Upgrading from Postgres 9.4 to 9.5 when upgrading to Ubuntu 16.04 Xenial Xerus"
permalink: ubuntu-16-04-upgrading-postgres-9-4-to-9-5
date: 2016-03-28 13:27:53
comments: true
description: "How to move databases from postgres 9.4 to 9.5 when upgrading to Ubuntu 16.04 Xenial Xerus"
keywords: ""
categories: devops, ubuntu, postgres, db

tags:

---
This is how I moved my databases from my postgresql 9.4 cluster to 9.5 when upgrading from 15.10 to 16.04 Xenial Xerus

<br />

### Step 1

#### switch to the postgres user

from <https://docs.docker.com/engine/installation/ubuntulinux/>
{% highlight sh %}sudo su - postgres{% endhighlight %}

as long as you use the - option for su you should be in the postgres home folder which is writable, if not you can just do this all in /tmp
{% highlight sh %}cd /tmp{% endhighlight %}
now the important stuff

### Step 2

#### Run the pg_upgrade command

{% highlight sh %}
/usr/lib/postgresql/9.5/bin/pg_upgrade \
-b /usr/lib/postgresql/9.4/bin/ \
-B /usr/lib/postgresql/9.5/bin/ \
-d /var/lib/postgresql/9.4/main \
-D /var/lib/postgresql/9.5/main \
-o ' -c config_file=/etc/postgresql/9.4/main/postgresql.conf' \
-O ' -c config_file=/etc/postgresql/9.5/main/postgresql.conf'
{% endhighlight %}
at some point this failed for me and I had to move both the old and new postmaster pid files to backups
{% highlight sh %}mv /var/lib/postgresql/9.4/main/postmaster.pid /var/lib/postgresql/9.4/main/postmaster.pid.bak{% endhighlight %}
{% highlight sh %}mv /var/lib/postgresql/9.5/main/postmaster.pid /var/lib/postgresql/9.5/main/postmaster.pid.bak{% endhighlight %}

Then, I got another error because I had moved the postmaster.pid files :)
Instead of moving them back I first simply tried re-running the /usr/lib/postgresql/9.5/bin/pg_upgrade command which was successful.

### Step 3

#### Finishing touches

Next, I tried to run {% highlight sh %}./analyze_new_cluster.sh{% endhighlight %} but I needed to {% highlight sh %}exit{% endhighlight %} the postgres user first and then start postgres with systemctl {% highlight sh %}sudo systemctl start postgresql{% endhighlight %}

Now just go back to your postgres user and the folder you ran the commands and run (or re-run) {% highlight sh %}./analyze_new_cluster.sh{% endhighlight %} which should now be successful.

<br />
