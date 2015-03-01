# Tutorial: Windows Azure, Ruby on Rails, Capistrano 3 & PostgreSQL

This tutorial will show you how to create and deploy a basic Ruby on Rails app onto your own [Windows Azure](http://windowsazure.com) Linux based Virtual Machine using [Capistrano 3](http://capistranorb.com) to manage the deployment tasks including database migrations and versioning. Be sure to follow closely and don't skip any steps, missing just one can result in lots of frustration (trust me, I know!).

## NOTE

GitHub does not support embeding gists in markdown. Therefore either follow the README.html file or download and compile this yourself.

## Time

![Time][clock] Approximately 40 - 60 minutes

## Assumptions

* You have a subscription ([or free trial](http://www.windowsazure.com/en-us/pricing/free-trial/)) with [Windows Azure](http://windowsazure.com)
* You have a [GitHub account](http://github.com)
* You are using OSX (though any Linux distro should be fine)
* You have Ruby &amp; Ruby on Rails installed (options include [rubyonrails.org](http://rubyonrails.org/), [rbenv](https://github.com/sstephenson/rbenv), [bitnami](http://bitnami.com/stack/ruby) etc.)
* You are comfortable with [Terminal](http://en.wikipedia.org/wiki/Terminal_(OS_X)) &amp; [SSH](http://en.wikipedia.org/wiki/Secure_Shell)
* You know basic commands for [nano](https://wiki.gentoo.org/wiki/Nano/Basics_Guide) or [vi](http://www.cs.colostate.edu/helpdocs/vi.html) at least enough to open, edit and save files

## Getting Started

This tutorial is based on this Windows Azure article [Deploy a Ruby on Rails Web application to a Windows Azure VM using Capistrano](http://www.windowsazure.com/en-us/develop/ruby/tutorials/web-app-with-capistrano/) and [Capistrano 3 Tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/) by [@TalkingQuickly](http://twitter.com/TalkingQuickly).

### Create your SSH Key
![Time][clock] 1 minute

You will need to upload a **Windows Azure compatible SSH key**. To create one, open `Terminal` & run the following in a folder where you wish to store your keys. I recommend using the `~/.ssh` folder. [Jeff Wilcox](http://www.jeff.wilcox.name/2013/06/secure-linux-vms-with-ssh-certificates/) has a great post on creating SSH Keys which I've borrowed from.

You'll be prompted for information like country, state or province, organization etc. You can put whatever you want in there, or leave it blank, up to you.

> **Don't** copy+paste this entire script, it won't work. Run each command separately.

<script src="https://gist.github.com/m-gagne/7afdcff597099e8b5fd1.js"></script>

### Create a Virtual Machine

![Time][clock] 5 minutes

Virtual Machine, VM, Virtual Private Server, VPS or server, basically it all means a "machine" in the cloud for you to setup.

The Windows Azure team has a [more detailed article](http://www.windowsazure.com/en-us/manage/linux/tutorials/virtual-machine-from-gallery/) here that I've used, but the basics are:

* log into **[portal.windowsazure.com](http://portal.windowsazure.com)**
* select **New**
* select **Compute**
* select **Virtual Machine**
* select **From Gallery**

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/new.png)

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/vm-add-from-gallery-620.png)

**Step 1** Select your **Linux distro** (I'm partial to Ubuntu). Not sure which version to chose? Ubuntu Server 12.04 LTS is a good option, (LTS stands for Long Term Support). Once selected click ![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows_azure_wizard_next-28.png).

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows-azure-create-vm-step-1-resized.png)

**Step 2** Configure your VM

* Enter a unique **Virtual Machine Name**
* Choose a **Size** for your VM (small will do just fine)
* Enter your **User Name** (or leave it set to the default of 'azureuser' for simplicity)
* Upload your `azure.pem` SSH certificate (be sure to upload the `.pem` file not the `.key` file)
    * While in the file upload dialog **to navigate to your .SSH folder** simply start typing `~/.ssh` and hit ente
* You can **optionally** set a **password**, however it's good practice to use certificate based authentication for your linux servers

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows-azure-create-vm-step-2-resized.png)

**Step 3** Let's just use the defaults here. You can learn more about these options [here](http://www.windowsazure.com/en-us/documentation/articles/virtual-machines-linux-tutorial/).

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows-azure-create-vm-step-3-resized.png)

** Step 4** Be sure to **add the HTTP endpoint** so you can access your app.  Endpoints are public ports that are opened on your server so they are accessible via the internet to everyone. For most websites and web apps you'll want to add HTTP (port 80) and HTTPS (port 443). Click ![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows_azure_wizard_submit-28.png) to create your server.

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/windows-azure-create-vm-step-4-resized.png)

### Once your server is ready

![Time][clock] 5 minutes

It'll take about 4-5 minutes or so to create your server. Once it's ready open `Terminal` and connect to your server by running

<script src="https://gist.github.com/m-gagne/b0066b6a91db4068e1c0.js"></script>

Replacing

* `<azureuser>` with your username that you specified during creation (default is azureuser)
* `<vmname>` with the  virtual machine name (or DNS name) provider during creation.

When promoted be sure to agree to add your server to the known hosts.

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/ssh_to_vm.png)

### Install Git, Ruby &amp; Nginx

![Time][clock] 12 minutes

Once connected to your server install [git](http://git-scm.com/) and clone the following file into `~/setup` and run the setup script. This script ([which you can view here](https://gist.github.com/m-gagne/9234940))  will

* Update & upgrade installed packages
* Install various build tools
* Install [Node.js](http://nodejs.org/)
* Install [Nginx](http://wiki.nginx.org/Main)
* Setup a local (user) deployment of [rbenv](http://github.com/sstephenson/rbenv)
* Install (using [rbenv](https://github.com/sstephenson/rbenv)) Ruby 2.0.0-p451
* Install the [Bundler](http://bundler.io/) gem

> **Don't** copy+paste this entire script, it won't work. Run each command separately.

<script src="https://gist.github.com/m-gagne/2ddfdaf2ca3cc1223035.js"></script>

Now would be a good time to grab a coffee or maybe explore the Windows Azure portal a little more because this is going to take 10-12 minutes.

### Install PostgreSQL

![Time][clock] 3 minutes

Time to install a database server, although this tutorial uses [PostgreSQL](http://www.postgresql.org/) you could just as easily use [MySQL](http://www.mysql.com/), [Windows Azure SQL](http://www.windowsazure.com/en-us/services/sql-database/) or any database supported by Ruby.

<script src="https://gist.github.com/m-gagne/9259868.js"></script>

### Create a user & database

![Time][clock] 2 minutes

You'll need to edit PostgreSQL config to change from [Peer to MD5 authentication type to allow for password-based authentication](http://www.postgresql.org/docs/9.1/static/auth-methods.html).

<script src="https://gist.github.com/m-gagne/80c1d08ca47448860c8b.js"></script>

The above command opens the config file to line 90, all you need to do is change from peer (user) to md5 (user+pass) for authentication.

Change this:

<script src="https://gist.github.com/m-gagne/8dba324a9c45ee1a741c.js"></script>

To this:

<script src="https://gist.github.com/m-gagne/5b60a831eac45664d11e.js"></script>

> Hint: `ctrl + x`, `y`, `<enter>` will save and exit nano

Now reload PostgreSQL configuration

<script src="https://gist.github.com/m-gagne/0f397bf81bb267a2d26c.js"></script>

Time to create a user & database (remember these settings as you'll need to configure your app to use them later!). Don't forget to change

*   `my_username` to your desired username
*   `my_database` to your desired database name

<script src="https://gist.github.com/m-gagne/e2815b6a678b60a92f69.js"></script>

Verify it works by connecting to it

<script src="https://gist.github.com/m-gagne/d9bc3968e93fd89a5068.js"></script>

> Hint: to quite simply type `\q <enter>`

### Remove the default Nginx website

If you try connecting to your server (`<vmname>.cloudapp.net`) you will see the default website included when you installed Nginx. We are going to let Capistrano create a new nginx configuration for us that points to our app, so let's delete that default website.

<script src="https://gist.github.com/m-gagne/2c02d1ccc5b169673ba0.js"></script>

## Setting up your Ruby on Rails App

We will now setup your (local) machine & app for deployment.

### Capistrano

[Capistrano](http://capistranorb.com/) is a  remote server automation and deployment tool written in Ruby. We will use it to manage setting up and deploying our app to our server(s).

### Sample App

I recommend forking & cloning [this sample app](https://github.com/m-gagne/ror-azure-demo) for a few reasons.

* First if you don't yet have a Ruby on Rails app you can simply deploy this one
* Second, it will make it easier to copy configuration files into your own application once you know how it works
* This app is very simple and so it's easier to trouble shoot if somethig doesn't work

### Step by step instructions on integrating Capistrano 3

For detailed step by step instructions, especially good for those wanting to edit an existing app, please reference this post [Capistrano 3 Tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/) by [@TalkingQuickly](http://twitter.com/TalkingQuickly).

### Starting with the sample app

![Time][clock] 4 minutes

Fork the demo app

* Open [https://github.com/m-gagne/ror-azure-demo](https://github.com/m-gagne/ror-azure-demo)
* Click the fork icon ![](http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/github_fork_28.png) to fork this repo to your GitHub account

Open `Terminal` on your local machine &  clone the sample app into your coding directory (for example `~/Code`)

<script src="https://gist.github.com/m-gagne/f2c49bdb873985e94cfd.js"></script>

#### Create a database.yml configuration file

The sample app ships with an example database config file `database.example.yml`. Copy it to `database.yml` and configure it as needed to match your development machine. The sample uses sqlite3, but you can use Mysql, PostgreSQL and others. Why didn't I include a database.yml file? It's good practice to not include any configuration that would contain usernames, passwords or server information which is why I've excluded database.yml in the `.gitignore` file.

Don't forget to `cd` into the app folder `cd ror-azure-demo`

<script src="https://gist.github.com/m-gagne/d43ad4ef89d8eab6d80f.js"></script>

You will want to edit the default values in `config/database.yml` to match your own development environment

    development:
      adapter: sqlite3
      database: db/development.sqlite3
      pool: 5
      timeout: 5000

#### Gemfile

The following are gems needed for Capistrano deployment. These are already included in the sample apps `Gemfile` and are only included here for reference if you are adapting this to your own app.

<script src="https://gist.github.com/m-gagne/c449c7e62cd2af374ab0.js"></script>

Run the following to install any missing gems

<script src="https://gist.github.com/m-gagne/a79c7fb6d6fa49a2ed49.js"></script>

#### Capistrano Files
The following are folders & files created by Capistrano if you were to run `cap install`. The sample app is already setup and this is not required.

<pre>
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
</pre>

The sample app also includes a number of configuration files to instruct Capistrano to setup nginx, unicorn and more. To better understand how that works see the files in `config/deploy/shared` and read this [tutorial](http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/).

#### Edit config/deploy.rb

The file `config/deploy.rb` is the basis for your deployment configuration in Capistrano. The settings to change in this file are:

* `:application`
    * Set this to a name for your application
    * It will be used in the deployment for things like the deployment folder
* `:deploy_user`
    * Set this to the username you selected when you created your server
    * The default when setting up a server on Windows Azure is `azureuser`
* `:repo_url`
    * Set this to the URL for your applications GitHub repository

<script src="https://gist.github.com/m-gagne/2ef3fd15492164e1a0ab.js"></script>

#### Edit config/deploy/production.rb

Capistrano allows you to create multiple stages (dev, test & production for example). In this tutorial we'll focus on production.

The idea is that `config/deploy.rb` which we configured above contains the common (stage agnostics) configuration and the stage specific files contain the rest.

In `config/deploy/production.rb` the main line to edit is

<script src="https://gist.github.com/m-gagne/4a1c5b5af160d51b3887.js"></script>

You will need to change

* `yourserver.cloudapp.net`
    * change this to your your servers DNS address (you should just need to change yourserver to the name of the Virtual Machine you recently created.)
* `azureuser`
    * change this to the username you specific when creating your server. By default on Windows Azure this is `azureuser`

### Time to deploy

#### Setting up your apps configuration

![Time][clock] 2 minutes

First tell Capistrano to deploy the base configuration of your application to the desired stage. For this tutorial we'll deploy to `production` by running the following command

<script src="https://gist.github.com/m-gagne/4e65ed551c06ead966e5.js"></script>

This will create the following files & folders on your servers home (`~/`) directory:

    ~/apps/
    └── ror-azure-demo_production
      └── shared
          └── config
              ├── application.yml
              ├── database.example.yml
              ├── log_rotation
              ├── monit
              ├── nginx.conf
              ├── unicorn_init.sh
              └── unicorn.rb

So what did Capistrano do exactly?

*   Created an `apps` folder in your home directory
*   Created a folder for your app using the structure `appname_stage`
*   Created a `shared/config` folder for configuration that each release/deployment of your app will reference

You might have noticed that you have a `database.example.yml` file. That's because it's a bad idea to store your production credentials in your code. So you'll need to create a database.yml file on your production serer. For that you can simply copy the example file and edit as needed.

`SSH` to your server and run the following to create & edit a `database.yml` file

<script src="https://gist.github.com/m-gagne/b0066b6a91db4068e1c0.js"></script>

<script src="https://gist.github.com/m-gagne/9c6fc35c7ff5194ed625.js"></script>

This will create and open the following `database.yml` file

<script src="https://gist.github.com/m-gagne/5e4c85a8ef7381b2c692.js"></script>

You will want to edit it as appropriate, however if you are using PostgreSQL then all you'll need to edit is

* `username`
* `password`
* `database`

#### Restart Nginx

While SSH'd into your server you will want to restart Nginx to pick up the new virtualhost that was added when you ran `cap production deploy:setup_config`

<script src="https://gist.github.com/m-gagne/5d2ec8c9a0f4b48367e2.js"></script>

#### Deploy your application

![Time][clock] 3 minutes

Time to deploy your application to your server! On your local machine in your app folder run

<script src="https://gist.github.com/m-gagne/697e37f7d2be2d1b32e7.js"></script>

This will take a few minutes as part of the initial (or 'cold') deployment is to compile and install all the gems.

### Fingers Crossed

If all went well you should now be able to point your browser to your server and see your app deployed in production on your very own Windows Azure Linux Virtual Machine!

![](http://mediafiles.w00t.ms/Images/Articles/uploads/2013/11/achievement-unlocked-rails-azure-first-app.png)

### Learn More

#### Startups
Did you know you can get free Windows Azure credits just for being a [BizSpark](http://aka.ms/bizsparkcanada) startup (a free program from Microsoft). As a BizSpark startup you get $150 a month in Windows Azure hosting credits for three years, there's even an opportunity to be nominated for $60,000 in Windows Azure credits if your startup needs that kind of scale. [Find out more here](http://aka.ms/bizsparkcanada).

#### Windows Azure
Want to learn more? Hit up [windowsazure.com](http://windowsazure.com) for scenarios, documentation and tutorials, also why not check out [Microsoft Virtual Academy](http://www.microsoftvirtualacademy.com/product-training/windows-azure) which includes free courses for Windows Azure.

Follow me [@marc_gagne](http://twitter.com/marc_gagne) to learn more about Windows Azure, startups  and more.


[clock]: http://mediafiles.w00t.ms/Images/Articles/uploads/2014/03/azure-ror-tutorial/clock_28.png
