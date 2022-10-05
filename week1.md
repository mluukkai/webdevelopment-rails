## Web-sovellusten toimintaperiaatteita


The idea behind a Web application is simple. Users open a browser and write the page URL they want into the address line, for instance http://www.cs.helsinki.fi/opiskelu/index.html. The first part of the URL, in this case www.cs.helsinki.fi is usually the DNS name, which helps us to identify the IP address of the server of the www-page. The browser sends a page request to the server Web using the GET-method of the HTTP protocol. If the address is correct, the user who sent the page request has the right to the resources defined in the URL path (for instance opiskelu/index.html), the server returns to the browser a _status code_ 200 and the page content in HTML format. The browser renders the page to the user. If the page does not exist, the server returns a status code 404 to the browser, which tells of a mistake.

The page returned by the server can be __static__, that is "manually" written down into a HTML file which is located in the server. Also, it can be __dynamic__: for instance, it can be generated while the request is made, based on the data in the server database. For instance, the course list at the page http://www.cs.helsinki.fi/courses is read in the database and the html code rendering the page is created again every time we access the page, based on the list of the courses available in the database at that time.

Sometimes, the path direction of the www-page information changes and the date are sent from the server to the browser. Typically, there will be a form on the page where the user can enter the information for the server. The HTTP protocol provides us with the POST method to send information (the HTTP GET method can be used to send information, too). For instance, you can see the search form "Hae tältä sivustolta" in the corner on the top right of the www-pages of the departnment, which allows users to send data to the Web server. When the users presses on "hae" (search), the browser sends a POST-method request to the server http://www.cs.helsinki.fi. The method posts the string the user wrote into the form. Most commonly, the server answers to the calls made by posting the form and returning a new HTML file, which the browser renders to the user. (In fact, POST-calls are not usually answered by returning an HTML page, but we are rederected to the page which contains the HTML code which has to be rendered, see http://en.wikipedia.org/wiki/Post/Redirect/Get, we'll speak more on the topic in the second week.)

