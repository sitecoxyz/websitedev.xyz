---
layout: post
title: "Multiple SSL Rails apps on one VPS with Docker"
permalink: multiple-ssl-rails-apps-on-one-vpn-with-docker
date: 2016-02-10 12:47:53
comments: true
description: "Multiple SSL Rails apps on one Virtual Private Server with Docker"
keywords: ""
categories: devops, docker, deployment, aws, passenger, nginx

tags:

---
This is a step by step guide to setting up an Ubuntu 14.04 VPS from scratch to serve multiple apps quickly and easily, with SSL automatically set up.  We will be using nginx, docker, and letsencrypt and we will even throw in a Postgres container to house our databases.

<br />

### Step 1

#### Install Docker

from <https://docs.docker.com/engine/installation/ubuntulinux/>
{% highlight sh %}sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D{% endhighlight %}

open up the .list file
{% highlight sh %}sudo vi /etc/apt/sources.list.d/docker.list{% endhighlight %}
add the line
{% highlight debsources %}deb https://apt.dockerproject.org/repo ubuntu-trusty main{% endhighlight %}
save and close

{% highlight sh %}
apt-get update
sudo apt-get install linux-image-extra-$(uname -r)sudo apt-get install linux-image-extra-$(uname -r)
sudo apt-get install docker-engine
{% endhighlight %}
<br />

### Step 2

#### Set up Postgres container

This will hold the postgres databases for all apps on the server, just pass in the password.
{% highlight sh %}sudo docker run --restart=always --name postgres -e POSTGRES_PASSWORD=pUbl1Shthis -d postgres{% endhighlight %}
<br />

### Step 3

#### Set up Nginx proxy container

{% highlight sh %}sudo docker run --restart=always --name nginx-proxy -d -p 80:80 -p 443:443 -v /home/ubuntu/certs:/etc/nginx/certs -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy{% endhighlight %}
<br />

### Step 4

#### Set up letsencrypt container

Here we are using the excellent <https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion>

{% highlight sh %}sudo docker run --name letsencrypt -d -v /home/ubuntu/certs:/etc/nginx/certs:rw --volumes-from nginx-proxy -v /var/run/docker.sock:/var/run/docker.sock:ro jrcs/letsencrypt-nginx-proxy-companion{% endhighlight %}

Yay! SSL!

<br />

### Step 5

#### Launch your apps

#### Sample Dockerfile

from <https://libertyseeds.ca/2015/09/04/Deploy-multiple-Rails-apps-with-Passenger-Nginx-and-Docker/>

{% highlight docker%}
FROM phusion/passenger-ruby22:latest
MAINTAINER Some Groovy Cat "hepcat@example.com"

# Set correct environment variables.
ENV HOME /root
ENV RAILS_ENV production

# Use baseimage-docker's init process.
CMD ["/sbin/my_init"]

# Start Nginx and Passenger
EXPOSE 80
RUN rm -f /etc/service/nginx/down

# Configure Nginx
RUN rm /etc/nginx/sites-enabled/default
ADD docker/my-app.conf /etc/nginx/sites-enabled/my-app.conf
ADD docker/postgres-env.conf /etc/nginx/main.d/postgres-env.conf

# Install the app
ADD . /home/app/my-app
WORKDIR /home/app/my-app
RUN chown -R app:app /home/app/my-app
RUN sudo -u app bundle install --deployment

# TODO: figure out how to install `node` modules without `sudo`
RUN sudo npm install

RUN sudo -u app RAILS_ENV=production rake assets:precompile

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
{% endhighlight %}
