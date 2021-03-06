Quickstart
==========

The first thing you have to do in order to get started with Ion is to download one of it's releases. For now we only support source code releasing.

Grabbing the Code
-----------------

In order to get the code, you need to have Git installed. There are several tutorials on the web that teach how to install git, so we'll skip that.

After git is installed just clone Ion from `git@github.com:heynemann/ion.git <git@github.com:heynemann/ion.git>`_ and you are good to go::

    git clone git@github.com:heynemann/ion.git

Starting a new Project
----------------------

In order to start a new project using Ion, you can use the bundled console app. It's creatively named ion. The console features several commands, one of them being create. The syntax for create is as follows::

    ion create <project_name>

Where <project_name> is the name for the folder and application you want. Ion creates the whole structure for you, so all the folders and files needed to run a simple website will be created for you. 

Just to test that you got it right, enter the folder ion created (with the same name you entered in the console) and type::

    ion run

You should see something like::

    [08/Feb/2010:10:27:17] ENGINE Bus STARTING
    [08/Feb/2010:10:27:17] ENGINE Started monitor thread '_TimeoutMonitor'.
    [08/Feb/2010:10:27:17] ENGINE Started monitor thread 'Autoreloader'.
    [08/Feb/2010:10:27:18] ENGINE Serving on 0.0.0.0:8001
    [08/Feb/2010:10:27:18] ENGINE Bus STARTED

This means your website is running in port 8001 of your local ip. Just open a new browser and navigate to http://localhost:8001 and you should see the sample page.

DISCLAIMER
----------

For the sake of clarity any tests (unit or functional) that should've been created, will be ommitted in this tutorial. You can learn more about testing your applications using Ion in the :doc:`Testing </testing>` page.

Creating your First Controller Action
-------------------------------------

Ion is an MVC (`model-view-controller <http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller>`_) web framework, so everything is centered in our controllers. Ion provides some basic infrastructure to help you productively create new routes (more about routes later) in order to allow the users of your site to perform more operations.

The sample page you saw before is a good example of a controller action. Let's create a new action so we can see the operations involved.

First open the controllers.py file located in your <project_name>/controllers folder. As you can see, Ion created a controller called DefaultController for you. We'll use this controller for now, but you are free to create as many controllers as you feel like. 

Type the following action in your new controller::

    @route('/posts')
    def show_posts(self):
        return self.render_template('posts.html')

This action won't work until we create the posts.html template it refers to. The render_template method is one of the features of the controller class. It renders a template file (in the templates folder) with the given arguments (we'll see more about the arguments later).

Let's create a template file in the templates folder called posts.html. Type the following into it::

    <html>
        <head>
            <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
            <title>My Sample Project - Posts</title>
        </head>
        <body>
            <p>List of Posts:</p>
            <p>No posts so far.</p>
        </body>
    <html>

Now if you navigate to http://localhost:8001/posts you'll see our newly created page.

Let's Add Some Models
---------------------

Our new page does very little. We can improve it by letting it know more about our model. Let's create our first model, called Post.

Open the <project_name>/models/__init__.py file and type the following::

    #!/usr/bin/env python
    #-*- coding:utf-8 -*-

    from storm.locals import *

    class Post(object):
        __storm_table__ = "post"

        def __init__(self, title, body):
            self.title = title
            self.body = body

        id = Int(primary=True)
        title = Unicode()
        body = Unicode()

This tells our application that posts are entities that feature a title and a body. We can start using them to create and retrieve posts now. In order to allow Ion to properly communicate with your database you have to change the entries in the configuration file, located in your <project_name>/config.ini.

The section that we care about now is the [Db] section, so ignore the rest of the file for now.

The db section of the config.ini is composed of the following keys::

    [Db]
    #Protocol to use to connect to db - Available protocols are 'mysql', 'sqlite' and 'postgres'
    protocol=mysql

    #Database Host
    host=localhost

    #Database Name
    database=database

    #Database User
    user=user

    #Database Password
    password=password

    #Database Port
    port=3306

Each parameter is explained in the config file as above.

To continue, create a database called ionsample and change the parameters above accordingly.

Let's create a table called post with the following script::

    CREATE TABLE `projects` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `title` varchar(255) NOT NULL,
      `body` varchar(2000) NOT NULL,
      PRIMARY KEY (`id`)
    )

Creating a new post
-------------------

First let's create a quick post form in our previous template ('/posts'). Let's change it to be like this::

    <html>
        <head>
            <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
            <title>My Sample Project - Posts</title>
        </head>
        <body>
            <p>List of Posts:</p>
            <p>No posts so far.</p>
            <form action="newpost" method="post">
                Title: <input type="text" name="title" /><br />
                Body: <br />
                <textarea name="body"></textarea>
            </form>
        </body>
    <html>

Ok, now that we have a form that posts the user data to a '/newpost' action. Let's create that action. Add this code to the controller (do not forget from <project_name>.models import *)::

    @route('/newpost')
    def new_post(self, title, body):
        post = Post(title=title, body=body)

        self.store.add(post)

        self.redirect('/posts')

Let's decompose this action into steps. First we create a brand new post with the data the user provided. Then we use the Data Store (from storm) to add this entity to the database. Now that we have created the post we redirect the user to the posts list so he can see his new message.

But...he won't see his message until we list all the posts.

For more information on storm you can check the `documentation <https://storm.canonical.com/#Documentation>`_ for the product.

How about listing our posts
---------------------------

First thing we need to do is return the list of posts in our action, so that our template can use it. Let's change our show_posts action to return all the posts::

    @route('/posts')
    def show_posts(self):
        all_posts = list(self.store.find(Post))
        return self.render_template('posts.html', posts=all_posts)

Now we iterate on the posts in our template::

    <html>
        <head>
            <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
            <title>My Sample Project - Posts</title>
        </head>
        <body>
            <p>List of Posts:</p>
            {% for post in posts %}
                <p>{{ post.title }}</p>
                <p>{{ post.body }}</p>
                <hr />
            {% else %}
                <p>No posts so far.</p>
            {% endfor %}
            <br /><br />
            <form action="newpost" method="post">
                Title: <input type="text" name="title" /><br />
                Body: <br />
                <textarea name="body"></textarea>
            </form>
        </body>
    <html>

You should see a list of posts in the first page now and whenever you post a new message it shows up.

For more information on the templating language and possibilities, check `Jinja's documentation <http://jinja.pocoo.org/2/documentation/>`_.

Conclusion
----------

This quick start should have given you basic directions on how to get started with Ion. 

