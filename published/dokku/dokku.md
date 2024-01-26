Setup Dokku (a free Heroku alternative) and Deploy a Ruby on Rails 7 Application
================================================================================

> Create your own Heroku clone by using Dokku

![Dokku-1](dokku-1.webp "Dokku-1")
A server room, generated with Adobe Express

[Heroku](https://www.heroku.com/) was a popular solution for quick and easy deployments for Ruby on Rails developers. It was handy for testing and learning purposes with its free tier option, offering Platform as a Service (PaaS) solutions for everyone. Unfortunately, [Heroku](https://www.heroku.com/) dropped its free tier solution, making it necessary to pay for every service offered by Heroku. On the hunt for a new solution to quickly and easily deploy Ruby on Rails applications, I found a Heroku alternative called [Dokku](https://dokku.com/).

Dokku is a free open-source Platform as a Service solution that allows you to build your own Heroku clone on your server. It allows you to do git push-based deployments directly to your own Dokku instance running on your server with useful auto-deployment features. Dokku supports pretty much anything which could have been deployed to Heroku.

The main use case I’m covering with Dokku is for testing and development, however, I don’t see any problems running smaller applications also in production using Dokku.

This guide shows you how to set up Dokku on your server and explains how to deploy and configure a Ruby on Rails 7 application (including Postgres, Let’s Encrypt and Database Encryption) as well as how to work with your application (executing commands, logging and configuration).

Prerequisites
=============

What do you need?

*   A VPS (Virtual Private Server) or dedicated server with a fresh Ubuntu 20.04 installation with at least 1GB of RAM.
*   (Optional) Domain name pointing to your server’s IP address.
*   Basic familiarity with the Linux command line as well as SSH.

Part 1 — Install and Setup Dokku
================================

The first part of this article explains how to install and set up Dokku on your server. For this, the following steps have to be executed:

*   SSH into your server as the root user:

```
ssh root@your\_server\_ip
```

*   Download and run the Dokku installation script:

```
wget -NP . https://dokku.com/install/v0.31.0/bootstrap.sh  
sudo DOKKU\_TAG=v0.31.0 bash bootstrap.sh
```

*   Add your domain to Dokku (“acme.com” is your domain). You also have to add an A or CNAME record in your domain’s DNS settings pointing to your server’s IP address.

```
dokku domains:set-global acme.com
```

*   Alternatively, you can just use your servers IP address directly:

```
dokku domains:set-global <your servers IP address>  
dokku domains:set-global <your servers IP address> # add \`.sslip.io\` for subdomain support
```

That’s pretty much it for the Dokku setup. After these steps, your Dokku instance is ready for its first deployment. If you need more instructions, check their excellent documentation [here](https://dokku.com/docs/getting-started/installation/).

Part 2 — Ruby on Rails 7 Application Deployment
===============================================

You can skip the first two steps if you already have an existing application. If not, follow along.

*   Create a new Ruby on Rails application (it’s assumed that you have your system setup for Rails 7 development):

```
rails new myapp && cd myapp
```

*   Create your first commit

```
git add .   
git commit -m "chore: Initial commit"
```

*   To deploy the application to your Dokku server on git push, you simply need to add your server as a new remote git server. This can be done by running the following command:

```
git remote add dokku dokku@<global dokku domain or IP address>:myapp
```

*   After this, you can simply push your application to your Dokku instance:

```
git push dokku master
```

*   By doing this, Dokku will automatically detect that you are pushing a Ruby on Rails application and start the deployment process.

After a successful deployment, you can access your Rails application by visiting: [http://your-domain.com](http://your-domain.com). Dokku also prints the deployment log to your terminal so you can follow along. In the end, the URL of your deployed application is printed for you to access your application.

Part 3 — Database Configuration
===============================

To get your database set up correctly the following steps are necessary. For the sake of this tutorial, it’s assumed that you use Postgres as your production database.

*   Update your database configuration in your Ruby on Rails project. Dokku exports a DATABASE\_URL environment variable which you can use to set up the configuration properly. Therefore, adapt your “database.yml” configuration file with the following lines:

```
production:  
  adapter: postgresql  
  encoding: unicode  
  url: <%= ENV\['DATABASE\_URL'\] %>  
  pool: <%= ENV.fetch("RAILS\_MAX\_THREADS") { 5 } %>
```

*   Next, SSH into your Dokku server and install Postgres plugin for Dokku on your server:

```
dokku plugin:install https://github.com/dokku/dokku-postgres.git
```

*   After that, you can simply create and link a new Postgres database by running this:

```
dokku postgres:create mydatabase  
dokku postgres:link mydatabase myapp
```

*   Now your application deployment can talk to your Postgres database. The linking command automatically restarts the deployment for you s.t. the Rails application is correctly configured.

Additionally, if you work with the Rails migration system, it’s wise to set up an automatic migration command for each deployment. For this, add an “app.json” file to your Rails project’s root path, containing the following lines:

```
{  
  "name": "<Name of Application>",  
  "description": "<Description of your application>",  
  "scripts": {  
    "dokku": {  
      "postdeploy": "bundle exec rake db:migrate"  
    }  
  }  
}
```

This will automatically execute the migration command after every deployment. You can also configure any other command you might need here. Commit and push this file and your deployment should execute the migration automatically.

If you want to use Rails [Active Record Encryption](https://guides.rubyonrails.org/active_record_encryption.html), you need to configure your Rails deployment with the “RAILS\_MASTER\_KEY”, containing the content of the “master.key” file. This can be done by adding the environment variable to the deployment by using the following Dokku command:

```
dokku config:set <name-of-application> RAILS\_MASTER\_KEY=<content of master.key>  
dokku config:show <name-of-application> # shows all environment variables set for the application
```

With this command, it’s also possible to set up any other environment variable you might need in production.

Part 4 — Redis
==============

If your rails application uses Redis as a cache or session store, Dokku offers a solution for that. As with the Postgres setup, you need to install a plugin and configure some environment variables:

*   SSH into your server and install the Redis plugin

```
dokku plugin:install https://github.com/dokku/dokku-redis.git redis 
```

*   Create a Redis instance and link it to your application by running:

```
dokku redis:create myredis  
dokku redis:link myredis myapp
```

*   The Redis plugin exposes a “REDIS\_URL” environment variable to add to your production Redis configuration. Add the environment variable so your Rails application knows where to find your Redis instance. Update your application configuration accordingly and push the changes to Dokku. Your production deployment should pick up the connection to the Redis instance automatically and work as expected.

Part 5— SSL & Let’s Encrypt Setup
=================================

If you need SSL for your application, Dokku has a Let’s Encrypt plugin you can easily integrate. For this, the following steps are necessary.

*   SSH into your server and install the plugin by running this command:

```
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git 
```

*   Configure your application with the Let’s Encrypt email:

```
dokku config:set --no-restart <appname> DOKKU\_LETSENCRYPT\_EMAIL=your@email.tld
```

*   Add the plugin to your application

```
dokku letsencrypt myapp
```

*   That should be all. If you want auto-renewal of your certificates, you can add an easy cron-job script with the following command:

```
dokku letsencrypt:cron-job --add myapp
```

Now you are all set and your application should be accessible via HTTPS.

Tips and Tricks
===============

*   If you want to avoid the SSH connection to your server, Dokku offers the option to run your remote commands from your local command line. For this, you need to install the official Dokku client and configure it with your server’s connection information. For this, see the following [guide](https://dokku.com/docs/deployment/remote-commands/).
*   I encountered a problem with the first deployment of my Ruby on Rails application with a mismatch of the bundler version. This is and was a common problem also for Heroku. You can simply fix this by updating your local bundler version and reinstalling bundler. See this Stackoverflow thread for more information: [Link](https://stackoverflow.com/questions/56774854/how-do-i-fix-a-bundler-conflict-when-pushing-to-heroku).
*   Dokku recommends at least 1GB of RAM for your server infrastructure. From personal experience, if you have a smaller Rails application with a DB connection I recommend at least 4GB of RAM. This ensures proper performance.

Conclusion
==========

The experience of setting up Dokku and deploying a Ruby on Rails 7 application is pretty seamless. It offers everything you expect from a PaaS solution and can be easily configured and extended to your application needs. This tutorial covers the basic configuration as well as the deployment instructions necessary.

For further information, check the reference links down below.

Happy Coding

Michael

Thanks for reading this Medium article. If you want, follow me on [Twitter](https://twitter.com/michael_wapp).

**Related Links**

*   [https://dokku.com/docs/getting-started/installation/](https://dokku.com/docs/getting-started/installation/)
*   [https://github.com/dokku/dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt)
*   [https://github.com/dokku/dokku-postgres](https://github.com/dokku/dokku-postgres)
*   [https://github.com/dokku/dokku-redis](https://github.com/dokku/dokku-redis)
*   [https://guides.rubyonrails.org/getting\_started.html](https://guides.rubyonrails.org/getting_started.html)
