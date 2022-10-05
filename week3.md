You will continue to develop your application from the point you arrived at the end of week 2. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week [from the course repository](https://github.com/mluukkai/WebPalvelinohjelmointi2015/tree/master/malliv/viikko1). If you already got most of the previous week exercises done, it might be easier if you complement your own answer with the help of the material.

If you start working this week on the base of last week sample answer, copy the folder from the course repository (assuming you have alrady cloned it) and make a new repository of the folder with the application.

**Attention:** some Mac users have found problems with the pg-gem which Heroku needs. Gems are not needed locally and we defined to set them only in the production environment. If you have problems, you can set gems by adding the following expression to <code>bundle install</code>:

    bundle install --without production

This setting will be remembered later on, so a simple `bundle install` will be enough if you want to set up new dependences.

## Rails developer workflow

The optimal way to program on Rails is evidently different than Java. As a rule, when you are programming on Rails you should _avoid_ writing lots of code, and test it only later on when you go to the page which implements your code. A partial reason for this is the fact rails is a dinamically typed and interpreted language, which makes it impossible for even the best IDEs to check the program syntax. On the other hand, the fact that it is an interpreted language together with its console tools (the console itself and the debugger) makes it possible to test the functionality of smaller code chunks before they are put into the edited code file.

Let us take for example what we implemented last week, the implementation of the avarage value of beer ratings, and let us follow a natural Rails workflow.

Each beer contains a collection of ratings:


```ruby
class Beer < ActiveRecord::Base
  belongs_to :brewery
  has_many :ratings

end
```

We have to create the method <code>average</code> to the beer

```ruby
class Beer < ActiveRecord::Base
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    # code here
  end
end
```

If we wanted to do it in "Java's way" we could find the sum by going through all the ratings item after item and we could divide it by the number of items.

All Ruby's things which have something to do with collections (for instance tables and <code>hash_many</code> field) contain the auxiliary methods provided by the Enumerable module (ks. http://ruby-doc.org/core-2.1.0/Enumerable.html). Now we want to use the auxiliary methods to find out the avarage value.

When you write the code, you should _definitely_ use your console. In fact, the debugger would be an even better option than the console. The debugger will open the console straight where we are writing the code. Add the <code>byebug</code> command to the method call, which starts the debugger:

```ruby
class Beer < ActiveRecord::Base
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    byebug
  end
end
```

Open the console, read from the database an object containing the ratings, and call the method <code>average</code>:

```ruby
➜  ratebeer git:(master) ✗ rails c
Loading development environment (Rails 4.1.5)
2.0.0-p451 :001 > b = Beer.first
  Beer Load (0.2ms)  SELECT  "beers".* FROM "beers"   ORDER BY "beers"."id" ASC LIMIT 1
 => #<Beer id: 1, name: "Iso 3", style: "Lager", brewery_id: 1, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">
2.0.0-p451 :002 > b.average

[4, 13] in /Users/mluukkai/kurssirepot/ratebeer/app/models/beer.rb
    4:   belongs_to :brewery
    5:   has_many :ratings, dependent: :destroy
    6:
    7:   def average
    8:     byebug
=>  9:   end
   10:
   11: end
(byebug)
```

a debugger session will open inside the method. You will find all the information about that beer.

You have access to the object itself, by using the reference <code>self</code>

```ruby
(byebug) self
#<Beer id: 1, name: "Iso 3", style: "Lager", brewery_id: 1, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">
(byebug)
```

and you have access to the object field using either dot notation or simply the field name:

```ruby
(byebug) self.name
"Iso 3"
(byebug) style
"Lager"
(byebug)
```

Notice that if you want to modify the object field value inside a method, you have to use dot notation:

```ruby
  def metodi
    # seuraavat komennot tulostavat olion kentän name arvon
    puts self.name
    puts name

    # alustaa metodin sisälle muuttujan name ja antaa sille arvon
    name = "StrongBeer"

    # muuttaa olion kentän name arvoa
    self.name = "WeakBeer"
  end
```

This means you can refer to beer ratings from inside a beer method using the field name <code>ratings</code>:

```ruby
(byebug) ratings
  Rating Load (0.2ms)  SELECT "ratings".* FROM "ratings"  WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
#<ActiveRecord::Associations::CollectionProxy [#<Rating id: 1, score: 10, beer_id: 1, created_at: "2015-01-17 13:09:31", updated_at: "2015-01-17 13:09:31">, #<Rating id: 2, score: 21, beer_id: 1, created_at: "2015-01-17 13:09:33", updated_at: "2015-01-17 13:09:33">, #<Rating id: 3, score: 17, beer_id: 1, created_at: "2015-01-17 13:09:35", updated_at: "2015-01-17 13:09:35">, #<Rating id: 10, score: 22, beer_id: 1, created_at: "2015-01-17 15:51:02", updated_at: "2015-01-17 15:51:02">, #<Rating id: 11, score: 34, beer_id: 1, created_at: "2015-01-17 15:51:52", updated_at: "2015-01-17 15:51:52">]>
(byebug)
```

Take a look at the singular ratings:

```ruby
(byebug) ratings.first
#<Rating id: 1, score: 10, beer_id: 1, created_at: "2015-01-17 13:09:31", updated_at: "2015-01-17 13:09:31">
(byebug)
```

if you want to sum up ratings, you have to take the value of the <code>score</code> field from each rating object:

```ruby
(byebug) ratings.first.score
10
(byebug)
```

The <code>map</code> method of the enumerable module provides us with a way to make a new collection out of an old one. You can retrieve the items of the new collection from the original collection, by executing a mapping function to each of the orginal items.

If you use the name <code>r</code> to refer to the items of the original collection, the mapping function will be simple:

```ruby
(byebug) r = ratings.first
(byebug) r.score
10
(byebug)
```

You can try what <code>map</code> will do:

```ruby
(byebug) ratings.map { |r| r.score }
[10, 21, 17, 22, 34]
(byebug)
```

the mapping function is given to the code chunk between curly brakets which is given as parameter to the method <code>map</code>. The code chunk could also be defined using the <code>do end</code> pair. They both bring about the same result:

```ruby
(byebug) ratings.map do |r| r.score end
[10, 21, 17, 22, 34]
(byebug)
```

The map method makes use of the rating collection to help you build the values of the table ratings. You'll have to sum these values next.

Rails has added the method [sum](http://apidock.com/rails/Enumerable/sum) to all Enumberables. Try that out on the table you got with map.

```ruby
(byebug) ratings.map{ |r| r.score }.sum
104
(byebug)
```

In order to find out the avarage value, you will still have to devide the sum by the total number of items. Check out the <code>count</code> method, which can be used to find the total:

```ruby
(byebug) ratings.count
5
(byebug) ratings.map{ |r| r.score }.sum / ratings.count
```

and then form a oneliner to find the avarage value:

```ruby
(byebug) ratings.map{ |r| r.score }.sum / ratings.count
20
(byebug)
```

you will see that the result is rounded erroneously. The problem is evidently that both the devidend and divisor are integers. Change to float one of them. Before you do it, check how the method works to change integers to floats:

```ruby
(byebug) 1.to_f
1.0
(byebug)
```

If you don't know how to do something with Ruby, Google knows.

Think of a proper search term and you will have an answer quite certainly. You'll have to be careful though, and check out a couple of Google answers at least. You will have to make sure at least that the answer is for a version of both Ruby and Rails which are modern enough. For instance, a number of things in Rails 2 and 3 changed in the fourth version.

In Ruby and Rails there is typically a ready-made method or gem for almost everything, so you should always google or check the documentation instead of reinventing the wheel.

You can make now the final version of your code to find the avarage value:

```ruby
(byebug) ratings.map{ |r| r.score }.sum / ratings.count.to_f
20.8
(byebug)
```

The code is now ready tested, so you can copy it into a method:

```ruby
class Beer < ActiveRecord::Base
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    ratings.map{ |r| r.score }.sum / ratings.count.to_f
  end

end
```

Test the method now: exit from the debugger (typing c, that is, continuing the execution of the previously empty method till the end, _loading_ the new code, retrieving the object and executing the method:

```ruby
(byebug) c
2.0.0-p451 :006 > reload!
Reloading...
2.0.0-p451 :007 > b = Beer.first
2.0.0-p451 :008 > b.average
 => 20.8
2.0.0-p451 :009 >
```

The following test will reveal that there is something wrong, however:

```ruby
2.0.0-p451 :009 > b = Beer.last
 => #<Beer id: 17, name: "Hardcore IPA", style: "IPA", brewery_id: 4, created_at: "2015-01-17 17:04:50", updated_at: "2015-01-17 17:04:50">
2.0.0-p451 :010 > b.average
 => NaN
2.0.0-p451 :011 >
```

Hardcode IPA's rating avarage value is <code>NaN</code>. Go back to your debugger. Write the command <code>byebug</code> in the method for the avarage value, reload the code and call the method to the problematic object:

```ruby
[3, 12] in /Users/mluukkai/kurssirepot/ratebeer/app/models/beer.rb
    3:
    4:   belongs_to :brewery
    5:   has_many :ratings, dependent: :destroy
    6:
    7:   def average
    8:     byebug
=>  9:     ratings.map{ |r| r.score }.sum / ratings.count.to_f
   10:   end
   11:
   12: end
(byebug)
```

Evaluate the expression parts in the debugger:

```ruby
(byebug) ratings.map{ |r| r.score }.sum
0
(byebug) ratings.count.to_f
0.0
(byebug)
```

You are dividing integers by zero. See the result of the operation:

```ruby
(byebug) 0/0.0
NaN
(byebug)
```

In order to prevent a number is devided by zero, the method will have to handle the case separately:

```ruby
  def average
    return 0 if ratings.empty?
    ratings.map{ |r| r.score }.sum / ratings.count.to_f
  end
```

We are using an oneliner-if and the collection method <code>empty?</code> which evaluates whether the collection is empty. This is Ruby's way to check emptiness, wheras a Java user would write:

```ruby
  def average
    if ratings.count == 0
      return 0
    end
    ratings.map{ |r| r.score }.sum / ratings.count.to_f
  end
```

In any case, you should comply to the peculiar style of the language when you use a new one, expecially if you are dealing with a project where there are various different developers.

If using the debugger has not become a routine yet, rember to review last week [debugger material].
(https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#lis%C3%A4%C3%A4-rails-sovelluksen-debuggaamisesta)

## Users and sessions

Next, you will expand your application, so that users will be able to register a user name for themselves in the system.
You will soon modify the functionality so that each rating will belong to a registered user.

![mvc picture](http://yuml.me/ddc9b7c9)

Start with creating a user object which has a user name, and later add a password too.

Create a model, a view, and a controller for the user, with the command <code>rails g scaffold user username:string</code>

New users are created according Rails conventions with the form at the address <code>users/new</code>. It would be more natural however, if the address were <code>signup</code>. Add an optional route in routes.rb.

    get 'signup', to: 'users#new'

So the HTTP GET request to the signup address will also be handled with the help of the Users controller method <code>new</code>.

HTTP is a stateless protocol, which means that all the requests executed with an HTTP protocol are mutually dependent. If we want to implement a state in our Web application, for instance user registration, the information of a Web session state will have to be transmitted together with every browser HTTP request. The most common way to transmit state information is to use coockies, see http://en.wikipedia.org/wiki/HTTP_cookie

To tell a long story short, the idea behind coockies is the following: when the browser tries to access a Web site, the server can answer the browser and send a request to store a coockie. After that, the browser will add a coockie to all the HTTP requests for the Web site. A coockie is nothing else than a small amount of data, and the server can make use of the coockie data as it prefer to recognise the coockie browser.

Rails application developers will not have to work with coockies directly, because thanks to coockies, Rails has implemented __sessions__ which work at higher level and which are used by the application to "remember" browser information, such as the user identity, and the time of various HTTP requests, see
http://guides.rubyonrails.org/action_controller_overview.html#session.

Try to use sessions to remember what users did their last rating. In Rails applications, you have access to the session of a user (or of a browser) who did an HTTP request through the object <code>session</code> which works like a hash.

Store the rating session by adding the following chunk in the rating controller:

```ruby
  def create
    rating = Rating.create params.require(:rating).permit(:score, :beer_id)

    # we store the new rating in the session
    session[:last_rating] = "#{rating.beer.name} #{rating.score} points"

    redirect_to ratings_path
  end
```

add the following chunk of code to the application layout (tiedostoon app/views/layouts/application.html.erb) to make sure the rating will be seen in all pages:

```erb
<% if session[:last_rating].nil? %>
  <p>no ratings given</p>
<% else %>
  <p>previous rating: <%= session[:last_rating] %></p>
<% end %>
```
Try your application now. There is nothing stored in the session at the beginning, and the value of <code>session[:last_rating]</code> is <code>nil</code>, meaning that the page should say "no ratings given". Create a rating and see that it is saved in the session. Create a new rating again and see that it will overwrite the session data.

Open your application in an incognito window or another browser now. You will see that the session value is <code>nil</code> in the other browser. This means that the session is dependent on the browser.

## Signing in

The idea is implementing the registration functionality so that when users sign in, the <code>ID</code> of the corresponding <code>User</code> object is  saved in the session. When users sign out, the session is reset.

Attention: almost any kind of object can be saved in the session basically, and you could save in the session also the <code>User</code> object corresponding to the users who signed in. It is a best practice however (see http://guides.rubyonrails.org/security.html#session-guidelines) to store as little data as possible in the session (you can save up to 4kB of information in Rails sessions, by default). You should store right the amount you need to identify users who signed in, whereas their other information can be retrieven from the database if needed.

Create now a controller to sign in and out of your application. Tipically, you should follow Rails RASTful idea and conventional path names to implement the signing in functionality.

You can think of the session as something which is born when users sign up, and signing up can almost be considered as the same kind of resource as a beer, fo instance. Accondingly, the controller for signing up will be called <code>SessionsController</code>.

A session resource is anyway different than beers, for instance, because users either are or are not signed in, in a particular moment. Differently than beers, a user can have not many but maximum one session. Differently than beers, it does not make sense to have a list with all sessions. Routes should be written in singular and this can be at least done when you create a session routes into routes.rb with the <code>resource</code> command:

    resource :session, only: [:new, :create, :delete]

**Attention: make sure you write the routes.rb definition exactly in the form above, and not _resources_ as it is in the definitions of other paths.**

The sign up address is now **session/new**. The POST call to the address **session** executes the signing in, creating a session for the user. Users sign out when their session is distroyed, with the execution of a POST-delete call to the address **session**.

Create a controller for sessions (in the file app/controllers/sessions_controller.rb):

```ruby
class SessionsController < ApplicationController
  def new
    # render the signing up page
  end

  def create
    # retrieves from the database the user that matches the username
    user = User.find_by username: params[:username]
    # saves the user ID who signed up (if the user exists)
    session[:user_id] = user.id if not user.nil?
    #regirects the user to their own page
    redirect_to user
  end

  def destroy
    # resets the session
    session[:user_id] = nil
    # redericts the application to the main page
    redirect_to :root
  end
end
```

Notice that even though routes are written in the singular form now (**session** and *session/new**), the controller and the view directory spelling conventions should follow Rails normal plural form.

The code of the sign up page app/views/sessions/new.html.erb is below:

```erb
<h1>Sign in</h1>

<%= form_tag session_path do %>
  <%= text_field_tag :username, params[:username] %>
  <%= submit_tag "Log in" %>
<% end %>
```

Differently than the form you made for the ratings (review [the information from last week](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#lomake-ja-post)), the form you are going to create is not an object and you will create it with method <code>form_tag</code>, see http://guides.rubyonrails.org/form_helpers.html#dealing-with-basic-forms

Sending the form will cause an HTTP POST request to the session_path (notice the singular) that is the address **session**.

The method to handle the call takes the user ID which was saved in the <code>param</code> object and retrieves the corresponding user object from the database to save the object ID to the session, if the object exists. At the end, users are redirected to their own page. Once more, the controller code looks like below:

```ruby
  def create
    user = User.find_by username: params[:username]
    session[:user_id] = user.id if not user.nil?
    redirect_to user
  end
```

Attention 1: the command <code>redirect_to user</code> is a short form for <code>redirect_to user_path(user)</code>, see [week 1](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko1.md#kertausta-polkujen-ja-kontrollerien-niment%C3%A4konventiot).

Attention 2: Instead of the <code>if not</code> combination we can use <code>unless</code> in Ruby, and the second line of the method could have been written like

```ruby
  session[:user_id] = user.id unless user.nil?
```

Add the following code to the applicaiton layout to add the name of the signed in user to all the pages (you can delete now the session code we added for training in the last section):

```erb
<% if not session[:user_id].nil? %>
  <p><%= User.find(session[:user_id]).username %> signed in</p>
<% end %>
```

Users can now sign in the application at the address [http://localhost:3000/session/new](/session/new) (if users have been created in the application). Signing out does work work yet.

**ATTENTION:** if you see the error message <code>uninitialized constant SessionController></code> **make sure that you properly defined all the routes in routes.rb, like**

```ruby
  resource :session, only: [:new, :create, :delete]
```

> ## Exercise 1
>
> Implement all the changes above and make sure that signing in works out smoothly with an existing user ID (so that a signed in user is shown on the page.) You can create the ID at [http://localhost:3000/signup][/signup]). Even though signing out is not possible, you can sign in with a new ID and the old signing in will be overwritten.

## Controller and view auxiliary method

Making a database request in the view code is quite bad (as we did a moment ago with the code added to the application layout). Add the following method to the class <code>ApplicationController</code>:

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  # defines that the method current_user will be used also in the views
  helper_method :current_user

  def current_user
    return nil if session[:user_id].nil?
    User.find(session[:user_id])
  end
end
```

Because all the application controllers inherits the class <code>ApplicationController</code>, the method you are defining will be available to all the controllers. You also defined the method <code>current_user</code> as a helper method, which will be available not only to all the controllers but to all views, too. We can change the code added to the application layout in the following way:

```erb
<% if not current_user.nil? %>
  <p><%= current_user.username %> signed in</p>
<% end %>
```

The signing in address __sessions/new__ is annoying. Create another more natural address and call it __signin__. Define also a route to sign out. Implement these two things by adding the following code to routes.rb:

```ruby
  get 'signin', to: 'sessions#new'
  delete 'signout', to: 'sessions#destroy'
```

The signing in form can be found now at the address [http://localhost:3000/signin][/signin] and signing out is possible through the HTTP DELETE request to the address _signout_.

The following would also have worked

```ruby
  get 'signout', to: 'sessions#destroy'
```

so that singing out would happen through HTTP GET. It is not a best practice, though, that HTTP GET requests modify the application status. Stick to the the REST phylosophy conventions, which tell to destroy resources with HTTP DELETE requests. In this case, the resource is only a wider issue, users signing in.

Attention: if you see the error message <code>BCrypt::Errors::InvalidHash</code> when you try to sign in, you will have most probably forgotten to set up a password for the user. So set upt the password from your console, and try again.

> ## Exercise 2
>
> Modify the navigation bar in the application layout so that the bar will contain link to sign in and out. Notice that you should use HTTP DELETE for the signing out functionality, you find an example of this from the page which lists all users.
>
> In addition to the previous two, add a link to the page with all users to the navigation bar, as welll as the signed up user name, which should link to the user personal page. When the user is signed in, the bar should also show a link to create a new beer rating.
>
> Remember: you can see the defined routes and path methods using the command <code>rake routes</code> from the command line, or you can go to whatever unexisting application address, like [http://localhost:3000/wrong](http://localhost:3000/wrong)

At the end of the exercise, your application will look more or less like the following, if a user is signed in:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-1.png)

and if users are not signed in, it will be like below (notice also the sign up link now):

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-2.png)

## User ratings


Muutetaan seuraavaksi sovellusta siten, että reittaus kuuluu kirjautuneena olevalle käyttäjälle, eli tämän vaiheen jälkeen olioiden suhteen tulisi näyttää seuraavalta:

![kuva](http://yuml.me/ccdb3938)

The change will be nothing special at model level:

```ruby
class User < ActiveRecord::Base
  has_many :ratings   # a user have many ratings
end

class Rating < ActiveRecord::Base
  belongs_to :beer
  belongs_to :user   # a rating also belongs to one user

  def to_s
    "#{beer.name} #{score}"
  end
end
```

Our solution does not work like this, however. Because of the connections, you need a reference to the user ID as foreign key in your _rating_ database table. All the changes in Rails databases are implemented in Ruby code with the help of *migrations. Create a migration which adds a new column. Generate a migration file from the command line first, using the command:

    rails g migration AddUserIdToRatings

A file will appear in the directory _db/migrate_, with the following contents

```ruby
class AddUserIdToRatings < ActiveRecord::Migration
  def change
  end
end
```

Notice that the directory already contains its own migration files for all the database tables created. Each migration will contain the information about the change in the database as well as how to cancel such change eventually. If the migration is simple enough, so that Rails can derive the cancel operation based on the addition you executed, it will be enough for the migration if it contains only the method <code>change</code>. If the migration is more complex, you will have to define the methods <code>up</code> and <code>down</code> which define separately how to execute the migration and how to cancel it.


This time, the migration we need is simple:

```ruby
class AddUserIdToRatings < ActiveRecord::Migration
  def change
    add_column :ratings, :user_id, :integer
  end
end
```

In order to implement the migration change, execute the well-known commande <code>rake db:migrate</code> from the command line.

Migration are a vast field, and we will go back to them later on in the course. More information about migrations can be found at the address http://guides.rubyonrails.org/migrations.html

You will see from the console, that the connection between objects is implemented correctly:

```ruby
2.0.0-p451 :001 > u = User.first
2.0.0-p451 :002 > u.ratings
  Rating Load (0.4ms)  SELECT "ratings".* FROM "ratings"  WHERE "ratings"."user_id" = ?  [["user_id", 1]]
 => #<ActiveRecord::Associations::CollectionProxy []>
2.0.0-p451 :003 >
irb(main):003:0>
```

The rating you gave does not have a user at the moment:

```ruby
2.0.0-p451 :003 > r = Rating.first
2.0.0-p451 :004 > r.user
 => nil
2.0.0-p451 :005 >
```

You want to define the first user created as the user of all the existing ratings:

```ruby
2.0.0-p451 :005 > u = User.first
2.0.0-p451 :006 > Rating.all.each{ |r| u.ratings << r }
 => 14
2.0.0-p451 :008 >
```

**ATTENTION:** creating ratings from the user interface does not work properly at the moment, because the beers created in this way will not belong to any user. You will fix this soon.

> ## Exercise 3
>
>  Add the followin things in the user page, which is the view app/views/users/show.html.erb:
> – the user amount of ratings and their avarage (attention: use the module defined last week, <code>RatingAvarage</code> to find the avarage!)
> – a list of the user ratings and the possibility to delete them

The user page will look more or less like below (**ATTENTION:** we should have added information about the user rating avarage value, but we forgot...):

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-3.png)

After a rating is deleted, users will be redirected to the page with all ratings. The most natural thing to do would be returning to the user page after a deletion. To make this happen, implement the change below in the rating controller:


```ruby
  def destroy
    rating = Rating.find(params[:id])
    rating.delete
    redirect_to :back
  end
```

As you might guess, <code>redirect_to :back</code> makes so that users are redirected to the address of the link they clicked and that executed the HTTP DELETE request. 

Creating new ratings from the www-page does not work yet, because ratings are not connected to the signed in user. Change the rating controller so that a signed in user will be linked to the rating created.

```ruby
  def create
    rating = Rating.create params.require(:rating).permit(:score, :beer_id)
    current_user.ratings << rating
    redirect_to current_user
  end
```

Notice that <code>current_user</code> is the method we just added to the class <code>ApplicationController</code>, and it returns the signed up user by executing the code:

```ruby
  User.find(session[:user_id])
```

After we create a rating, the controller redirects the browser to the page of the signed in user.

> ## Exercise 4
>
> Change your application so that users can not delete ratings at the page with all ratings. Also, it should be possible to see the name of who created a rating, and there should be a link to their page.

The page with all ratings should look like below, after doing the exercise:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-4.png)

## Polishing the signing up

You application will give you pains at the moment, if users try to sign in with a user name which does not exist.

Change your application to dedirect users back to the sign in page, if signing in does not work out. Change the session controller like below:

```ruby
    def create
      user = User.find_by username: params[:username]
      if user.nil?
        redirect_to :back
      else
        session[:user_id] = user.id
        redirect_to user
      end
    end
```

change the code above to give messages to user, explaining what happened:

```ruby
    def create
      user = User.find_by username: params[:username]
      if user.nil?
        redirect_to :back, notice: "User #{params[:username]} does not exist!"
      else
        session[:user_id] = user.id
        redirect_to user, notice: "Welcome back!"
      end
    end
```

If you want that your message will be seen in the sign up page, add the element below to the view ```app/views/sessions/new.html.erb```:

```erb
<p id="notice"><%= notice %></p>
```

The element is ready in the user page template (unless you have deleted it by mistake), so the message will work there.

The __flashes__ are the messages connected to redirections and that have to be remembered for the next HTTP request and that have to be shown on the page when needed. They are implemented in Rails thanks to the sessions, more about this at http://guides.rubyonrails.org/action_controller_overview.html#the-flash

## Olioiden kenttien validointi

Our application has a small problem now: it is possible to create many users with the same user name. In the <code>create</code> method of our user controller there should be the functionality to check that the <code>username</code> is not used.

A versitile mechanism to validate object fields comes built-in with Rails, see http://guides.rubyonrails.org/active_record_validations.html and http://apidock.com/rails/ActiveModel/Validations/ClassMethods

Validating the unicity of user names is simple, and you just need to add a short chunk of code to your User class:

```ruby
class User < ActiveRecord::Base
  include RatingAverage

  validates :username, uniqueness: true

  has_many :ratings
end
```

If you try to create a user which exists already, you will see that Rails will be able to generate an appropriate error message automatically.

Rails (or more properly, ActiveRecord) executes the object validations right before trying to store the object in the database, for instance with the operations <code>create</code> or <code>save</code>. If the validation fails, the object will not be stored.

Add other validations right away too. Add the requirement that user ID length should be at least three-character long, implementing the following line to the User class:

```ruby
  validates :username, length: { minimum: 3 }
```

If various validation rules concern the same attribute, they can all be connected under one <code>validates :attribuutti</code> call:

```ruby
class User < ActiveRecord::Base
  include RatingAverage

  validates :username, uniqueness: true,
                       length: { minimum: 3 }

  has_many :ratings
end
```

The controllers created with Rails scaffold generator are implemented in a way that if the validation works out and the object is stored in the database, the browser is redirected to the page of the object created. If the validation does not work out, it shows again the form to create objects and the error message is rendered in the page with the form.

How can the controller know, whether the validation worked out? The validation happens when it tries to save something in the database. If the controller saves the object with the method <code>save</code>, the controller can test the method return value whether the validation worked out or not:

```ruby
  @user = User.new(parametrit)
  if @user.save
  	# the validation worked out, so the browser is redirected to the appropriate page
  else
    # the validation did not work out, so the view template :new is rendered
  end
```

The controller generated by scaffold is a bit more complex:


```ruby
  def create
    @user = User.new(user_params)

    respond_to do |format|
      if @user.save
        format.html { redirect_to @user, notice: 'User was successfully created.' }
        format.json { render action: 'show', status: :created, location: @user }
      else
        format.html { render action: 'new' }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end
```

First of all, where does <code>user_params</code> come from which is used as parameter to create the object? You'll see that the following method is defined at the bottom of the file:

```ruby
    def user_params
      params.require(:user).permit(:username)
    end
```

so the first line of the method <code>create</code> is the same as

```ruby
   @user = User.new(params.require(:user).permit(:username))
```

So what does <code>respond_to</code> which defines a method? If the object is created with a normal form and if the browser receives an HTML answer, the functionality will be this, by default:

```ruby
 if @user.save
  redirect_to @user, notice: 'User was successfully created.'
 else
   render action: 'new'
 end
```

the code chunk of the entry <code>format.html</code> (which technically is a method call) is executed in the code chunk of the command <code>respond_to</code> (which is also a method). However, if the HTTP POST call to create a user object was made so to expect an answer in json-form (which would happen for instance if the request was done in Javascript for a different service or Web page), it would execute the doce of <code>format.json</code>. The syntax could look strange at first, but you'll soon get acquainted with it.

Continue with your validations in pairs. Define that beer ratings have to be integers between 1-50:

```ruby
class Rating < ActiveRecord::Base
  belongs_to :beer
  belongs_to :user

  validates :score, numericality: { greater_than_or_equal_to: 1,
                                    less_than_or_equal_to: 50,
                                    only_integer: true }

   # ...
end
```

If users create inappropriate ratings, they won't be saved any more. You will notice however, that users won't receive any error message. The problem is that you created the form by hand, it does not contain error reports like the forms generated automatically with scaffold and the controller will never check whether the validation worked out.

Change first the rating controller method <code>create</code> so that it renders again the form to create ratings if the validation failed:

```ruby
  def create
    @rating = Rating.new params.require(:rating).permit(:score, :beer_id)

    if @rating.save
      current_user.ratings << @rating
      redirect_to user_path current_user
    else
      @beers = Beer.all
      render :new
    end
  end
```

The method creates a Rating object with the command <code>new</code> first, and this is not saved in the database yet. Then, it executes the database saving process with the method <code>save</code>. Before saving, it validates the object, and if this fails, the method returns false, and the object will not be saved in the database. In such case, the new view template will be rendered. Rendering the view template requires that the beer list is stored in the variable <code>@beers</code>.

As you try to create an erroneous rating, users remain in the view showing the form (which technically is rendered again after the POST call). However there is no error message, yet.

When the validation fails, Rails validator saves the error messages in the field <code>@rating.errors</code> (which belongs to the object <code>@ratings</code>.

Change the form to show the value of <code>@rating.errors</code>, if the field contains something:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
  	<%= @rating.errors.inspect %>
  <% end %>

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```

If you create an erroneous rating now, you will be able to find out the reason from the object stored in the field <code>@rating.errors</code>.

Take the view template views/users/_form.html.erb as example and change your form (views/ratings/new.html.erb) like below:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@rating.errors.count, "error") %> prohibited rating from being saved:</h2>

      <ul>
      <% @rating.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```
When it finds valitation errors, the view template renders all the error message contained in <code>@rating.errors.full_messages</code>.

**Attention:** when the validation fails the redirection is not executed (why doesn't it work here?), but the view template is rendered instead, as it usually happens when you execute the <code>new</code> method.

You find help for the next exercises at
http://guides.rubyonrails.org/active_record_validations.html ja http://apidock.com/rails/ActiveModel/Validations/ClassMethods

> ## Exercise 5
>
> Add the following validations to your program
> * whether the beer and brewery names are not empty
> * the brewery founding year is an integer between 1042-2015
> * the length of the user ID – that is, the username attribute of the User class – is 3 – 15 characters

> ## Exercise 6
>
> ### doing this exercise is not essential to continue with the rest of the week material, so you should not get stuck with it. You can also do this exercise only after you have done the rest of the week.
>
> Improve exercise 5 validations so that the brewery founding year is an integer which is at least 1042 and maximum the current year. You cannot hard-code the year.
>
> Notice that this will not work as you want:
>
>   validates :year, numericality: { less_than_or_equal_to: Time.now.year }
>
> <code>Time.now.year</code> is evaluated when the program loads the class code. If the program starts to run at the end of 2015, in 2016 users will not to register a 2016 brewery, because when the program started it evaluated 2015 as the year upper limit and the validation will fail.>
> A possible way is acting on the validation method definition http://guides.rubyonrails.org/active_record_validations.html#custom-methods
>
> You could find even an even shorter solution in terms of code, a hint could be lambda/Proc/whatever...


## Connections many to many

One beer has many ratings, and a rating has always one user, which means a beer has a user who made many ratings. Similarly, a user has many ratings and a rating has one beer. This means that a user has many rated beers. The connection between beers and users is **many to many** where the rating table acts as a union table.

We can create this many to many connection at code level easily using the way we got acquainted with [last week](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#olioiden-ep%C3%A4suora-yhteys), the **has_many through** connection:

```ruby
class Beer < ActiveRecord::Base
  include RatingAverage

  belongs_to :brewery
  has_many :ratings, dependent: :destroy
  has_many :users, through: :ratings

  # ...
end

class User < ActiveRecord::Base
  include RatingAverage

  has_many :ratings
  has_many :beers, through: :ratings

  # ...
end
```

And the many to many connection will work for users:

```ruby
2.0.0-p451 :009 > User.first.beers
 => #<ActiveRecord::Associations::CollectionProxy [#<Beer id: 1, name: "Iso 3", style: "Lager", brewery_id: 1, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">, #<Beer id: 1, name: "Iso 3", style: "Lager", brewery_id: 1, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">, #<Beer id: 11, name: "Punk IPA", style: "IPA", brewery_id: 4, created_at: "2015-01-17 13:12:12", updated_at: "2015-01-17 13:12:12">, #<Beer id: 11, name: "Punk IPA", style: "IPA", brewery_id: 4, created_at: "2015-01-17 13:12:12", updated_at: "2015-01-17 13:12:12">, #<Beer id: 11, name: "Punk IPA", style: "IPA", brewery_id: 4, created_at: "2015-01-17 13:12:12", updated_at: "2015-01-17 13:12:12">, #<Beer id: 12, name: "Nanny State", style: "lowalcohol", brewery_id: 4, created_at: "2015-01-17 13:12:27", updated_at: "2015-01-17 13:12:52">, #<Beer id: 12, name: "Nanny State", style: "lowalcohol", brewery_id: 4, created_at: "2015-01-17 13:12:27", updated_at: "2015-01-17 13:12:52">, #<Beer id: 7, name: "Helles", style: "Lager", brewery_id: 3, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">, #<Beer id: 1, name: "Iso 3", style: "Lager", brewery_id: 1, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">, #<Beer id: 4, name: "Huvila Pale Ale", style: "Pale Ale", brewery_id: 2, created_at: "2015-01-11 14:29:25", updated_at: "2015-01-11 14:29:25">, ...]>
2.0.0-p451 :011 >
```

and for beers:

```ruby
2.0.0-p451 :011 > Beer.first.users
 => #<ActiveRecord::Associations::CollectionProxy [#<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 2, username: "pekka", created_at: "2015-01-24 16:51:42", updated_at: "2015-01-24 16:51:42">]>
2.0.0-p451 :013 >
irb(main):010:0>
```

It looks like working, but it seems stupid to refer to users who rated a beer with the name <code>users</code>. A more natural to refer to users who rated a beer could be perharps <code>raters</code>. This works if you change the connection definition in the following way:

```ruby
has_many :raters, through: :ratings, source: :user
```

By default, <code>has_many</code> will look for a table whose name is the same as its parameter. Because <code>raters</code> is not the name of the connection destination, this has to be defined apart using the _source_ option.

The new name of our connection will work now:

```ruby
2.0.0-p451 :014 > b = Beer.first
2.0.0-p451 :015 > b.raters
 => #<ActiveRecord::Associations::CollectionProxy [#<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 2, username: "pekka", created_at: "2015-01-24 16:51:42", updated_at: "2015-01-24 16:51:42">]>
2.0.0-p451 :016 >
```

Because the same user can create various ratings of the same beer, the user will be seen various times among the beer raters. If you want that one rater is seen only once, you can do like thi:

```ruby
irb(main):013:0> b.raters.uniq
2.0.0-p451 :016 > b.raters.uniq
 => [#<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 14:20:10">, #<User id: 2, username: "pekka", created_at: "2015-01-24 16:51:42", updated_at: "2015-01-24 16:51:42">]
2.0.0-p451 :017 >
```

It would also be possible to define that by default beer <code>raters</code> should return singular users only once. You could implement this by setting __scope__ to the <code>has_many</code> attribute, limiting the sets of the users that are shown to refer to the association:

```ruby
class Beer < ActiveRecord::Base
  #…

  has_many :raters, -> { uniq }, through: :ratings, source: :user

  #…
end
```

More about defining connections in normal and more complicated circumstances at http://guides.rubyonrails.org/association_basics.html

Attention: there is also another way to create many-to-many connections on Rails, <code>has_and_belongs_to_many</code>, see http://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association which might be useful if the only purpose of your connection table is to establish a connection.

However the trend is to use the has_many through combination and explicitely defined connection tables, instead of the method has_and_belongs_to_many (because of its various issues). Among the others, Chad Fowler suggests that users should avoid using has_and_belongs_to_many in his book [Rails recepies](http://pragprog.com/book/rr2/rails-recipes), Obie Fernandez gives the same suggestion in his autoritative work [Rails 4 Way](https://leanpub.com/tr4w)

> ## Exercises 7-8: Beer clubs
>
> ### This and the following exercise are not essential to continue with the week material. You can also do them after the other exercises of the week.
>
> Extend the system so that users can be beer_clubs_members.
>
> Use scaffold to create <code>BeerClub</code> with the attributes <code>name</code> (a string) <code>founded</code> (an integer) and <code>city</code> (a string)
>
> Create a many-to-many connection between <code>BeerClub</code> and <code>User</code>. Create a connection table for this, the <code>Membership</code> model, with the foreign keys to the objects <code>User</code> and <code>BeerClub</Code> as attributes (they are <code>beer_club_id</code> and <code>user_id</code>, notice how the capital letter in the middle of the object name becomes an underscore!). Use scaffold for this model too.
>
> At this point, you can implement the functionality to add members to beer clubs as the current beer rating functionality – adding the link "join a club" to the navigation bar, which adds registered users to one of the beer clubs in the list.
>
> List all the members in the beer club page, and similarly, list all the beer clubs that a person belongs to on his page. Add a link to the list with all beer clubs in the navigation bar.
>
> You don't need to implement the functionality to remove a user from a beer club, so far.

> # Exercise 9
>
> Refine the previous exercise so that users cannot be assigned various times to the same beer club.

The following two pictures will help you understand what your application should look like after exercises 7–9.

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-5.png)

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-6.png)

## Password

Modify your application again so that users will have a password. Because of information security issues, you should avoiding storing passwords in the database. In the database, we only store the password hash, which was calculated with a parallel function. Let's implement a migration for this:

    rails g migration AddPasswordDigestToUser

the code of your migration (see the folder db/migrate) should be like below:

```ruby
class AddPasswordDigestToUser < ActiveRecord::Migration
  def change
    add_column :users, :password_digest, :string
  end
end
```

notice that the name of the added column should be <code>password_digest</code>.

Add the code below to <code>User</code>:

```ruby
class User < ActiveRecord::Base
  include RatingAverage

  has_secure_password

  # ...
end
```

<code>has_secure_password</code> (see http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) provides the class with the functionality to save the password _hash_ into the database and the user can be identified when needed.

Rails uses the <code>bcrypt-ruby</code> gem to store the hash. Get started with it by adding the following line to the Gemfile

    gem 'bcrypt', '~> 3.1.7'

After this, run <code>bundle install</code> from the command line to set up the gem.

**Attention:** if you are using an older Rails version, you will have to take a different gem:

    gem 'bcrypt-ruby', '~> 3.1.2'


Try out the new functionality from the console now (you will have to restart the console to set up the new gem).

__Also, remember to execute the migration!__

The password functionality <code>has_secure_password</code> adds the attributes <code>password</code> and <code>password_confirmation</code> to the object. The idea is that to place password and its confirmation in these attributes. When the object is stored in the database – for instance with the <code>save</code> method call – the hash which is stored in the database as value in the column <code>password_digest</code> will be calculated. The proper password, the attribute of <code>password</code> is not stored in the database, and only its representation is recorded by the object.


Storing a password for our user

```ruby
2.0.0-p451 :004 > u = User.first
2.0.0-p451 :005 > u.password = "salainen"
2.0.0-p451 :006 > u.password_confirmation = "salainen"
2.0.0-p451 :007 > u.save
   (0.2ms)  begin transaction
  User Exists (0.3ms)  SELECT  1 AS one FROM "users"  WHERE ("users"."username" = 'mluukkai' AND "users"."id" != 1) LIMIT 1
Binary data inserted for `string` type on column `password_digest`
  SQL (0.4ms)  UPDATE "users" SET "password_digest" = ?, "updated_at" = ? WHERE "users"."id" = 1  [["password_digest", "$2a$10$DZaWkl73GurTQG3ilOVz9./X6jGT49ngZb3Q9ZCF3YjVvXPrl1JLm"], ["updated_at", "2015-01-24 18:28:24.069587"]]
   (0.8ms)  commit transaction
 => true
2.0.0-p451 :008 >
```

If the command <code>u.password = "salainen"</code> causes the error message <code>NoMethodError: undefined method `password_digest=' for ...</code>, remember to restart the console and execute the migration!

The authentication happens thanks to the method <code>authenticate</code> which was added to the <code>User</code> object:

```ruby
2.0.0-p451 :008 > u.authenticate "salainen"
 => #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 18:28:24", password_digest: "$2a$10$DZaWkl73GurTQG3ilOVz9./X6jGT49ngZb3Q9ZCF3Yj...">
2.0.0-p451 :009 > u.authenticate "wrong"
 => false
2.0.0-p451 :010 >
```

the method <code>authenticate</code> returns <code>false</code> if the password given as parameter is wrong. If the password is right. the method returns the object itself.

Implement the functionality to check the password when users sign in. Change first the sing-in page (app/views/sessions/new.html.erb) so that in addition to ask for the user name, it asks for the password (notice that the type of the form field is *password_field*, which only shows stars instead of the written password):

```erb
<h1>Sign in</h1>

<p id="notice"><%= notice %></p>

<%= form_tag session_path do %>
  username <%= text_field_tag :username, params[:username] %>
  password <%= password_field_tag :password, params[:password] %>
  <%= submit_tag "Log in" %>
<% end %>
```

and change the sessions-controller so that it uses the method <code>authentificate</code> to verify whether the form password is right.

```ruby
    def create
      user = User.find_by username: params[:username]
      if user && user.authenticate(params[:password])
        session[:user_id] = user.id
        redirect_to user_path(user), notice: "Welcome back!"
      else
        redirect_to :back, notice: "Username and/or password mismatch"
      end
    end
```

Try out whether signing in works (**attention: you will have to restart rails server to set up bcrypt-gem**). So far, signing in only works for the users whose passwords were added from the console by hand.

Add a password input field to the user creation (that is to say, to the view view/users/_form.html.erb):

```erb
  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password %>
  </div>
  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation  %>
  </div>
```

The controller auxiliary method <code>user_params</code> which is in charge of creating users has to be modified so that it can retrieve the form password and its authentification:

```erb
 def user_params
     params.require(:user).permit(:username, :password, :password_confirmation)
  end
```

Try to see what happens if you give an erroneous value to the password confirmation.

> ## Exercise 10
>
> Implement a user validation to the class, to make sure the password is at least four characters and it contains at least one capital letter and one figure.

**Attention**: you can test Ruby's regular expressions with the Rubular application: http://rubular.com/


## Deleting only one's own ratings

At this point, whatever user is able to delete the ratings of anyone else. Modify your application so that users can remove only their own ratings. It will be simple if it's verified in the rating controller:

```ruby
  def destroy
    rating = Rating.find params[:id]
    rating.delete if current_user == rating.user
    redirect_to :back
  end
```

implement the remove operation only if the ```current_user``` is the same as the rating user.

There is no reason actually why you should show the rating remove link in other pages than in the personal page of the signep up user. So change the user show page like below:

```erb
  <ul>
    <% @user.ratings.each do |rating| %>
      <li>
        <%= rating %>
        <% if @user == current_user %>
            <%= link_to 'delete', rating, method: :delete, data: { confirm: 'Are you sure?' } %>
        <% end %>
      </li>
    <% end %>
  </ul>
```

Notice that simply removing the **delete** link does not prevent deleting other users ratings, because it is extremely easy to make an HTTP DELETE operation to the urls of a massive number of ratings. Therefore, it is essential to check the identity of the signed-in user in the control method which executes the deletion.

> ## Exercise 11
>
> The list with all users [http://localhost:3000/users](http://localhost:3000/users) contains the links **destroy** and **edit** now, which can be used to destroy users and to edit their information. Remove both links and add them to the user page (you can move only delete, actually, because edit is already there).
>
> Show the edit and distroy links only to the personal page of the signed-in user. Change also the <code>update</code> and <code>destroy</code> methods of the User controller so that updating or deleting the object information can only be done by the signed-in user.

> ## Exercise 12
>
> Create a new user name, make the user sign in, and then distroy the user. Deleting the user name will cause an annoying error. **You will get over it by deleting the browser cookies.** Try to think what caused the error and fix the bug in the application too, so that deleting a user would not bring about an error situation.

> ## Exercise 13
>
> Laajenna vielä sovellusta siten, että käyttäjän tuhoutuessa käyttäjän tekemät reittaukset tuhoutuvat automaattisesti. Ks. https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#orvot-oliot
>
> Jos teit tehtävät 7-8 eli toteutit järjestelmään olutkerhot, tuhoa käyttäjän tuhoamisen yhteydessä myös käyttäjän jäsenyydet olutkerhoissa



## More adjustments

Among the user editing actions you can also change the <code>username</code>. This does not make much sense, so remove the option.

Creating a new user and editing it make use of the same form, which is defined in the file views/users/_form.html.erb. The view templates which start with an underscore are Rails so-called [partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials), which are put together other templates in case of a <code>render</code> call.

The view template for editing users is below:

```erb
<h1>Editing user</h1>

<%= render 'form' %>

<%= link_to 'Show', @user %> |
<%= link_to 'Back', users_path %>
```

first it renders the elements in the _form template, and then a couple of links. The form code is below:

```erb
<%= form_for(@user) do |f| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>

      <ul>
      <% @user.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :username %><br>
    <%= f.text_field :username %>
  </div>
  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password %>
  </div>
  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation  %>
  </div>

  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>

```

This means you also want to delete the following chunk of code from the form

```erb
  <div class="field">
    <%= f.label :username %><br>
    <%= f.text_field :username %>
  </div>
```

_if_ you are editing the user information – that is, if the user object was already created.


With the method <code>new_record</code>, you can request from the object <code>@user</code> whether the form hasn't been stored in the database. In this way, you will show the <code>username</code> field in the form only when it's being created a new user:

```erb
  <% if @user.new_record? %>
    <div class="field">
      <%= f.label :username %><br />
      <%= f.text_field :username %>
    </div>
  <% end %>
```

Your form will do now, but it is still possible to change the user name by sending an HTTP POST request with a new username straight to the server.

Implement another verification in the <code>update</code> method of the User controller, to prevent changing the user name:

```ruby
  def update
    respond_to do |format|
      if user_params[:username].nil? and @user == current_user and @user.update(user_params)
        format.html { redirect_to @user, notice: 'User was successfully updated.' }
        format.json { head :no_content }
      else
        format.html { render action: 'edit' }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end
```

The form to change the user information will look like below, after all the changes you've implemented:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-7.png)

> ## Exercise 14
>
> The only information of users are their password now. Change the form to modify the user information so that it will look like the picture below. Notice, that the new user signup has to look like before.

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w3-8.png)

## Problems with heroku

If the program updated version is deployed in heroku, you will run into problems again. The page with all ratings, the one with all users, and the signup link cause a well-known error:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w2-12.png)

As we saw [last week](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#ongelmia-herokussa) you will have to find the reason from heroku logs.

The page with all users causes the following error:

    ActionView::Template::Error (PG::UndefinedTable: ERROR:  relation "users" does not exist

so the *users* database table does not exist because the application recent migrations haven't been executed in heroku. The issue will be solved by executing the migrations:

    heroku run rake db:migrate

The signup page will also work after executing the migrations.

The issue with the rating page will not be solved with the help of migrations, and you will have to look into the logs to find a solution:

```ruby
2015-01-24T19:19:58.672580+00:00 app[web.1]: ActionView::Template::Error (undefined method `username' for nil:NilClass):
2015-01-24T19:19:58.672582+00:00 app[web.1]:     2:
2015-01-24T19:19:58.672583+00:00 app[web.1]:     3: <ul>
2015-01-24T19:19:58.672584+00:00 app[web.1]:     4:   <% @ratings.each do |rating| %>
2015-01-24T19:19:58.672586+00:00 app[web.1]:     5:     <li> <%= rating %> <%= link_to rating.user.username, rating.user %> </li>
2015-01-24T19:19:58.672588+00:00 app[web.1]:     6:   <% end %>
2015-01-24T19:19:58.672589+00:00 app[web.1]:     7: </ul>
2015-01-24T19:19:58.672590+00:00 app[web.1]:     8:
```

The reason is the old one – the view code tries to call the <code>username</code> method of a nil object. It must be beacuse of the parameter in the <code>link_to</code> method

```ruby
    rating.user.username
```

the system contains ratings which don't belong to any user object.

Even though the database migration has been executed, a part of the data are still conform to the old database scheme. When it came to the database migration, it would have been wise to write a code to check that also the system data are brought to the form expected by the code, after the migration, meaning that each existing rating should belong to a user or otherwise the rating should have been removed.

Create a user in the database and use heroku console make so the first user created becomes the user of all existing ratings.

```ruby
irb(main):002:0> u = User.first
=> #<User id: 1, username: "mluukkai", created_at: "2015-01-24 19:56:38", updated_at: "2015-01-24 19:56:38", password_digest: "$2a$10$g3AEFZtiOa186yfBql3tOO9ELAIgBUwOFnnWIVwwfYS...">
irb(main):003:0> Rating.all.each{ |r| u.ratings << r }
=> [#<Rating id: 1, score: 21, beer_id: 1, created_at: "2015-01-17 17:55:43", updated_at: "2015-01-24 19:56:51", user_id: 1>, #<Rating id: 2, score: 15, beer_id: 2, created_at: "2015-01-18 20:12:59", updated_at: "2015-01-24 19:56:51", user_id: 1>]
irb(main):004:0>
```

Now your application will work.

Let me repeat the conclusion of "Problems in heroku" from last week, to end this week too.

<quote>
Most commonly, the problems we have in production depend of the inconsistent state that some beers have got because of our changes in the database scheme. For instance, they may be belonging to objects which do not exist or the references might be missing. **It is a good practice to deploy the application in the production mode as often as possible**, in this way, you will know that the potential problems are caused by the changes you have done and fixing them will be easier.
</quote>

## Returning the exercises


Commit all your changes and push the code to Github. Deploy to the newest version of Heroku, too.

You should mark that you have returned the exercises at http://wadrorstats2015.herokuapp.com