In addition to the address, the data (that is, the message body) and the [statuscodes](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes), HTTP requests also containtain the __headers__ data (see http://en.wikipedia.org/wiki/List_of_HTTP_header_fields). They help to make requests and their answers more specific, for instance by defining what kind of data the browser is ready to receive.

With the words Web Server Programming, we refer to how the Web server creates the Web pages to show to the browser and handles the data the user enters with the help of a form.

Nowadays, Web pages are not simple HTML. HTML is used to represent the page structure and contants. The page design is usually affected by using files like CSS, see http://en.wikipedia.org/wiki/Cascading_Style_Sheets. Recently, there's been a trend to provide the www-page with more and more code which has to be executed in the __browser__, which is made in Javascript. It's a matter of opinion what has to be created in the browser and what in the server. For instance, suppose the www-page contains a form to log in into the website. It's clear that password and user name have to be checked by the server. Differently, we can allow the browser Javascript to check whether the user name box is empty when they try to log in. In such situations it is useless to bother the browser at all, because the log in would never succeed.

The latest trends have been pushing towards Web applications which remind of normal desktop applications as much as possible. A good example is Google drive, which "imitates" Word/Openoffice functionality as accurately as possible. In such applications, the largest part of the application logic is realised in the browser. In any case, we always need the functionality executed in the server, otherwise information can not be shared by users who use the application in different places.

When we retrive data from the server in modern applications, the server does not necessary return ready-made HTML-pages. On the contrary, it returns raw data (usually in json format); these data will be handled by the Javascript code that is executed in the browser and will be displayed on the page. In this way, only the required part of the website will update.

In this course, we focus almost completely on implementing the functionality on the Web-application side. In weeks 6 and 7, we take a look at a couple of examples of functionality which is implemented with Javascript as well as of application design through CSS files.

All the exercises of the course are contained in this material. Except for the following exercise, all the others have to be returned using github. In addition to returning them with github, we keep track of the homework by marking the exercises in the appropriate system. We'll speak more about this at the end of the page. Let's start with the first exercise, in any case.

> ## Excercise 1: HTTP in action
>
> The browsers' Web developer tools are extremely useful when we implement the functionality on the browser. The most friendly browser to Web developers is Chrome, and we assume here that you are using Chrome. Other browsers also have similar functionality.
>
> Open Chrome developer tool by pressing Shift, Control and i (or F12). You also have access to the developer tool through the Tools menu. Open the Network tab. The tab shows the HTTP requests which are sent by the browser and the server answers.
>
> Copy-paste http://www.cs.helsinki.fi/courses into the browser address line and press enter.
> You see the GET-requests which have been caused to charge the page. Open it (clicking on the call) and inspect what is that happens with the call. Have a special look at the headers and the response part. The Developer tools show request headers and response headers separately.
>
> Each request returns HTML code which is shown in the response tab. The code contains references to files like CSS, Javascript, as well as pictures. When it comes to rendering the page, the browser retrieves each of these with their own GET-request.
>
> Keep open the same networking tab. Clear the tab by pressing on the symbol which looks like a no entry sign, on the left. Write something in the "hea tältä sivustolta" (search in this page) form and press on "hea" (search). The form information is sent through the POST-method of the HTTP-protocol.
>
> Inspect the POST-request contents (on the top of the list). You'll see that the request was answered with a status code 302, which means that the server __redirects__ the browser, that is the browser was asked to go to the address defined in the response headers. The answer to a POST-request does not contain any HTML code which the browser could render for the user. Right after a POST request, the browser does automatically a GET call for the address contained in the __Location__ header of the POST reponse. Only the pages which are returned in answer to the request caused by this redirection are rendered to the users.
>
> Spend some time inspecting the HTTP protocol communication caused by the requests to other pages.

## Ruby on Rails foundamentals

In this course, we use Ruby on Rails application framework to implement Web applications.

Rails applications comply conform to the MVC model (or WebMVC, which is slightly different from the original MVC). The idea behind this model is that the application data and logic (Model), the visual composition (View) and the functions coordination (Controller) are separated into clearly defined parts. Almost all modern Web frameworks follow the MVC principle. In addition to MVC, modern Web applications can also be characterised by Multilayered architecture, Service-oriented architecture (SOA), and [Microservices architecture](http://martinfowler.com/articles/microservices.html) which is a hot topic nowadays.

Let's take a look at what happens when users go to a Web page which was implemented with Rails, suppose the URL be http://wad-ratebeer.herokuapp.com//breweries, that is the sample application page which we are going to make during our course. The page lists all the breweries which our sample application knows.

![mvc-kuva](http://www.cs.helsinki.fi/u/mluukkai/rails_mvc.png)

1. Once users write the URL into the browser address line, the browser makes an HTTP GET-request to the server wad-ratebeer.herokuapp.com

2. The Web server sowftwer which runs on the server (for instance, Apache) guides the request to the address, to the registered Rails application. The application finds out what application _controller_ is registered to take care of the GET calls for the resources of the breweries. This phase is called internal routing of the Rails application, which means that we are looking for "a way to handle the request".

3. When it finds out the right controller (in our example the controller which takes care of the breweries) and its method, the application calls the method. The method is given as parameter the data which had possibly been retrieven with the HTTP request. The controller takes care of the actions concerning the operation. The executions of these actions usually requires method calls to some _models_, which contain the application data and logic.

4. In our example, the controller asks from the model class which takes care of the breweries to load the list of all breweries from the database.

5. Once it has retrieven the list of all beers, the controller asks the beer list _view_ to render itself. 

6. The view renders itself, that is, the controller gets an HTML page listing all the beers.

7. The controller returns the HTML page to the Web server

8. The Web server returns the generated HTML page and its headers to the browser

In the MVC model, the so called models are usually objects and their status is saved in the database. The database is handled in an abstract way, meaning that rarely there is the need to write SQL language and database configurations at the level of the program code. The details make use of the Object Relational Mapping (ORM) library. The ORM used in Rails is called ActiveRecord, which works a bit differently than EclipseLink and Hibernate, which are based on the JPA standard and might be better know to Java users.

Rails is strongly based on the __convention over configuration__ principle, which means that Rails aims at minimizing the need of configuration by defining a group of conventions for file names and their hierarchic location, among the other things. We will see soon what the CoC principle means for the application developer, in practice. In fact, Rails does allow us to break the conventions, but it such case, the developer has to configure things by hand, to some extent.

Creating applications on Rails requires some kind of Ruby knowledge. Ruby is a dynamic-type object language which makes it possible for functional coding. In other words, Ruby's code is never translated. On the contrary, the interpreter executes the code command after command. Because there is no translator, syntax mistakes appear only when the code is running, which is in contrast with languages which have to be translated. Modern developer environments help us a little, providing us with some kind of "syntax proofreading" on the go, but the developer environment help is not hardly as good as it is in Java, for instance.

> ## Excercise 2: Introduction to Ruby
>
> Do/go through the following
> * http://tryruby.org/levels/1/challenges/0
> * http://www.ruby-lang.org/en/documentation/quickstart/

## Passing the course

The course structure is slightly different from the course standard of the department. In the course, we only make one application, the same application will be created both while you follow the course material and when you do the exercises. It's not possible to read the course material alone; while you go through the material, you have to develop your application, otherwise it will be impossible to pass the exercises. In other words, you have to follow the course assiduously all the time.

Every week, we publish a sample answer for the previous week after the deadline (on Sunday at 23.59). The following week, you can continue from your own application or use the sample answer from the previous week as a model.

A part of the exercises is compulsory, in practice, otherwise you stop for the week. Others are optional, and they regard the implementation of non-critical features. Because we expect to find a part of these features the following week, you'd better start from the sample answer or copy paste the useful parts from it, if you haven't done all the exercises of the week.

## Setting up Rails

Guidelines to set up rails at https://github.com/mluukkai/WebPalvelinohjelmointi2015/wiki/railsin-asennus

## Creating an application

In the course, we create a service for beer aficionados. They should be able to scan the breweries, the beers, the beer types, as well as to evaluate the beers they have tasted (that is, to give a vote to beers according to their personal taste). After week 7, the application should look more or less like the following: http://wad-ratebeer.herokuapp.com/

Rails provides various generators for application developers (see http://guides.rubyonrails.org/generators.html), and they make it easy to generate a functional file background which is almost ready.

We create a new Rails application with the generator new. Go to the suitable directory and create an application called ratebeer using the following command in the command line:

    rails new ratebeer

This generates the ratebeer directory which will contain our application.

Attention: once you continue with the course, it will be convenient that you make a git reposition of the created directory. This means that you should not place the application inside any other git reposition!

Go to the folder. Use the command <code>tree</code> to find out what the new generator brought about. Mind that the tree command is not installed by default in OSX. You can install it with [homebrew](http://brew.sh/) using the command <code>brew install tree</code>.

You see a hint of what happens below:

<pre>
  ratebeer  tree
.
|-- Gemfile
|-- Rakefile
|-- app
|   |-- assets
|   |-- controllers
|   |   |-- application_controller.rb
|   |   `-- concerns
|   |-- helpers
|   |   `-- application_helper.rb
|   |-- mailers
|   |-- models
|   |   `-- concerns
|   `-- views
|       `-- layouts
|           `-- application.html.erb
|-- bin
|-- config
|   |-- routes.rb
|-- db
|-- lib
|-- log
|-- public
|-- test
38 directories, 39 files
</pre>

The most important directory is **app** which contains the application code. Under the **config** directory you find data concerning the application configuration, among which routes.rb, which defines how the application handles the different HTTP requests. The database configurations have to be located in the directory **db**. Gemfile defines the libraries used by the application. With time, we'll get to know the application directory better.

The directory structure is an important part of Rails' Convention over Configuration principle. Each component (for instance, the controller which takes care of the breweries) has a strictly defined location, so that Rails can find the component without the help of the application developer, who does not need to explain in what directory and file the component is located.

Start the application with the following command

    rails server

You can also use the short form rails s

The command starts WEBrick HTTP server (see http://en.wikipedia.org/wiki/WEBrick) by default, which begins to execute the directory Rails application in the local computer (that is, in localhost) in the port 3000.

Attention: you may run into problems at this point, if you haven't installed a Javascript execution environment in your computer. One way to overcome the problem is adding the following line to the Gemfile (or you can remove the ashtag # before the line which is written ready in the file):

    gem 'therubyracer', platforms: :ruby

and execute the following command from the command line <code>bundle install</code>

Try to check that theapplication starts by browsing the following address [http://localhost:3000](http://localhost:3000) että sovellus on käynnissä.

ATTENTION: **While you read the document, the idea is that you execute in your application the same actions we do in our sample application**. A part of the things we implement are designed as homework – as it is with the following point. A part of the instructions is compulsory anyway, so that you may proceed with the material.

> ## Excercise 3
>
> We save our course application in the Github repository.
>
> Create a git repository for your application directory (that is, for you created with the command new) by executing the command <code>git init</code> in the directory.
>
> Create a repository in Github for the application and join it with the repository of your application directory as remote repository.
>
> You find git and Github guidelines at https://github.com/mluukkai/WebPalvelinohjelmointi2014/wiki/versionhallinta
>
> At the end of this document you find guidelines to make the final submission

Let's start with building our application. We want to start with breweries, that is:

* we create a database table for breweries
* we implement the funtionality to list all the breweries
* we implement the funtionality to add a new brewery
* while we are at it, we implement the functionality to modify the information and delete breweries

Conventionally, (almost) every 'thing' we save in Rails should have their own model class, controller class, as well as a set of files that make the view.

We create all this using the Rails' ready-made scaffold generator. Every brewery have a name (string) and foundation year (integer). Let's give the following command from the command line (making sure you are in the folder which contains the application):

    rails generate scaffold brewery name:string year:integer

A number of files is born. The most important are:
* app/models/Brewery.rb
* app/controllers/breweries_controller.rb
* app/views/breweries/index.html.erb
* app/views/breweries/show.html.erb
* There should be few other files in the views directory in addition to these

Rails' scaffold generator creates all the base for all required files, naming and placing them according to Rails conventions.

We created the code with the generator <code>rails g scaffold brewery name:string year:integer</code>. In the generator we wrote the thing we wanted to create – that is, the brewery database table and its features singularly (brewery). According to Rails name conventions, we have generated:
* a database table called breweries
* a controller called BreweriesController (the file breweries_controller.rb)
* the model, that is to say, the class Brewery which represents the brewery (file Brewery.rb)

At the beginning it might look a bit confusing to understand when we use singular and when plural, how files are named and what their position is. Conventions will gradually make roots and start to look a more logical.

If the application is not running already, let's start it again using the command <code>rails s</code>. Attention: rarely you need to restart your application in Rails. For instance, you don't need to run it again to modify and add code.

According to Rails conventions, all the beers are displayed at the address breweries. Let's go to this page, then:

    localhost:3000/breweries

This will cause an error, anyway:

```ruby
Migrations are pending; run 'bin/rake db:migrate RAILS_ENV=development' to resolve this issue.
```

The cause is that the *database migration* which takes care of creating adatabase recording the breweries is left unrun.

Scaffold execution creates a file with a peculiar name

    db/migrate/20150116173907_create_breweries.rb

This is the so called migration file, which contains guidelines to created the breweries database table. We can create the database table by executing the migration and giving the command <code>rake db:migrate</code>:

```ruby
➜  ratebeer  rake db:migrate
== 20150111131426 CreateBreweries: migrating ==================================
-- create_table(:breweries)
   -> 0.0011s
== 20150111131426 CreateBreweries: migrated (0.0012s) =========================
```

Tge database table which records the breweries is created now, and the application should be working.

Refresh the page which shows the breweries [http://localhost:3000/breweries](http://localhost:3000/breweries) and add three breweries using the database.

As we can see, Rails scaffold produced quite some ready-made functionality. The functionality which was produced with Scaffold is a good way to get started quickly. Scaffolds are no silver bullet, however. The bigger part of the ready-made functionality we created with scaffolds will be replaced with time the code we'll write on our own. During the course and starting from week 2, we'll create functionaly completely by hand, which means that the scaffolds' authomatically generated code will become familiar.

**Attention:** you can delete the files created with a Rails generator by using the command *destroy*:

    rails destroy scaffold brewery

If you have already executed the migration and you notice that the code created by the generator has to be destroyed, it's **extremely important** to cancel the migration with the command

    rake db:rollback

## The Console

One of the most important instruments for a Rails application developer is the Rails console. The console is an interactive command interpreter, which comes together with the application database. 

Make sure you are in the application folder and open the console with the command

    rails c

If the console gives an error containing the text "cannot load such file -- readline (LoadError), this should be corrected at least in Ubuntu 13.10 by setting up libreadline-dev and translating ruby again

    apt-get install libreadline-dev

    rbenv install 2.2.0

In case this does not help still, add the following line to the Gemfile

    gem 'rb-readline'
    
and run <code>bundle install</code> from the command line.

Also run yourself all the following commands:

```ruby
2.0.0-p451 :001 > Brewery.all
  Brewery Load (1.3ms)  SELECT "breweries".* FROM "breweries"
 => #<ActiveRecord::Relation [#<Brewery id: 1, name: "Schlenkerla", year: 1687, created_at: "2015-01-11 13:17:06", updated_at: "2015-01-11 13:17:06">, #<Brewery id: 2, name: "Koff", year: 1897, created_at: "2015-01-11 13:17:27", updated_at: "2015-01-11 13:17:27">, #<Brewery id: 3, name: "Malmgård", year: 2001, created_at: "2015-01-11 13:17:37", updated_at: "2015-01-11 13:17:37">]>
2.0.0-p451 :002 > Brewery.count
   (0.2ms)  SELECT COUNT(*) FROM "breweries"
 => 3
2.0.0-p451 :003 > Brewery
 => Brewery(id: integer, name: string, year: integer, created_at: datetime, updated_at: datetime)
2.0.0-p451 :004 >
```

The command <code>Brewery.all</code> shows all the breweries in the database. The console shows first the SQL request which was caused by database operation and then all the brewery objects retrieven from the database. The command <code>Brewery.count</code> shows the amount of breweries in the database.

We retrieve the connection breweries–database table through the class <code>Brewery</code>. Rails' scaffold generator created the class code.

If you have a look at what the class (that is, the file app/models/brewery.rb) looks like, you'll see that it contains only a hint of code:

```ruby
class Brewery < ActiveRecord::Base
end
```

As you saw from the previous console session, the class has also got the methods all and count. The class receives these two and a great amount of other methods from the class it __inherited__ which is called <code>ActiveRecord::Base</code>.

To tell it in the words of Rails guide (http://guides.rubyonrails.org/active_record_basics.html):

<blockquote>
Active Record is the M in MVC - the model - which is the layer of the system responsible for representing business data and logic. Active Record facilitates the creation and use of business objects whose data requires persistent storage to a database. It is an implementation of the Active Record pattern (https://en.wikipedia.org/wiki/Active_record_pattern) which itself is a description of an Object Relational Mapping system.
</blockquote>

In short, the idea behind ActiveRecord is that each database table (for instance, breweries) is connected to their own class (Brewery) in the code. The class provides methods as __class methods__, which are used to handle the database. When we retriev a line of data from the database (the information of one brewery), a class instance of it is created (that is, a Brewery object).

ActiveRecord classes have a double role, with the help of class methods (which are called in Ruby through the class name like <code>Brewery.all</code>) we handle a consistent part of the database operations, for instance, the database requests. The data saved in the database are mapped as instances of ActiveRecord classes.

Let's continue with our experiments on consoles. Let us create a new brewery:

    Brewery.new(name:"Stadin Panimo", year:1997)

As you see, Rails constructors are called in a different way than in Java, for instance. Notice that you don't have to use the brakets in constructor or method calls. The previous call could have been like

    Brewery.new name:"Stadin Panimo", year:1997

List all the breweries and check their amount using the methods <code>Brewery.all</code> and <code>Brewery.count</code>. Note that even though we created a new object it did not go into our database.

We save objects into our database in the following way:

    b = Brewery.new name:"Stadin Panimo", year:1997
    b.save

The object was created and saved into a variable <code>b</code>, then we called the object method <code>save</code>. Notice that you don't need (and you can not) define the variable type, because Ruby is a dynamically typed language!
Save is an object method inherited from ActiveRecord, which saves our object in the database – as you might have guessed.

You can also create and save the obect directly by using the class method create instead of new:

    Brewery.create name:"Weihenstephan", year:1042

When the object is created with the command <code>new</code>, we notice that the object contains fields whose value has not been defined:

```ruby
2.0.0-p451 :004 > b = Brewery.new(name:"Stadin Panimo", year:1997)
 => #<Brewery id: nil, name: "Stadin Panimo", year: 1997, created_at: nil, updated_at: nil>
```

After we have saved the object, these fields also receive their values:

```ruby
2.0.0-p451 :006 > b.save
2.0.0-p451 :007 > b
 => #<Brewery id: 4, name: "Stadin Panimo", year: 1997, created_at: "2015-01-11 13:21:37", updated_at: "2015-01-11 13:21:37">
```

As you might have guessed, object variables – or object fields – are connected to the columns of our database table. When an object is save into the database, the database genereates a primary key for the object automatically – the ID – as well as a couple of timestaps. The ID is unique for the beer that has just been created.

Check the situation now at the [www page](http://localhost:3000/breweries). The created breweries should be available on the page now.

The calls of the methods <code>new</code> create <code>create</code> look a bit particular

    Brewery.new name:"Stadin Panimo", year:1997

In this case we have made use of Ruby's liberal approach to brackets. Using brackets, the call would look like the following:

    Brewery.new( name:"Stadin Panimo", year:1997 )

The parameter has also got a particular format. It's an associative array indexed with symbols, that is to say, hash, see https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/rubyn_perusteita.md#hash-ja-symbolit

As you can see from the link above, hashes are defined with curly brakets, like

    { name:"Stadin Panimo", year:1997 }

In fact, the method call could be written

    Brewery.new( { name:"Stadin Panimo", year:1997 } )

The method parameter hash does not require curly brakets in all the cases, and sometimes they are left out. If the method has various parameters, curly brakets are sometimes required.

Attention: Ruby also have an optional syntax to define hashes, as it follows

    Brewery.new :name => "Stadin Panimo", :year => 1997

If you want to create structures or train to use Rails from console by hand, without long-term changes in the database, you can run your console in the sandbox form with the command:

```rails console --sandbox``` or shorter ```rails c -s```

## ActiveRecord search interface

ActiveRecord provides us with various options to make database searches programmatically, without writing SQL, see http://guides.rubyonrails.org/active_record_querying.html

Various exercises follow below, try them all from the console:

    Brewery.find 1       # returns an object whose ID is 1

    b = Brewery.find 2   # returns an object whose ID is 2 and stores it into the variable b
    b.year               # the value of the field year of the object saved in the variable b
    b.name               # the value of the field name of the object saved in the variable b

    Brewery.find_by name:"Koff"   # returns an object whose name is Koff

    Brewery.where name:"Koff" # returns a table with all the breweries whose name Koff

    Brewery.where year:1900..2000  # returns a table with the breweries esteblished between the years 1900-2000

    Brewery.where "year<1900"      # returns a table with the breweries esteblished before 1900

    b = Brewery.where name:"Koff"
    b.year                # the operation does not work because the method where returns a table which contains Koff

    t = Brewery.where name:"Koff"
    t.first.year          # t.first same as t[0]

More about Ruby tables at https://github.com/mluukkai/WebPalvelinohjelmointi2014/blob/master/web/rubyn_perusteita.md#taulukko

Note that we left the brakets out of all the method calls: <code>Brewery.find 1</code> means the same as <code>Brewery.find(1)</code>

## Underscore

The values returned by the previous method calls can be referenced with an underscore (the sign <code>_</code>). While you work with your console, this means that if you forget to store the outcome of a method call into a variable, you can retrieve that value using an underscore:


```ruby
2.0.0-p451 :013 > Brewery.where "year<1900"
```
we forgot to save outcome into a variable... we'll use an underscore
```ruby
2.0.0-p451 :014 > vanhat = _
2.0.0-p451 :015 > vanhat.count
 => 3
2.0.0-p451 :016 > vanhat.first
 => #<Brewery id: 1, name: "Schlenkerla", year: 1687, created_at: "2015-01-11 13:17:06", updated_at: "2015-01-11 13:17:06">
2.0.0-p451 :017 >
```

> ## Exercise 4
>
> Read http://guides.rubyonrails.org/active_record_basics.html#crud-reading-and-writing-data
>
> Make all the following with a Rails console:
> * Create a brewery which is called "Kumpulan panimo", which was establish in the year 2012<br />
> * Retrieve the brewery from the database using the method <code>find_by</code>, searching against brewery names<br />
> * Change the founding year of the brewery to 2015 <br />
> * Retrieve the brewery with <code>find_by</code> again and make sure the change of founding year happened properly<br />
> * Also make sure, that the value of the brewery field <code>updated_at</code> has changed, check that it is different from the <code>created at</code> value <br />
> * Delete the brewery<br />
> * Make sure the brewery was deleted

Let us take a look at the brewery code once more:

```ruby
class Brewery < ActiveRecord::Base
end
```

As you see, we have access to all the fields of our brewery objects with a "dot notation" and we can set them new values in the same way:

    b = Brewery.first         # retrieves the first added object from the database
    b.created_at              # shows the creation timestamp
    b.name = "Sinebrychoff"   # changes the field value, attention: the change happens in the database only when you save the object!

Rails' mistery is behind all this. In fact, Rails creates automatically set and get methods for all the fields of the database tables of the objects, so that the methods names are exactly the same as the database columns.

In our console, when we say <code>b.created_at</code> we really execute the <code>created_at</code> method which was automatically created in <code>Brewery</code> and which returns the value of the filed with the same name. Similarly, the command <code>b.name = "Sinebrychoff"</code> causes the execution of the <code>name=</code> method which changese the value of the <code>name</code> field which was automatically added to Brewery.

## Beers and the one-to-many connection

Let us expand our application to include beers. Every beer is associated with one brewery, and one brewery is tipically associated with many beers. After our expansion, the class design of our application domain (that is to say, of the persistent objects in the database which contain business logic) will look as the following
Laajennetaan sovellustamme seuraavaksi oluilla. Jokainen olut liittyy yhteen panimoon, ja panimoon luonnollisesti liittyy useita oluita. Laajennuksen jälkeen sovelluksemme domainin (eli bisneslogiikkaa sisältävien tietokantaan persistoitavien olioiden) luokkamalli näyttää seuraavalta:

![Breweries and beers](http://yuml.me/76d4d115)

Let us create a model, a controller and ready-made views for the beers using Rails' scaffold generator (the command is given in the command line):

    rails g scaffold Beer name:string style:string brewery_id:integer

in order to update the database, let us execute the database migretion by giving a command in the command line

    rake db:migrate

The following things have been created
* the database table beers which records beers
* the class Beer which is used to map the database has been created into the file app/models/beer.rb
* the controller BeersController which takes care of the beers has been created into the file app/controllers/beers_controller.rb
* view files have been created in the directory app/views/beers/

We created the fields <code>name</code> and <code>style</code> whose type is string and record the name and the style of the beers. We also created an integer field, <code>brewery_id</code>, which is supposed to act as __foreign key__ to connect a beer with the brewery.

In case it's needed, you can check the fields by writing the class name which corresponds to the database table into the console:

```ruby
irb(main):035:0> Beer
=> Beer(id: integer, name: string, style: string, brewery_id: integer, created_at: datetime, updated_at: datetime)
```

As you can see, every beer also has the fields which are added automatically to all ActiveRecord objects, such as <code>id</code>, <code>created_at</code> and <code>updated_at</code>.

Let us create by hand a couple beers and let us add link them to the brevery with the help of the <code>brewery_id</code> (attention: if your console was open already, you might have to give the command <code>reload!</code> to the console, which loads the beer program code so that the console may use it):

```ruby
2.0.0-p451 :043 > koff = Brewery.first
2.0.0-p451 :044 > Beer.create name:"iso 3", style:"Lager", brewery_id:koff.id
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">
2.0.0-p451 :045 > Beer.create name:"Karhu", style:"Lager", brewery_id:koff.id
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">
2.0.0-p451 :046 >
```

The beers __iso 3__ and __Karhu__ we created are linked to the brewery Koff. However, the connection does not work at the code level, yet.

In order to make it work, we have to modify the models in the following way:

```ruby
class Beer < ActiveRecord::Base
  belongs_to :brewery
end

class Brewery < ActiveRecord::Base
  has_many :beers
end
```

which means that a beer in connected to one brewery and one brewery has various beers. Pay attention to the singular and plural!

Let us go back to our console. If the console was open when you modified the code, use the command <code>reload!</code> to load the new version of the code so that the console can use it.

We want to try first how to tap into the brewery beers:

```ruby
2.0.0-p451 :047 > koff = Brewery.find_by name:"Koff"
2.0.0-p451 :048 > koff.beers.count
 => 2
2.0.0-p451 :049 > koff.beers
 => #<ActiveRecord::Associations::CollectionProxy [#<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">, #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">]>
```

<code>Brewery</code> objects now have the method <code>beers</code>, which returns the <code>Beer</code> objects of the brewery. Rails generated this method automatically when it saw the line <code>has_many :beers</code> in the class <code>Brewery</code>. In fact, the method <code>beers</code> does not return the brewery beers, directly, but an object typed <code>ActiveRecord::Associations::CollectionProxy</code> which represents a collection of the beers. We can use it to tap into the beer collection. The Proxy object acts as Ruby collections, which means that you have access to the singular brewery beers in the following way:

```ruby
2.0.0-p451 :050 > koff = Brewery.find_by name:"Koff"
2.0.0-p451 :051 > koff.beers.first
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">
2.0.0-p451 :052 > koff.beers.last
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">
2.0.0-p451 :053 > koff.beers[1]
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">
2.0.0-p451 :054 > koff.beers.to_a
 => [#<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">, #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">]
2.0.0-p451 :055 >
```

You can use the collection proxy as if it was a normal table or a collection to access singular collection elements. As we notice in the last point, you can retrieve a table of the beers which belong to the proxy with the method <code>koff.beers.to_a</code>.

When you parse the elements of a collection proxy with through an iterator such as <code>each</code>, this happens like when you go through a normal table using each:

```ruby
2.0.0-p451 :055 > koff.beers.each { |beer| puts beer.name }
iso 3
Karhu
```

At code level, tt easy also to access the breweries which are connected to beers:

```ruby
2.0.0-p451 :056 > bisse = Beer.first
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">
2.0.0-p451 :057 > bisse.brewery
 => #<Brewery id: 8, name: "Koff", year: 1897, created_at: "2015-01-11 13:51:26", updated_at: "2015-01-11 13:51:26">
2.0.0-p451 :058 >
```

The line <code>belongs_to :brewery</code> which was added to the <code>Beer</code> class adds the method <code>brewery</code> to beers, returning the brewery object which is connected in the database to that beer.

## Initializing the database

When you develop a program, it might be useful to generate "harcoded" data in the database.
The right place for such data is the file db/seeds.rb

Copy the following contents into the seeds.rb file of your application:

```ruby
b1 = Brewery.create name:"Koff", year:1897
b2 = Brewery.create name:"Malmgard", year:2001
b3 = Brewery.create name:"Weihenstephaner", year:1042

b1.beers.create name:"Iso 3", style:"Lager"
b1.beers.create name:"Karhu", style:"Lager"
b1.beers.create name:"Tuplahumala", style:"Lager"
b2.beers.create name:"Huvila Pale Ale", style:"Pale Ale"
b2.beers.create name:"X Porter", style:"Porter"
b3.beers.create name:"Hefezeizen", style:"Weizen"
b3.beers.create name:"Helles", style:"Lager"
```

The file contents is normal Rails code. You can execute the file with the command

    rake db:seed

Let us delete all the old date in the database by using the following command in the command line:

    rake db:reset

The command resets the database automatically, meaning that in addition to delete the old data, it does also run the contents of the file seeds.rb

Let us inspect the new data by hand from the console:

```ruby
2.0.0-p451 :058 > koff = Brewery.first
 => #<Brewery id: 8, name: "Koff", year: 1897, created_at: "2015-01-11 13:51:26", updated_at: "2015-01-11 13:51:26">
2.0.0-p451 :059 > koff.beers
 => #<ActiveRecord::Associations::CollectionProxy [#<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:09", updated_at: "2015-01-11 13:54:09">, #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2015-01-11 13:54:20", updated_at: "2015-01-11 13:54:20">]>
2.0.0-p451 :060 >
```

Let us create a new beer object. This time we use the new method, so that the object is not yet stored in the database:

```ruby
irb(main):050:0> b = Beer.new name:"Lite", style:"Lager"
=> #<Beer id: nil, name: "Lite", style: "Lager", brewery_id: nil, created_at: nil, updated_at: nil>
```

The beer is not in the database yet, and it is not connected to any brewery, either:

```ruby
2.0.0-p451 :061 > b.brewery
 => nil
```

You can connect a beer to a brewery in couple of ways too. We can set up the brewery field value by hand:

```ruby
2.0.0-p451 :062 > b.brewery = koff
2.0.0-p451 :063 > b
 => #<Beer id: nil, name: "Lite", style: "Lager", brewery_id: 8, created_at: nil, updated_at: nil>
2.0.0-p451 :064 >
```

As we see, the brewery ID becomes the brewery_id foreign key of the beer. The beer is not in the database, yet, the brewery does not know that the beer which was created belongs to it, in fact:

```ruby
2.0.0-p451 :064 > koff.reload
2.0.0-p451 :065 > koff.beers.include? b
 => false
```

Attention: just in case, we first called the database method <code>reload</code>, otherwise the object status would not have been updated, and also its beer list would have mirrored the time when the object was loaded.

We can get the beers stored in our well-known way using the command <code>save</code>. After this, the brewery will also know that the beer belongs to the brewery (once again, we have to reload the database first):

```ruby
2.0.0-p451 :066 > b.save
 => true
2.0.0-p451 :067 > koff.reload
2.0.0-p451 :068 > koff.beers.include? b
 => true
```

A more practical way is connecting the brewery to the set of beers by using the <code><<</code> operator:

```ruby
2.0.0-p451 :069 > b = Beer.new name:"IVB", style:"Lager"
 => #<Beer id: nil, name: "IVB", style: "Lager", brewery_id: nil, created_at: nil, updated_at: nil>
2.0.0-p451 :070 > koff.beers << b
```

Even though we don't explicitly use the method <code>save</code> here, the beer is saved into the database thanks to the operator <code><<</code>.

The third way is what we did in the <code>seeds.rb</code> file, where the <code>create</code> method is called straight to the beer collection of the brewery:

```ruby
2.0.0-p451 :071 > koff.beers.create name:"Extra Light Triple Brewed", style:"Lager"
```

> ## Exercise 5: Breweries and beers
>
>
> Work by hand on the console and implement the following:
> * Create the brewery Hartwall with three beers using the way above for all of them.
> * However we realise that Harwall has to be deleted because of its bad quality. Before deleting it, write down the Hartwall object ID
> * After deleting Hartwall, the database will keep on storing beer objects which belong to the brewery you have already deleted
> * Retrieve the orphan beers with the command <code>Beer.where suitableparameter</code>
> * Distroy the beers which were retrieven with the operation. You find information on how to go through a list at https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/rubyn_perusteita.md#taulukko

## The connection between controller and views

Let us analyze the ready-made controller of the brewery app/controller/breweries_controller.rb

The controller has been named according to Rails convensions in the plural form. According to Rails convetions, there are six methods in the controller. Let us inspect the <code>index</code> method first, which makes sure all the beers are displayed:

```ruby
class BreweriesController < ApplicationController
  def index
    @breweries = Brewery.all
  end
end
```

The method contains only one command

    @breweries = Brewery.all

this means that the method gives a list of all breweries to a variable called <code>@breweries</code> which forwards the brewery list to the view. After this, the index method renders the HTML page which was defined in the view template app/views/breweries/index.html.erb. The method does never point to the view template or contain a command to render it. Once again, it's about Rails' Convention Over Configuration principle: if we don't specify anything, the view template index.html.erb is rendered at the end of the index method.

We could also write down explicitely the rendering command:

```ruby
  def index
    @breweries = Brewery.all
    render :index   # renders the index.html.erb view template in the view/breweries directory
  end
```

The view templates (erb files) are made of HTML where has been added Ruby code.

Let us have a look at the view template which has been ready-generated, the file app/views/breweries/index.html.erb

```
<h1>Listing breweries</h1>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Year</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <% @breweries.each do |brewery| %>
      <tr>
        <td><%= brewery.name %></td>
        <td><%= brewery.year %></td>
        <td><%= link_to 'Show', brewery %></td>
        <td><%= link_to 'Edit', edit_brewery_path(brewery) %></td>
        <td><%= link_to 'Destroy', brewery, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Brewery', new_brewery_path %>
```

The view template creates a table where every brewery contained by the variable @breweries has their own line.

The Ruby code which was sqeezed into the view template is placed between the symbols <% %>. In turn, <% %> makes so that the value of the Ruby command is saved in the terminal.

We'll soon get a bit more familiar how tables are generated. First, let us add the information about the total number of breweries to our page (the erb template). You should add the following line at some point in the page, for instance right after the header contained by the h1 tags

```
<p> Number of breweries: <%= @breweries.count %> </p>
```

Go to the [page which lists the breweries](http://localhost:3000/breweries) with your browser and make sure the operation has worked out.

Let us go back to the code which makes the HTML table and let us take a closer look. Every brewery prints to their own line using Ruby's <code>each</code> iterator:

```
    <% @breweries.each do |brewery| %>
      <tr>
        <td><%= brewery.name %></td>
        <td><%= brewery.year %></td>
        <td><%= link_to 'Show', brewery %></td>
        <td><%= link_to 'Edit', edit_brewery_path(brewery) %></td>
        <td><%= link_to 'Destroy', brewery, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
```

We go through the brewery list which is stored in the variable ```@breweries``` with the help of the ```each``` iterator. (More information about each at https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/rubyn_perusteita.md#each). We refer to every singular brewery with the name <code>brewery</code> in the code chunk which is repeted. For every singular brewery we create a line contained by tr tags and with five columns. The first column is for the brewery name (```<%= brewery.name %>```) and the second is for the foundation year. The thirds column contains a link to a page which shows the brewery information. The code used in Ruby to generate a link is ```<%= link_to 'Show', brewery %>``` .

Actually, the above one is a short form of the following:

```
<%= link_to 'Show', brewery_path(brewery.id) %>
```
which generates HTML code which looks like the one below (the number below depends on the value of the ID field of the object in the table line):

```
<a href="/breweries/3">Show</a>
```

it is a link to the address "breweries/3". The first parameter of the command ```link_to``` is the name of the a-tag and the second is the link address.
 
The address itself is created in this longer form with the auxiliary method ```brewery_path(brewery.id)```which returns the path to the page of the brewery with the ID ```brewery.id```. The same thing is accomplished by the object itself, in our example the variable <code>brewery</code>, as a parameter to the method <code>link_to</code>.

We could also "hardcode" a command which generates a link using the form ```<%= link_to 'Show', "breweries/#{brewery.id}" %>```, but hardcoding is not usually a smart thing to do, and in this case even less.

What does ```"breweries/#{brewery.id}"``` mean? See https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/rubyn_perusteita.md#merkkijonot

> ## Excercise 6
> change the name of the brewery so that it can be clicked on and delete the show field and its link from the table

After the exercise, the pages showing the breweries of your application should look like the one below

![picture](http://www.cs.helsinki.fi/u/mluukkai/wadror/brewery-w1-0.png)

## Adding beers in the page of the brewery

Let us inspect how singular breweries are shown. The url to the brewery page is like "breweries/3", where the number is the brewery ID. The show method of the breweries controller let us access the brewery page:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]

  # other methods...

  def show
  end

end
```

The method does not contain any code! We notice however that at the beginning of the class defition there is the line

    before_action :set_brewery, only: [:show, :edit, :update, :destroy]

This means that we execute the code of the method <code>set_brewery</code> for each of the listed methods (show, edit, update ja destroy). The definition of the method is at the end of the class:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]

  # ...

  def show
  end

  # ...

  private
    def set_brewery
      @brewery = Brewery.find(params[:id])
    end

end
```

Before executing the method <code>show</code>, we execute the command

    @brewery = Brewery.find(params[:id])

this refers to the variable ```params```, which contains the information concerning the HTTP calls which are executed. The variable <code>params</code> is an associative table, or in other words, hash. Especially with the <code>:id</code> key (that is to say, ```params[:id]```), the variable value communicates the ID of the brewery to inspect in this case, that is the part which follows the slash in the page path breweries/xx.

We retrieve the brewery with the well-known command ```Brewery.find``` and we store it into the variable ```@brewery```.
At the end, the method <code>show</code> renders the view template ```show.html.erb```. The view template is generated again automatically based on Rails conventions: the ```show``` method of the brewery controller is executed, and towards the end, the view views/breweries/show.html.erb is rendered unless something else is defined in the code.

If we write it explicitely opened, the code to be executed together with the <code>show</code> method will look like the following:

```ruby
    @brewery = Brewery.find(params[:id])
    render :show
```

The code of the view template views/breweries/show.html.erb koodi is the following:

```
<p id="notice"><%= notice %></p>

<p>
  <strong>Name:</strong>
  <%= @brewery.name %>
</p>

<p>
  <strong>Year:</strong>
  <%= @brewery.year %>
</p>

<%= link_to 'Edit', edit_brewery_path(@brewery) %> |
<%= link_to 'Back', breweries_path %>
```

The part with the __notice__ ID at the beginning of the page is supposed to show the messages concerning the brewery creation or editing. We'll speak more about the topic later on.

> ## Excercise 7: Polishing the brewery page
>
>
> Add the information about the beer number of the brewery, with <code>@brewery.beers.count</code>
>
>
> Edit the ready-made page so that the brewery name becomes a header level h2 and the year is in cursive like "_Established_ _in_ _1897_".

Let us continue with our editing.

> ## Exercise 8: The beers on the brewery page
>
> Add a list of the brewery beers on the brewery page. First, add the following <code><%= @brewery.beers.to_a %></code> and see what happens.
>
> Next, list only the beer names using an each iterator:
>
> ```ruby
> <p>
>  <% @brewery.beers.each do |beer| %>
>    <%= beer.name %>
>  <% end %>
> </p>
> ```
> Change the beer names so that users can click on them; use the method <code>link_to</code> to implement this

After the exercise, you page should look like the following

![brewery and beers](http://www.cs.helsinki.fi/u/mluukkai/wadror/brewery-w1-1.png)

Let us improve a bit more the navigation of our application.

> ## Excercise 9
>
> In all the brewery pages, add a link to all the beer pages. Similarly add a link to all brewery pages, in all the beer pages. As an example, you can insert the link to a beer page with the command ```link_to 'list of beers', beers_path```

Let us try to get going the list with all the beers, finally.

> ## Excercise 10
>
> At this point, the brewery ID of the beers is shown on the page with all the beers
>
> Change the page so that beers show be the brewery name, not the ID. Also, users should be directed to the brewery page if they click on the name
>
> Modify the beer name too, so that users can click on it, and delete the show column
>
> Attention: if you encounter problems, you'd better read the following section!

The result should look like the following:

![Picture](https://github.com/mluukkai/WebPalvelinohjelmointi2014/raw/master/images/brewery-w1-3.png)

## nil

You might encounter the following error exception

![Picture](https://github.com/mluukkai/WebPalvelinohjelmointi2014/raw/master/images/brewery-w1-2.png)

The problem is actually the old nullpointer exeption, or a nilpointer exeption – as it is called in Ruby. Rails points out that you have tried to call the method name of nil (which is an object in Ruby!), and that method does not exist. The reason is most probably that your database contains beers which were not assigned a brewery or where their brewery has been deleted.

You can delete by hand the beers which cause troubles or with the help of your console. Using the console, you can retrieve orphan beers with the command:

    orvot_oluet = Beer.all.select{ |b| b.brewery.nil? }

then you can delete them with the help of an each iteretor

    orvot_oluet.each{ |orpo| orpo.delete }

because we only call one method for each iterated object, the previous command can be implemented in the following way too, where the syntax is quite particular:

	    orvot_oluet.each(&:delete)

## Reviewing: name convetions for paths and controllers

We have created database tables in our application for breweries and bears, as well as the controllers and views to handle them. Let us review again Rails name conventions, whenever it might take a while for the beginner to get used to them.

We created the brewery and its controllers and views with Rails' scaffold generator in the following way:

    rails g scaffold Brewery name:string year:integer

This produced:
* the database table <code>breweries</code>
* the controller <code>BreweriesController</code>, place in the directory app/controllers/
* the model <code>Brewery</code>, place in the directory app/models/
* a set of views in the directory app/views/breweries
* the migration file which takes care of shaping the database, in the directory /db/migrate

According to Rails conventions, the URL for the page of all breweries is breweries. On the contrary, the URLs for the pages of the singular breweries follow the pattern breweries/3, where the number is the brewery ID.

You don't need to write down the URLs in the view templates, because Rails provides us with path-helper methods (see http://guides.rubyonrails.org/routing.html#path-and-url-helpers), which will help you to generate the URLs.

The method <code>breweries_path</code> generates the URL of all breweries (or only the latter part of the URL, in fact). The URL of a singular brewery can be generated with the method <code>brewery_path(id)</code>, where the parameter is a link to the brewery ID.

We use various helpers with the method <code>link_to</code>. Link_to generates a link to the HTML page, that is an a-tag.

You can generate a link to the page of a <code>brewery</code> in the following way:

```ruby
    <%= link_to "link to the brewery #{brewery.name}", brewery_path(brewery.id) %>
```

The first parameter is the text of the link, and the second is the address.

When we create the link for a singular object page we use a shorter form:

```ruby
    <%= link_to "linkki panimoon #{brewery.name}", brewery %>
```

Now the second parameter is directly the object where the page leads to. When the second parameter is an object, Rails replaces it automatically by generating the real path with the code <code>brewery_path(brewery.id)</code>

The controllers which are generated automatically on Rails have six methods. The list of all breweries – that is, the address /breweries – is managed by the method <code>index</code>; the address of a singular brewery – for instance, /breweries/3 – is managed by the <code>show</code> method of the controller. Later on, we will get to know other methods of the controller.

The controllers methods render the template which shapes the HTML page which is returned to users. By default, the <code>index</code> method of the brewery controller renders the view template app/views/breweries/index.html.erb and the method <code>show</code> renders the view template app/views/breweries/show.html.erb.

This means that the controllers don't need to call the <code>render</code> method separately, if they render the default demplate. This means that the code

```ruby
class BreweriesController < ApplicationController
  def index
    @breweries = Brewery.all
    render :index
  end
```

works exactly the same as the following

```ruby
class BreweriesController < ApplicationController
  def index
    @breweries = Brewery.all
  end
```

The explicit <code>render</code> method is useful only when the controller renders a view different than the default one.

> ## Excercise 11
>
> Make a temporary change to the <code>index</code> method of the brewery controller. In the following way
>
> ```ruby
>  def index
>    @breweries = Brewery.all
>
>    render :panimot
>  end
> ```
>
> Try to see what happens when you go to the breweries page, at the address http://localhost:3000/breweries
>
> Add the file panimot.html.erb to the directory app/views/breweries and add something like the following
> ```ruby
>  panimoita <%= @breweries.count %>
> ```
>
> Go to the breweries page
>
> Take the method back to the original form.

## To the Internet

The easiest way to host applications nowadays is the PaaS (or Platform as a Service) service [Heroku](http://heroku.com). Heroku provides database and execution environment to Web applications. Heroku is for applications which use little capacity, but it is free.

You can deploy the application in Heroku very easily if the application folder has its own git repository.

Make a Heroku ID.

Create a ssh key and add it to Heroku at the page https://dashboard.heroku.com/account
* information on how to create an ssh key at  the Heroku Toolbelt page http://www.cs.helsinki.fi/group/kuje/compfac/ssh_avain.html, which contains a command line interface.

**Attention:** setting upt Heroku Toolbelt requires admin rights, so it will not succeed on the department computers. However, you can set up Heroku command line interfece on the department computers by doing:
* Remove the home directory file .netrc from the departnment computer
* Create an empty file with the same name in the fs home directory. The path for your fs home directory is `/home/tktl-csfs/fs/home/omakayttajatunnus` or `/home/tktl-csfs/fs2/home/omakayttajatunnus`. You can create an empty file for instance with the command `touch .netrc`.
* Create a symbolic link by executing the following command in the home directory `ln -s /home/tktl-csfs/fs2/home/omakayttajatunnus/.netrc`. (fs or fs2 according to where your home directory is)
* Make sure that you are in the home directory by typing the command `cd $HOME`
* Download and unpack Heroku client with the command `wget -qO- https://s3.amazonaws.com/assets.heroku.com/heroku-client/heroku-client.tgz | tar xz`
* Add Heroku client to PATH, with the command `echo 'export PATH="$HOME/heroku-client/bin:$PATH"' >> ~/.bashrc`
* Restart your terminal
* Make sure Heroku has been set up by running `heroku --version`, which should print something like `heroku-toolbelt/3.22.1 (x86_64-linux) ruby/2.2.0`.

Go to the root folder of your application and create a Heroku instance for it:

```ruby
➜  ratebeer git:(master) heroku create
Creating enigmatic-eyrie-1511... done, stack is cedar-14
https://enigmatic-eyrie-1511.herokuapp.com/ | https://git.heroku.com/enigmatic-eyrie-1511.git
Git remote heroku added
```

Type your Heroku ID when it's required.

The application URL will be  https://enigmatic-eyrie-1511.herokuapp.com/, in this case. The beginning of the application URL can be created by running **heroku create url_beginning**. 

**Note**, that there is nothing in the application root so far, at the address https://enigmatic-eyrie-1511.herokuapp.com/. The beers of our application will be found at https://enigmatic-eyrie-1511.herokuapp.com/beers and the breweries at https://enigmatic-eyrie-1511.herokuapp.com/breweries

Rails applications use the sqlite database by default, but the PostgreSQL database is in use in Heroku. The libraries used in Rails applications, that is to say Ruby terms gems, have been defined in the Germfile in the application root. In order to get started with PostgreSQL, we have to modify the Gemfile.

Delete a line

```ruby
gem 'sqlite3'
```

and add the following at some point of the file

```ruby
group :development, :test do
  gem 'sqlite3'
end

group :production do
   gem 'pg'
   gem 'rails_12factor'
end
```

as well as

```ruby
ruby '2.2.0'
```

Execute the command <code>bundle install</code> from the terminal, so that the changes are activated:

```ruby
mbp-18:viikko1 mluukkai$ bundle install
Fetching gem metadata from https://rubygems.org/..........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Using rake (10.1.1)
Using i18n (0.6.9)
...
Using uglifier (2.4.0)
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

If you encountered problems with the file pg config, and you are running on OS X 10.7, you should follow [this](http://stackoverflow.com/questions/19625487/impossible-to-install-pg-gem-on-my-mac-with-mavericks) directions, where PostreSQL is installed in the Applications directory and pg gem is set up separately. After this, bundle install should work normally.

If the link above did not help either, try the command <code>bundle install --without production</code>

Commit all the changes in the version management.

```ruby
➜  ratebeer git:(master) git add -A
➜  ratebeer git:(master) git commit -m"updated Gemfile for Heroku"
```

Now we are ready to start the application in Heroku. The application gets started with the operation <code>git push heroku master</code> from the command line

```ruby
➜  ratebeer git:(master) git push heroku master
Counting objects: 105, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (97/97), done.
Writing objects: 100% (105/105), 22.58 KiB | 0 bytes/s, done.
Total 105 (delta 10), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
...
remote: -----> Discovering process types
remote:        Procfile declares types -> (none)
remote:        Default types for Ruby  -> console, rake, web, worker
remote:
remote: -----> Compressing... done, 26.0MB
remote: -----> Launching... done, v6
remote:        https://enigmatic-eyrie-1511.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/enigmatic-eyrie-1511.git
 * [new branch]      master -> master
```

It looks like the application started without issues.

Use the browser to open the page which shows our brewery list: http://enigmatic-eyrie-1511.herokuapp.com/breweries

The result is an unpleasant error exception "We're sorry, but something went wrong.".

We can try to see what was the problem by inspecting Heroku logs, using the command <code>heroku logs</code>. A lot is printed, but if you look well, you'll easily find the cause:

<pre>
2015-01-11T14:55:55.010390+00:00 app[web.1]: Started GET "/breweries" for 87.92.42.254 at 2015-01-11 14:55:55 +0000
2015-01-11T14:55:55.108506+00:00 heroku[router]: at=info method=GET path="/breweries" host=enigmatic-eyrie-1511.herokuapp.com request_id=35dfd5f8-bafb-40e5-96f5-c0351e8f6e91 fwd="87.92.42.254" dyno=web.1 connect=1ms service=106ms status=500 bytes=1754
2015-01-11T14:55:55.102697+00:00 app[web.1]: PG::UndefinedTable: ERROR:  relation "breweries" does not exist
2015-01-11T14:55:55.102703+00:00 app[web.1]: LINE 5:                WHERE a.attrelid = '"breweries"'::regclass
2015-01-11T14:55:55.102705+00:00 app[web.1]:                                           ^
2015-01-11T14:55:55.102706+00:00 app[web.1]: :               SELECT a.attname, format_type(a.atttypid, a.atttypmod),
2015-01-11T14:55:55.102707+00:00 app[web.1]:                      pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
2015-01-11T14:55:55.102709+00:00 app[web.1]:                 FROM pg_attribute a LEFT JOIN pg_attrdef d
2015-01-11T14:55:55.102710+00:00 app[web.1]:                   ON a.attrelid = d.adrelid AND a.attnum = d.adnum
2015-01-11T14:55:55.102711+00:00 app[web.1]:                WHERE a.attrelid = '"breweries"'::regclass
2015-01-11T14:55:55.102712+00:00 app[web.1]:                  AND a.attnum > 0 AND NOT a.attisdropped
2015-01-11T14:55:55.102714+00:00 app[web.1]:                ORDER BY a.attnum
2015-01-11T14:55:55.102715+00:00 app[web.1]:                                          ^
</pre>

The problem is that the database has not been created (_PG::UndefinedTable: ERROR:  relation "breweries" does not exist_) In other words, we have to exectute the migrations to the application in Heroku. This works with the command <code>heroku run rake db:migrate</code>

And now the application works!

As you'll notice, the beers and the breweries in the database do not move to Heroku. If you want that the objects defined in the file <code>seed.rb</code> move to the database, you can execute the command

    heroku run rake db:seed

In the future, you'll always have to execute the migrations when you deploy the application to Heroku.

We can also open our Rails console with the application in Heroku, using the command


```ruby
➜  ratebeer git:(master) heroku run console
irb(main):001:0> Brewery.all
=> #<ActiveRecord::Relation [#<Brewery id: 1, name: "Koff", year: 1897, created_at: "2015-01-11 14:57:30", updated_at: "2015-01-11 14:57:30">, #<Brewery id: 2, name: "Malmgard", year: 2001, created_at: "2015-01-11 14:57:30", updated_at: "2015-01-11 14:57:30">, #<Brewery id: 3, name: "Weihenstephaner", year: 1042, created_at: "2015-01-11
```

This is a normal Rails console session, and you can use it to inspect the status of the application database which was deployed to Heroku.

## Handling associations and execution environments

As we mentioned in the previous section, the libraries used on Rails – the gems – are defined in the Gemfile which is located at the root of the application.

Before the changes we did in the previous section, the Gemfile used to look like the following (where the comments have been removed from the part on the top):

```ruby
source 'https://rubygems.org'

gem 'rails', '4.1.5'
gem 'sqlite3'
gem 'sass-rails', '~> 4.0.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'
```

The Gemfile has got a list of gams, which are used by the application. As we see, Rails itself is a gem. In some cases, the version to use or at least a suitable version number should be defined among the gems.

The associations are loaded at the address https://rubygems.org, using the program Bundler, see http://bundler.io/. You'll have to give the command <code>bundle install</code> from the command line. Bundler loads the gems and their associations from rubygem.org, and then the application is ready to use.

After <code>bundle install</code> is executed for the first time, the file <code>Gemfile.lock</code> is born and it defines precisely what gem versions are set up. It doesn't necessarily define precise versions for the Gemfile. After this, when you call <code>bundle install</code>, the versions which were defined in Gemfile.lock are set up. If you execute <code>bundle update</code>, you'll get the newer gems that you've uploaded in case of need and you create a new Gemfile.lock. You can see more information about Bundler at http://bundler.io/v1.5/rationale.html

By default, Rails provides us with three different execution environments
* development, the environment for the application development
* test, the environment to execute tests
* production, the environment which is used for the production

In each execution environment there is their own database and Rails also works slightly differently in each environment.

Normally, the programmer works so that the application is executed in the development environment. In this case, Rails offers error messages which help the work of the application developer. The application code is also uploaded again when it's run. Because of this, the application does not need to be restarted when the code is modified. On the contrary, the added code is always available to use for the application, automatically.

The application that has been deployed to Heroku starts to work in the production environment, which contains various features which optimize its performance and are different than the development environment. The error messages of the application are also different: instead of reporting the reason and place of the error, they only tell that "Something went wrong...".

We'll get to know the test environment in the course week 4.

Dirrent environments need different associations sometimes. For instance, when we execute the application in Heroku, the PostgreSQL database is used in the production environment. However, we use the sqlite3 database while we develop the applications. Same gems do not suit al the execution environments.

When we use different environments, gems can be defined with the group chunks in Gemfile. Below, the Heroku requirement of our application Gemfile after the changes:

```ruby
source 'https://rubygems.org'

gem 'rails', '4.1.5'

group :development, :test do
  gem 'sqlite3'
end

group :production do
   gem 'pg'
   gem 'rails_12factor'
end

gem 'sass-rails', '~> 4.0.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'
```

sqlite3 gem is used only in the development and test environments. Only the development environment makes use of the gems pg and rails_12 factor.

## Submitting the exercises

Commit all the changes you have made and push the code in Github. Add a link in Github readme file to the Heroku instance of your application. The contents which are generated by default in the readme file of your Rails application should be deleted.

Write down your exercise submission at the address http://wadrorstats2015.herokuapp.com/courses/1
