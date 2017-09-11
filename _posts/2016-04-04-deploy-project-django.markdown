---
layout: post
title:  "Deploy Django Project in CentOS 7.2"
date:   2016-04-04 15:47:00 -0200
excerpt: "Today We will deploy my first project created with Django Framework 1.9 in CentOS 7.2."
project: false
tag:
- deploy 
- django
- centOS
comments: true
---

Today We will deploy my first project created with Django Framework 1.9 in CentOS 7.2. To do that, first I created a droplet with CentOS 7.2 installed. If you don't know how to create a droplet in Digital Ocean, see this [tutorial][tutorial-create-droplet]{:target="_blank"}. Lastly, to deploy the project, we will use Apache with mod_wsgi.

Now, let's start:

- First update repositories:
{% highlight ssh %}
yum update -y
{% endhighlight %}

- To get `pip`, we'll need to enable the EPEL repository, which as some additional packages. You can do that easily by typing:
{% highlight ssh %}
sudo yum install epel-release
{% endhighlight %}

- With EPEL enabled, we can install the components we need by typing:
{% highlight ssh %}
sudo yum install python-pip httpd mod_wsgi 
{% endhighlight %}

- Now, we will install Mysql for database project. So, let's download and install MySQL repository:
{% highlight ssh %}
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
{% endhighlight %}

- After repositories is OK, let's install MySQL Server and Start it !
{% highlight ssh %}
yum install mysql-server
systemctl start mysqld
{% endhighlight %}

- With MySQL installed, we will configure using following command:
{% highlight ssh %}
mysql_secure_installation
{% endhighlight %}

- Follow steps configuration. Before we download the project from git, let's add user and put the project in user's folder. In this way, you can add others projects in others users, using one server. Substitute <username> in any user:
{% highlight ssh %}
useradd <username>
passwd <username>
{% endhighlight %}

- After added user, let's install git and download django project in user folder:
{% highlight ssh %}
yum install git -y
cd /home/<username>
git clone <URL-Repository>
{% endhighlight %}

- Using git clone, you will download all files project into your user's folder. Now, we use `virtualenv` and `virtualenvwrapper` to manage each project virtual enviroment. So let's install using `virtualenvwrapper`, because it will install `virtualenv` automatically:
{% highlight ssh %}
pip install virtualenvwrapper
{% endhighlight %}

- To bash CentOS recognize `virtualenvwrapper` commands, let's add 3 lines at ~/.bash_profile, so use command `nano ~/.bash_profile` and add this content:
{% highlight ssh %}
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/Devel
source /usr/bin/virtualenvwrapper.sh
{% endhighlight %}

- After that, we will create our virtualenv and install all requeriments of your project in txt:
{% highlight ssh %}
mkvirtualenv <nameproject>
yum install gcc python-devel mysql-devel libevent-devel
pip install -r requeriments.txt
{% endhighlight %}

- Let's now create database for your project and import structure by Django migrations:
{% highlight ssh %}
mysql -u root -p<PASSWORD>
	create database <databasename>;
	exit;

python manage.py migrate
{% endhighlight %}

- Create a superuser Administration:
{% highlight ssh %}
python manage.py createsuperuser
{% endhighlight %}

- Now, we will collect static files, but you should certificate your `settings.py` STATIC_FILES is configured correctly:
{% highlight ssh %}
python manage.py collectstatic
{% endhighlight %}

- So, all it's ok ! We just need to configure `Apache` with `mod_wsgi`. To do that, we will create a file configuration at `/etc/httpd/conf.d/` with any name:
{% highlight ssh %}
nano /etc/httpd/conf.d/django.conf
{% endhighlight %}

- And put this content, replacing variables <> with your username or nameproject. The <envproject> is the name of VirtualEnv you created to your project.
{% highlight ssh %}
<VirtualHost *:80>
    ServerName www.mydomain.com
    DocumentRoot /home/<username>/<nameproject>

    Alias /static /home/<username>/<nameproject>/static
    <Directory /home/<username>/<nameproject>/static>
        Require all granted
    </Directory>

    <Directory /home/<username>/<nameproject>/<nameproject>>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>

    WSGIDaemonProcess <nameproject> python-path=/home/<username>/<nameproject>:/<username>/.envs/<envproject>/lib/python2.7/site-packages
    WSGIProcessGroup <nameproject>
    WSGIScriptAlias / /home/<username>/<nameproject>/<nameproject>/wsgi.py

</VirtualHost>
{% endhighlight %}

- Now we will put some permissions and we finish !
{% highlight ssh %}
chmod 664 /home/<username>/<nameproject>
sudo chown :www-data /home/<username>/<nameproject>
{% endhighlight %}

- To finish, we restart Apache !
{% highlight ssh %}
sudo service apache2 restart
{% endhighlight %}

[tutorial-create-droplet]: https://www.digitalocean.com/community/tutorials/how-to-create-your-first-digitalocean-droplet-virtual-server