
## Some things to keep in mind

### Rubocop

Remember to use Rubocop to check that all your future code still follows the configured style rules.

If you are using Visual Studio Code, consider installing the [ruby-rubocop](https://marketplace.visualstudio.com/items?itemName=misogi.ruby-rubocop) plugin.

### Issues with forms

In week 2 we modified the beer creation form so that the style and the brewery of the new beer are chosen from a dropdown menu. We modified the form to use _select_ instead of a text field:

```ruby
<div>
  <%= form.label :style, style: "display: block" %>
  <%= form.select :style, options_for_select(@styles) %>
</div>

<div>
  <%= form.label :brewery_id, style: "display: block" %>
  <%= form.select :brewery_id, options_from_collection_for_select(@breweries, :id, :name) %>
</div>
```

the selection options of dropdown menus are sent to the form through the variables <code>@styles</code> and <code>@breweries</code>, and their values are set by the controller method <code>new</code>:

```ruby
def new
  @beer = Beer.new
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

Interestingly after these changes, editing a beer information will stop working. It causes the error message <code>undefined method `map' for nil:NilClass</code>, which you have probably already encountered in the course:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w4-0.png)

The reason for this is that creating a new beer and editing a beer require the use of the same view template to generate the form (app/views/beers/_form.html.erb). Also after the changes, the view requires that the variable <code>@breweries</code> contains a list of the breweries and that the variable <code>styles</code> contains the beers styles. You access the beer editing page by executing the <code>edit</code> controller method, and you will have to fix the controller like shown below, if you want to fix the issue:

```ruby
def edit
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

You will find exactly the same problem if you try to create a beer which is not valid. In such case, the controller method <code>create</code> is the one which tries to render the view template which generates the form again. In fact, you will have to set a value to the variables <code>@style</code> and <code>@breweries</code> which need a template before rendering the page.

```ruby
def create
  @beer = Beer.new(beer_params)

  respond_to do |format|
    if @beer.save
      format.html { redirect_to beers_path, notice: 'Beer was successfully created.' }
      format.json { render action: 'show', status: :created, location: @beer }
    else
      @breweries = Brewery.all
      @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter"]

      format.html { render action: 'new' }
      format.json { render json: @beer.errors, status: :unprocessable_entity }
    end
  end
end
```

It's typical that the controller methods <code>new</code>, <code>create</code> and <code>edit</code> contain much of the same code, which is used to initiate the variables which view templates need. The best thing you can do is extracting the similar code into a method:

```ruby
def set_breweries_and_styles_for_template
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

The method can be called from the controller methods <code>new</code>, <code>create</code> and <code>edit</code>:

```ruby
def new
  @beer = Beer.new
  set_breweries_and_styles_for_template
end
```

 or you could do it with the <code>before_action</code> expression, which looks even better:

```ruby
class BeersController < ApplicationController
  # ...
  before_action :set_breweries_and_styles_for_template, only: [:new, :edit, :create]

  # ...
```

in this way, the method to set the values of the variables <code>@styles</code> and <code>@breweries</code> is executed authomatically always before executing the methods <code>new</code>, <code>create</code> and <code>edit</code>. We might not need to set the variables values in the method <code>create</code> because they are needed only in case the validation fails. It might have made sense to use an explicit call in create.

### Problems with Heroku or Fly.io

Many students in the course have met problems with Heroku where sometimes an application which was working perfectly locally has caused the same old, annoying error message in Heroku, _We're sorry, but something went wrong_.

The thing to do right in the beginning is making sure all the code has been added from the local computer to the version management, that is <code>git status</code>!

Non-trivial problems are always solved using Heroku's logs. You can inspect logs through the command <code>heroku logs</code> for Heroku and <code>fly logs</code> for Fly.io, from the command line

You find the log of a typical problem (in Heroku) below:


```ruby
mbp-18:ratebeer-public mluukkai$ heroku logs
2022-08-28T18:53:05.867973+00:00 app[web.1]:                   ON a.attrelid = d.adrelid AND a.attnum = d.adnum
2022-08-28T18:53:05.867973+00:00 app[web.1]:
2022-08-28T18:53:05.867973+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.867973+00:00 app[web.1]:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.874380+00:00 app[web.1]: Completed 500 Internal Server Error in 10ms
2022-08-28T18:53:05.878587+00:00 app[web.1]: :               SELECT a.attname, format_type(a.atttypid, a.atttypmod),
2022-08-28T18:53:05.878587+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.878587+00:00 app[web.1]:
2022-08-28T18:53:05.868310+00:00 app[web.1]:
2022-08-28T18:53:05.867973+00:00 app[web.1]:                      pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
2022-08-28T18:53:05.867973+00:00 app[web.1]:                  AND a.attnum > 0 AND NOT a.attisdropped
2022-08-28T18:53:05.868310+00:00 app[web.1]:                ORDER BY a.attnum
2022-08-28T18:53:05.878587+00:00 app[web.1]:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.867973+00:00 app[web.1]:                 FROM pg_attribute a LEFT JOIN pg_attrdef d
2022-08-28T18:53:05.882824+00:00 app[web.1]: LINE 5:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.882824+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.878587+00:00 app[web.1]:                      pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
2022-08-28T18:53:05.878587+00:00 app[web.1]:                   ON a.attrelid = d.adrelid AND a.attnum = d.adnum
2022-08-28T18:53:05.874380+00:00 app[web.1]: Completed 500 Internal Server Error in 10ms
2022-08-28T18:53:05.878587+00:00 app[web.1]: ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "users" does not exist
```

if you read the logs carefully, you will find the reason is the following

```ruby
ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "users" does not exist
```

you haven't executed the migrations. Fixing this will be easy:

    heroku run rails db:migrate

Fly.io executes migrations automatically, so most likely you won't run into the above problem there.

You find the log of another typical error (also in Fly.io) situation below:

```ruby
2022-08-28T19:32:31.609344+00:00 app[web.1]:     6:   <% @ratings.each do |rating| %>
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]:   app/views/ratings/index.html.erb:6:in `_app_views_ratings_index_html_erb___254869282653960432_70194062879340'
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]: ActionView::Template::Error (undefined method `username' for nil:NilClass):
2022-08-28T19:32:31.609344+00:00 app[web.1]:   app/views/ratings/index.html.erb:7:in `block in _app_views_ratings_index_html_erb___254869282653960432_70194062879340'
2022-08-28T19:32:31.609530+00:00 app[web.1]:     7:       <li> <%= rating %> <%= link_to rating.user.username, rating.user %> </li>
2022-08-28T19:32:31.609530+00:00 app[web.1]:     4:
2022-08-28T19:32:31.609530+00:00 app[web.1]:     6:   <% @ratings.each do |rating| %>
2022-08-28T19:32:31.609530+00:00 app[web.1]:     5: <ul>
2022-08-28T19:32:31.609715+00:00 app[web.1]:    10:
```

The error was _ActionView::Template::Error (undefined method \`username' for nil:NilClass)_  and it was happened while executing line 7 in file _app/views/ratings/index.html.erb_ .

The line which caused the problem:

```ruby
<li> <%= rating %> <%= link_to rating.user.username, rating.user %> </li>
```

It seems that in the database there is a `rating` object whose associated user is `nil`. We already met this issue in [week 2](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#problems-with-heroku).

The reason behind this is either a <code>nil</code> value for the <code>user_id</code> field of a rating, or an erroneous ID. One of the ways to solve the issue is destroying the 'bad' rating objects from the console. Heroku console opens with <code>heroku run console</code> and Fly.io with first running <code>fly ssh console</code> and then <code>/app/bin/rails c</code>.


```ruby
> bad_ratings = Rating.all.select{ |r| r.user.nil? or r.beer.nil? }
=> [#<Rating id: 1, score: 10, beer_id: 2, created_at: "2022-08-28 19:04:43", updated_at: "2022-08-28 19:04:43", user_id: nil>]
> bad_ratings.each{ |bad| bad.destroy }
=> [#<Rating id: 1, score: 10, beer_id: 2, created_at: "2022-08-28 19:04:43", updated_at: "2022-08-28 19:04:43", user_id: nil>]
> Rating.all.select{ |r| r.user.nil? or r.beer.nil? }
=> []
>
```

The commands above retrieve also the ratings that don't belong to any beer.

So if you have troubles with Heroku or Fly.io, find out where is the problem: logs and console will always help you out!


### Cancelling a migration

Sometimes you may want to cancel a migration you just executed – for instance if you create a bad scaffold, see the following section. This is possible through the command

    rails db:rollback

### Bad scaffold

If you want to remove the files created by the scaffold generator, you can do it with the command

    rails destroy scaffold resource_name

where _resource_name_ is the name of the resource you created with scaffold. **ATTENTION:** if you executed a bad scaffold migration already, you should definitely do <code>rails db:rollback</code> before using scaffold destroy.


## Testing

So far, we have only tested our code in the browser. This is a great mistake. Every program which is supposed to last for long should include an ample range of automatic tests, otherwise expanding the program will be too risky.

You can use Rspec for tests, see http://rspec.info/,  https://github.com/rspec/rspec-rails and http://betterspecs.org/

Get started with rspec-rails gem by adding the following to your Gemfile:

```ruby
group :test do
  # ...
  gem 'rspec-rails', '~> 6.0.0.rc1'
end
```

> At the moment of writing, the only available version of rspec-rails 6 ends in .rc1. The repository of the rspec project recommends using version 6.0.0 which might start working agin during the course.

You can set up the new gem in the familiar way, executing <code>bundle install</code> from the command line.

You can initialize rspec in your application running the following from command line

    rails generate rspec:install

The initialization creates a folder /spec in the application and the various tests – or specs – will be placed in its subfolders.

According to Rails standard but currently less common testing framework, the test are place in the folder /test. The folder will be useless after taking rspec and you can delete it.

The tests – the correct words would be specs or specifications when it comes to rspec, we will be using the word test in the future however – can be written at different levels: unit tests for models and controllers, view tests, and integration tests for controllers. In addition to these, the application can be tested using a simulated browser with the help of the capybara gem https://github.com/jnicklas/capybara.

We will be writing mostly unit tests for models as well as simulated browser-level tests with capybara.

## Unit tests

Let's try out a couple of unit tests for the class <code>User</code>. You can create a test by hand or from the command line with the rspec generator

    rails generate rspec:model user

The file user_spec.rb will appear in the folder /spec/models

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

Try to run the tests from the command line using the command <code>rspec spec</code> (attention: you might have to restart the terminal at this point!).

The test execution runs like this:

```ruby
$ rspec spec
*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) User add some examples to (or delete) /Users/mluukkai/opetus/ratebeer/spec/models/user_spec.rb
     # Not yet implemented
     # ./spec/models/user_spec.rb:4


Finished in 0.00932 seconds (files took 3.31 seconds to load)
1 example, 0 failures, 1 pending
```


The command  <code>rspec spec</code> defines that you execute all the tests which are located inside spec subfolders. If you have a lot of tests, you can also run define the smaller group of tests you want to run:

    rspec spec/models                # executing the tests contained in the models folder
    rspec spec/models/user_spec.rb   # executing the tests defined by user_spect.rb


You can also automate the test execution to run any time a test or the code concerning it changes.
The library used to do this is [guard](https://github.com/guard/guard) and there are numerous extensions available for it.

Let's start writing tests. First we'll create a test that tests that the constructor sets the username correctly (this in file _user_spec.rb_):

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it "has the username set correctly" do
    user = User.new username: "Pekka"

    expect(user.username).to eq("Pekka")
  end
end
```

The test is written in the code chunk which is given to the method called <code>it</code>. The first parameter of the method is a string, which will act as the test's name. otherwise the test is written in similarly to e.g. jUnit, meaning that the data to test is created first, later the action to test is executed, and at the end the result will be evaluated.

Execute the test and we see that it goes through:

```ruby
$ rspec spec

Finished in 0.00553 seconds (files took 2.11 seconds to load)
1 example, 0 failures
```

Differently from the jUnit testing framework, Rspecs don't make use of assert commands to evaluate the result. They make use of a more particular syntax, as the last test line shows:

    expect(user.username).to eq("Pekka")

In the test you have just run, you used the command <code>new</code> so the object was not save in the database. Try to store the object now. You defined that User objects have a password whose length is over 4 and that contains at least one digit and one uppercase letter. So if the password is not set up, the object should not be stored in the database. 

Test this:


```ruby
RSpec.describe User, type: :model do

  # previously written test code...

  it "is not saved without a password" do
    user = User.create username: "Pekka"

    expect(user.valid?).to be(false)
    expect(User.count).to eq(0)
  end
end
```

The test goes smoothly.

The first validation in the test

```ruby
expect(user.valid?).to be(false)
```

is understandably. Thanks to rspec magic power, it can also be expressed like this.

```ruby
expect(user).not_to be_valid
```

This form is based on the fact the object <code>user</code> has the method <code>valid?</code> which returns a true-false value.

We notice that we are using two ways to check equality in tests: <code>be(false)</code> ja <code>eq(0)</code>. What is the difference between the two? The matcher <code>be</code> can help you to check if two objects are the same. When comparing true-false values, <code>be</code> is a useful matcher. It will not work with strings, for instance. Try to change the comparison of the first test:

```ruby
expect(user.username).to be("Pekka")
```

the test will not go through now:

```ruby
1) User has the username set correctly
    Failure/Error: expect(user.username).to be("Pekka")

      expected #<String:70322613325340> => "Pekka"
          got #<String:70322613325560> => "Pekka"

      Compared using equal?, which compares object identity,
      but expected and actual are not the same object. Use
      `expect(actual).to eq(expected)` if you don't care about
      object identity in this example.
```

When it is enough that the content of two objects is the same, you will have to use <code>eq</code>, which applies to most of the situations except when it comes to true-false values. Though, you could use <code>eq</code> even with true-false values, writing


```ruby
expect(user.valid?).to eq(false)
```

Make a test with a proper password

```ruby
it "is saved with a proper password" do
  user = User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1"

  expect(user.valid?).to be(true)
  expect(User.count).to eq(1)
end
```

The first test "expectation" makes sure the new object validation is successful, so the method <code>valid?</code> will return true. The second expectation will make sure that there is one object in the database.

You could have used another form which can be read even more easily for the user validation:

```ruby
expect(user).to be_valid
```

You have to consider that rspec **will always reset the database after running each test**, so if you do a new test where you need Pekka, you have to create him again:

```ruby
it "with a proper password and two ratings, has the correct average rating" do
  user = User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1"
  brewery = Brewery.new name: "test", year: 2000
  beer = Beer.new name: "testbeer", style: "teststyle", brewery: brewery
  rating = Rating.new score: 10, beer: beer
  rating2 = Rating.new score: 20, beer: beer

  user.ratings << rating
  user.ratings << rating2

  expect(user.ratings.count).to eq(2)
  expect(user.average_rating).to eq(15.0)
end
```

As you might have guessed, it is not smart to initiate a test object various times, and the part in common can be extracted. You can do this by adding a <code>describe</code> chunk for each test containing the same initialization and defining a <code>let</code> command at the beginning of the chunk. The command will be executed before each test and it will initialize again a user variable each time:

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it "has the username set correctly" do
    user = User.new username: "Pekka"

    expect(user.username).to eq("Pekka")
  end

  it "is not saved without a password" do
    user = User.create username: "Pekka"

    expect(user).not_to be_valid
    expect(User.count).to eq(0)
  end

  describe "with a proper password" do
    let(:user){ User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1" }
    let(:test_brewery) { Brewery.new name: "test", year: 2000 }
    let(:test_beer) { Beer.create name: "testbeer", style: "teststyle", brewery: test_brewery }

    it "is saved" do
      expect(user).to be_valid
      expect(User.count).to eq(1)
    end

    it "and with two ratings, has the correct average rating" do
      rating = Rating.new score: 10, beer: test_beer
      rating2 = Rating.new score: 20, beer: test_beer

      user.ratings << rating
      user.ratings << rating2

      expect(user.ratings.count).to eq(2)
      expect(user.average_rating).to eq(15.0)
    end
  end
end
```

Initializing variables happens with the sligthly peculiar looking <code>let</code> method. E.g.:

```ruby
let(:user){ User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1" }
```

makes it so that after the definition, the variable _user_ refers to the User object created in the <code>let</code> method's code block.

Even though the variable is initialized only in one part of the code, so far, the initialization will be executed again before each method. Attention: the method <code>let</code> executes the object initialization only when the objec is really needed, this will have unexpected results sometimes!

Especially in older Rspec tests, you will see that the initialization happens through the <code>before :each</code> chunk. In such cases, the variable shared by various tests have to be defined as instance variable, like <code>@user</code>.

The choice of test and describe chunk names was not subject to chance. By defining the result of a test in the "documentation" format (with the -fd parameter), you will be able to print the tests results on the screen in a nice form:

```ruby
$ rspec -fd spec

User
  has the username set correctly
  is not saved without a password
  with a proper password
    is saved
    and with two ratings, has the correct average rating

Finished in 0.12949 seconds (files took 1.95 seconds to load)
4 examples, 0 failures
```

You should aim at writing test names so that executing them will produce "specification" which can be read as easily as possible.

You can also add the line ```-fd``` to the ```.rspec``` file, which will always show the rspec  tests of your project in the documentation format.

> ## Exercise 1
>
> Add tests to the User class to check that no object will be stored in the database if users create a password (with the create method) which is too short or made of letters only. Also, the validation of the new object should not succeed.

Remember to name your tests in a way so that the "spec" produced will sound grammatically appropriate after you run Rspec in the document format.

> ## Exercise 2
>
> Create a test base  with Rspec generator (or by hand) for the class <code>Beer</code> and make tests to check that
> * a beer can be created, and the created beer is stored in the database if the name, brewery, and style of the beer have been set
> * a beer won't be created (that is, no valid object will be brought about by create) if it is not given a name
> * a beer won't be created, if its style hasn't been defined
>
> If the last test does not go through, extend your code so that it will pass the test. Hint: A brewery id needs to be set for a beer but what if no brewery exists?
>
> If you create the test file by hand, remember to place it in the folder spec/models

## Text environments or fixtures

What we did before, creating the object structures for the tests by hand, might not be the best thing to do in some cases. A better way is grouping the structures for the test environment – that is to say the data to initialise the tests – in their own place, a "test fixture". Instead of using Rails standard fixture mechanism to initialize the tests, try the gem called FactoryBot, see
https://github.com/thoughtbot/factory_bot and https://github.com/thoughtbot/factory_bot_rails

Add the following to your Gemfile

```ruby
group :test do
  # ...
  gem 'factory_bot_rails'
end
```

and update the gems with the command <code>bundle install</code>

Create the file spec/factories.rb for your fixtures and write the following:

```ruby
FactoryBot.define do
  factory :user do
    username { "Pekka" }
    password { "Foobar1" }
    password_confirmation { "Foobar1" }
  end
end
```

The file defines an "object factory" for creating objects of `User` class. There is no need to explicitly define the class of the created objects as FactoryBot deduces it directly from the name of the used fixture, `user`.

You can ask the defined factories to create objects in the following way:


```ruby
user = FactoryBot.create(:user)
```

Calling the factoryBot method _create_ will create an object in the testing environment database automatically.

Modify your tests now to use FactoryBot for creating user objects.

```ruby
describe "with a proper password" do
  let(:user) { FactoryBot.create(:user) } # tämä rivi muuttui
  let(:test_brewery) { Brewery.new name: "test", year: 2000 }
  let(:test_beer) { Beer.create name: "testbeer", style: "teststyle", brewery: test_brewery }

  it "is saved" do
    expect(user).to be_valid
    expect(User.count).to eq(1)
  end

  it "and with two ratings, has the correct average rating" do
    rating = Rating.new score: 10, beer: test_beer
    rating2 = Rating.new score: 20, beer: test_beer

    user.ratings << rating
    user.ratings << rating2

    expect(user.ratings.count).to eq(2)
    expect(user.average_rating).to eq(15.0)
  end
end
```

The change is yet quite minimal. Let's expand the fixtures so that we can use them to create also the rating objects used in the tests. Change file spec/factories.rb:

```ruby
FactoryBot.define do
  factory :user do
    username { "Pekka" }
    password { "Foobar1" }
    password_confirmation { "Foobar1" }
  end

  factory :brewery do
    name { "anonymous" }
    year { 1900 }
  end

  factory :beer do
    name { "anonymous" }
    style { "Lager" }
    brewery # the brewery associated with beer is created with brewery factory
  end

  factory :rating do
    score { 10 }
    beer # The beer associated with rating is created with beer factory
    user # The user associated with rating is created with user factory
  end
end
```

On top of the factory creating ratings, fixtures for creating breweries and beers are also now defined in the file.

The factory <code>FactoryBot.create(:brewery)</code> creates a brewery whose name is 'anonymous' and is founded in 1900.

The factory <code>FactoryBot.create(:beer)</code> creates a beer whose style is 'Lager' and name 'anonymous' and a brewery is created for it. Accordingly, the factory <code>FactoryBot.create(:rating)</code> creates a rating which is associated with the beer and user created by the factory. Additionally the value of the rating, field _score_, is set to 10.

The test can be edited to following:

```ruby
describe "with a proper password" do
  let(:user) { FactoryBot.create(:user) }

  it "is saved" do
    expect(user).to be_valid
    expect(User.count).to eq(1)
  end

  it "and with two ratings, has the correct average rating" do
    FactoryBot.create(:rating, score: 10, user: user)
    FactoryBot.create(:rating, score: 20, user: user)

    expect(user.ratings.count).to eq(2)
    expect(user.average_rating).to eq(15.0)
  end
end
```

The test creates two ratings, other's scrore is 10 and the other's 20, that are are associated with the user created with the help of a factory in the _let_ command.

```ruby
FactoryBot.create(:rating, score: 10, user: user)
FactoryBot.create(:rating, score: 20, user: user)
```


So you can ask the same factory to create various objects:

```ruby
FactoryBot.create(:brewery)
FactoryBot.create(:brewery)
FactoryBot.create(:brewery)
```

would create three _different_ brewery objects that would all have identical values.

You can edit the values of factory-created objects with parameters. E.g.:
```ruby
FactoryBot.create(:brewery)
FactoryBot.create(:brewery, name: 'crapbrew')
FactoryBot.create(:brewery, name: 'homebrew', year: 2011)
```

this would create three breweries of which one would get the default name _anonymous_ and foundation year _1900_. The second brewery would get the default foundation year but the name _crapbrew_. The third would get both its name and year from the given parameters. 

Also the user factory could be called twice:

```ruby
FactoryBot.create(:user)
FactoryBot.create(:user)
```

This would however raise an exception as <code>User</code> object validations expects that usernames are unique but the the factory by default always creates users with the username "Pekka".

The follwing would however be okay; creating two users with different usernames, the default _Pekka_ and additionally _Vilma_

```ruby
FactoryBot.create(:user)
FactoryBot.create(:user, username: 'Vilma')
```

More instructions for using FactoryBot at https://www.rubydoc.info/gems/factory_bot/file/GETTING_STARTED.md

## Users favourite beers, breweries, and styles


Create methods for user in a test driven style (or behaviour driven, as rspec creators would say). The methods will help you find out users' favourite beers, breweries, and styles based on users ratings.

Orthodox TDD requires that you do not code anything before it is required by a minimal test. Create a test first requiring that <code>User</code> objects have the method <code>favorite_beer</code>.


```ruby
  it "has method for determining the favorite_beer" do
    user = FactoryGirl.create(:user)
    expect(user).to respond_to(:favorite_beer)
  end
```

The test will fail, so create the method body in the class User:

```ruby
class User < ActiveRecord::Base
  # ...

  def favorite_beer
  end
end
```

The test succeeds. Next add a test to check that without ratings, users will not have a favourite beer, so the method should return nil:

```ruby
  it "without ratings does not have a favorite beer" do
    user = FactoryGirl.create(:user)
    expect(user.favorite_beer).to eq(nil)
  end
```

The test will pass because Ruby's methods return nil by default.

Refactor the test adding a personal <code>describe</code> chunk to the two tests you have just written.

```ruby
  describe "favorite beer" do
    let(:user){FactoryGirl.create(:user) }

    it "has method for determining one" do
      user.should respond_to :favorite_beer
    end

    it "without ratings does not have one" do
      expect(user.favorite_beer).to eq(nil)
    end
  end
```

Add a test then to check the method is able to return the rated beer, if there is only one rating. In addition to the rating object, for the test you will need the brewery object which has the rating. Extend the fixtures first and add the following things:

```ruby
  factory :brewery do
    name "anonymous"
    year 1900
  end

  factory :beer do
    name "anonymous"
    brewery
    style "Lager"
  end
```

The code <code>create(:brewery)</code> creates a brewery whose name is 'anonymous' and the founding year is 1900. Similarly, <code>create(:beer)</code> creates a beer whose style is 'Larger' and name is 'anonymous', then it creates a brewery which the beer belongs to. If there hadn't been brewery in the definition code, the beer brewery value would have been <code>nil</code> meaning that the beer would have belonged to no brewery. The <code>create(:rating)</code> which you defined earlier creates a rating object whose score is set to 10, but the rating will not be linked automatically to either a beer or a user.

Using factoryGirl, you can now create a beer in your test (which will belong to a brewery automatically) as well as a rating which belongs to the created beer and to the user:

```ruby
    it "is the only rated if only one rating" do
      beer = FactoryGirl.create(:beer)
      rating = FactoryGirl.create(:rating, beer:beer, user:user)

      # jatkuu...
    end
```

So first you have created a beer, and then a rating. The rating is given the beer object and the user object (which had both been created using FactoryGirl) as parameters of the <code>create</code> method.

The rating created will belong to a user and it will be the only rating of that user. The test will expect at the end that the rated beer is the user favourite beer:

```ruby
    it "is the only rated if only one rating" do
      beer = FactoryGirl.create(:beer)
      rating = FactoryGirl.create(:rating, beer:beer, user:user)

      expect(user.favorite_beer).to eq(beer)
    end
```

Your test will not succeed, because your method does not do anything so far, and its return value is always <code>nil</code>.

Use [in the spirit of TDD](http://codebetter.com/darrellnorton/2004/05/10/notes-from-test-driven-development-by-example-kent-beck/) a "fake solution", without trying to make the final working version yet:

```ruby
class User < ActiveRecord::Base
  # ...

  def favorite_beer
    return nil if ratings.empty?   # returns nil if there are no ratings
    ratings.first.beer             # returns the beer which belongs to the first rating
  end
end
```

Make another test which will force you to make a real implementation [(see triangulation)](http://codebetter.com/darrellnorton/2004/05/10/notes-from-test-driven-development-by-example-kent-beck/):

```ruby
    it "is the one with highest rating if several rated" do
      beer1 = FactoryGirl.create(:beer)
      beer2 = FactoryGirl.create(:beer)
      beer3 = FactoryGirl.create(:beer)
      rating1 = FactoryGirl.create(:rating, beer:beer1, user:user)
      rating2 = FactoryGirl.create(:rating, score:25,  beer:beer2, user:user)
      rating3 = FactoryGirl.create(:rating, score:9, beer:beer3, user:user)

      expect(user.favorite_beer).to eq(beer2)
    end
```

Create three beers first and then ratings which belong to the beers as well as to the user object. The first rating will receive the default score of 10 points. The second and third rating are given a score in parameter.

The test will not succeed naturally, because the implementation of the method <code>favorite_beer</code> was left incomplete earlier.

Modify the method implementation to look like below:

```ruby
  def favorite_beer
    return nil if ratings.empty?
    ratings.sort_by{ |r| r.score }.last.beer
  end
```

So first the ratings are sorted by score, the last rating – the one with the highest score – is taken, and its beer is returned.

Because sorting was directly based on the rating attribute <code>score</code>, the last line of the method could have been written in a more compact form

```ruby
    ratings.sort_by(&:score).last.beer
```

How does the method work, actually? Execute the operation from the console:

```ruby
irb(main):020:0> u = User.first
irb(main):021:0> u.ratings.sort_by(&:score).last.beer
  Rating Load (0.2ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
  Beer Load (0.1ms)  SELECT "beers".* FROM "beers" WHERE "beers"."id" = ? ORDER BY "beers"."id" ASC LIMIT 1  [["id", 1]]
```

It produces 2 SQL enquiries, where the first

```ruby
SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
```

retrieves all the user ratings from the database. The ratings are sorted in the central memory. If the amount of ratings belonging to the user is extremely vast, you had better optimize the operation so that it is executed at database level.

If you look at the documentation (http://guides.rubyonrails.org/active_record_querying.html#ordering and http://guides.rubyonrails.org/active_record_querying.html#limit-and-offset) you will reach the following conclusion:

```ruby
  def favorite_beer
    return nil if ratings.empty?
    ratings.order(score: :desc).limit(1).first.beer
  end
```

You can check the SQL enquiry which resulted from the operation by hand from the console (notice the method <code>to_sql</code>):

```ruby
irb(main):033:0> u.ratings.order(score: :desc).limit(1).to_sql
=> "SELECT  \"ratings\".* FROM \"ratings\"  WHERE \"ratings\".\"user_id\" = ?  ORDER BY \"ratings\".\"score\" DESC LIMIT 1"
```


In order to optimize the execution capability, you should be patient and avoid optimizing each operation in the developing phase, if not necessary. 

## Auxiliary methods for tests

You must have noticed that the code to build the beers needed in tests is annoying. You could configure beers with ratings in FactoryGirl. You want to create however an auxiliary method <code>create_beer_with_rating</code> in the test file:

```ruby
    def create_beer_with_rating(score, user)
      beer = FactoryGirl.create(:beer)
      FactoryGirl.create(:rating, score:score, beer:beer, user:user)
      beer
    end
```

Making use of the auxiliary method will help you to polish your test

```ruby
    it "is the one with highest rating if several rated" do
      create_beer_with_rating(10, user)
      best = create_beer_with_rating(25, user)
      create_beer_with_rating(7, user)

      expect(user.favorite_beer).to eq(best)
    end
```

Auxiliary method can (and should) be defined in rspec files. If the auxiliary method is needed in only one test file, it can be place at the end of the file, for instance.

Improve again what you did before by defining another method <code>create_beers_with_ratings</code>, which allows to create various rated beers. The method receives as parameter a variable-length list which behaves like a table (see http://www.ruby-doc.org/docs/ProgrammingRuby/html/tut_methods.html, section "Variable-Length Argument Lists"):

```ruby
def create_beers_with_ratings(*scores, user)
  scores.each do |score|
    create_beer_with_rating(score, user)
  end
end
```

If you call the method like

```ruby
    create_beers_with_ratings(10, 15, 9, user)
```

the parameter <code>scores</code> will have a collection as value, containing the numbers 10, 15, and 9. The method creates three beers (with the help of the method <code>create_beer_with_rating</code>). They are given a user as parameter, the user has a rating, and the ratings will be given a score based on the numbers of the <code>scores</code> parameter.

Again, below you find the whole code to test the favourite beer:

```ruby
require 'rails_helper'

describe User do

  # ..

  describe "favorite beer" do
    let(:user){FactoryGirl.create(:user) }

    it "has method for determining one" do
      user.should respond_to :favorite_beer
    end

    it "without ratings does not have one" do
      expect(user.favorite_beer).to eq(nil)
    end

    it "is the only rated if only one rating" do
      beer = create_beer_with_rating(10, user)

      expect(user.favorite_beer).to eq(beer)
    end

    it "is the one with highest rating if several rated" do
      create_beers_with_ratings(10, 20, 15, 7, 9, user)
      best = create_beer_with_rating(25, user)

      expect(user.favorite_beer).to eq(best)
    end
  end

end # describe User

def create_beers_with_ratings(*scores, user)
  scores.each do |score|
    create_beer_with_rating score, user
  end
end

def create_beer_with_rating(score,  user)
  beer = FactoryGirl.create(:beer)
  FactoryGirl.create(:rating, score:score,  beer:beer, user:user)
  beer
end
```

### FactoryGirl troubleshooting

It is good to point out that if you define the FactoryGirl gem not only in the test environment but also in the development environment, like

```ruby
group :development, :test do
    gem 'factory_girl_rails'
    # ...
end
```

if you create new resources with Rails generator, fo instance:

    rails g scaffold bar name:string

a default factory will also be create:

```ruby
mbp-18:ratebeer_temppi mluukkai$ rails g scaffold bar name:string
      ...
      invoke    rspec
      create      spec/models/bar_spec.rb
      invoke      factory_girl
      create        spec/factories/bars.rb
      ...
```

the default position and contents are the following:

```ruby
mbp-18:ratebeer_temppi mluukkai$ cat spec/factories/bars.rb
# Read about factories at https://github.com/thoughtbot/factory_girl

FactoryGirl.define do
  factory :bar do
    name "MyString"
  end
end
```

This may put you into strange situations (if you define a factory with the same name yourself, the default one will be used instead!), so you'd better define the gem only in the test environment following the instructions of the section https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#testiymp%C3%A4rist%C3%B6t-eli-fixturet ohjeen.

Normally, rspec resets the database after each test execution. This is because rspec executes each test in a transaction by default which is rollbacked or canceled after the test execution. Tests are not saved in the dataabase, then.

Objects can go to the database for good in tests, however.

Suppose that you are testing the class <code>Beer</code:

```ruby
  describe "when one beer exists" do
    beer = FactoryGirl.create(:beer)

    it "is valid" do
      expect(beer).to be_valid
    end

    it "has the default style" do
      expect(beer.style).to eq("Lager")
    end
  end
```

the <code>Beer</code> object created by the test would go to your test database for good, because the command <ode>FactoryGirl.create(:beer)</code>  is located outside the tests, and it is not executed it during cancel transactions

Therefore, you will not want to place object creation code outside the tests (except for the methods which are called by the tests). Objects should be created in the test contex, either inside the method <code>it</code>:

```ruby
  describe "when one beer exists" do
    it "is valid" do
      beer = FactoryGirl.create(:beer)
      expect(beer).to be_valid
    end

    it "has the default style" do
      beer = FactoryGirl.create(:beer)
      expect(beer.style).to eq("Lager")
    end
  end
```

inside the command <code>let</code> or <code>let!<code>;

```ruby
  describe "when one beer exists" do
    let(:beer){FactoryGirl.create(:beer)}

    it "is valid" do
      expect(beer).to be_valid
    end

    it "has the default style" do
      expect(beer.style).to eq("Lager")
    end
  end
```

or only in the <code>before</code> chunk which you familiarize yourself with later on.

You can delete the beers which eventually ended up in the test database by starting the console in the test environment with the command <code>rails c test</code>.

The unicity options which are defined in the validation can produce something unespected sometimes. The User username has been defined as unique, so the test

```ruby
describe "the application" do
  it "does something with two users" do
    user1 = FactoryGirl.create(:user)
    user2 = FactoryGirl.create(:user)

  # ...
  end
end
```

would cause the error message

```ruby
     Failure/Error: user2 = FactoryGirl.create(:user)
     ActiveRecord::RecordInvalid:
       Validation failed: Username has already been taken
```
because FactoryGirl tries to create two user objects now through the definition

```ruby
  factory :user do
    username "Pekka"
    password "Foobar1"
    password_confirmation "Foobar1"
  end
```

so that 'Pekka' will be the username of both. You could solve the problem by giving another name to either of the objects which are being created:

```ruby
describe "the application" do
  it "does something with two users" do
    user1 = FactoryGirl.create(:user)
    user2 = FactoryGirl.create(:user, username:"Arto")

  # ...
  end
end
```
Another option would be defining the usernames used by FactoryGirl with the help of sequences, see
https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md#sequences

Sometimes, the validation problem might be hiding deeper down.

Suppose the brewery names had been defined as unique:

```ruby
class Brewery < ActiveRecord::Base
  validates :name, uniqueness: true

  #...
end
```

if you created two beers in your test now

```ruby
describe "the application" do
  it "does something with two beers" do
    beer1 = FactoryGirl.create(:beer)
    beer2 = FactoryGirl.create(:beer)

  # ...
  end
end
```

it would cause an error message

```ruby
     Failure/Error: beer2 = FactoryGirl.create(:beer)
     ActiveRecord::RecordInvalid:
       Validation failed: Name has already been taken
```

The error message might be confusing, because <code>Name has already been taken</code> refers exactly to the _brewery_ name!

The reason is the following. The beer factory has been defined in this way:

```ruby
  factory :beer do
    name "anonymous"
    brewery
    style "Lager"
  end
```

so it creates a _new_ brewery object for each beer by default and the object is created through the brewery factory:

```ruby
  factory :brewery do
    name "anonymous"
    year 1900
  end
```

So _all_ breweries will be called 'anonymous' and if the brewery name has been defined as unique (which does not make sense, because there might be many breweries with the same name), when creating a new beer will cause an issue, because the brewery which is created together with the beer would break the unicity condition.

## Tests and debugger

Hopefully you've made a rutine to use byebug. Because tests are also normal Ruby code, debug can also be used in both the test code and the code to be tested. The database status of the testing environment can be surprising sometimes, as que have seen in the examples above. In case of problems you should definitely stop your test code with the debugger and check whether the state of the ojects to test corresponds to what you expected. oletettua.

> ## Exercise 3
>
> ### This an the following exercise might be challenging. It is not essential that you make these two exercises if you want to continue with the material of the rest of the week, so do not get stuck here. You can also do them after you are done with the others.
>
> Make the method <code>favorite_style</code> for the <code>User</code> object in a TDD style. The method should return the style whose beers have received the highest avarage rating from the user. Add the information about the user favourite style in his page.
>
> Do not do everything with one method (unless you solve the problem at database level with AcriveRecord, which is also possible!), instead, define the suitable auxiliary methods! If you notice that you method is more than 5-line long, you are doing either too much or too complex things, so refactor your code. Ruby's collections have various auxiliary methods which might be useful for the exercise, see http://ruby-doc.org/core-2.2.0/Enumerable.html

> ## Exercise 4
>
> Make now the method <code>favorite_brewery</code> for the <code>User</code> object in a TDD style. The method should return the brewery whose beers have received the highest avarage rating from the user. Add the information about the user favourite brewery in his page.
>
> Create auxiliary methods in the rspect file if you need, in order to keep your tests clean and neat. If the auxiliary methods are similar, you should not copy-paste them, but abstract them.

The functionality needed for the methods <code>favorite_brewery</code> and <code>favorite_style</code> is very similar and the methods will contain more or less copy-pasted code. The material will show an example of code polishing in week 5.

## Capybara

We will move to program-level testing now. You are going to write automatic tests which use the application through the browser as normal users do. The de-facto solution for browser level testing with applications on Rails is Capybara https://github.com/jnicklas/capybara. The tests themselves are written in Rspec still, Capybara provides you with the browser simulation for the Rspec tests.

Add the gems 'capybara' and 'launchy' to the Gemfile (in test scope), so your test scope should look like what is written below:

```ruby
group :test do
  gem 'factory_girl_rails'
  gem 'capybara'
  gem 'launchy'
end
```

In order to set up the gems, you will have to execute the command <code>bundle install</code>. The command execution will take quite long time on department computers, even 15 minutes.

**ATTENTION** if bundle install does not work in the department computers, first run the following:

    gem install nokogiri -- --with-xml2-include=/usr/include/libxml2/libxml/ --with-xml2-lib=/usr/lib  --with-xslt-include=/usr/include/libxslt --with-xslt-lib=/usr/lib


You will also have to add the following line at the top of the spec/rails_helper.rb file

    require 'capybara/rspec'

You are ready now for your first browser-level test.

It is common to place the browser-level tests in the folder _spec/features_. Unit tests are usually organised so that the tests for each class are put in their own file. It is not always so clear how user-level tests which are executed through the browser should be organised. One option is using a file for each controller, another option is deviding the tests in different files according to the different functionalities of the system.

Get started by defining the tests for your breweries funcionality, and create the file spec/features/breweries_page_spec.rb:

```ruby
require 'rails_helper'

describe "Breweries page" do
  it "should not have any before been created" do
    visit breweries_path
    expect(page).to have_content 'Listing breweries'
    expect(page).to have_content 'Number of breweries: 0'
  end
end
```

The test will start navigating to the brewery list using the method <code>visit</code>. As you will see, Rails path helpers are used by Rspec tests too. After doing this, it checks whether the rendered page contains the text 'Listing breweries' and whether it tells the brewery number is 0 – the text 'Number of breweries: 0'. Capybara sets the page whenever the test has the <code>page</code> variable.

While testing, there will be a (great) amount of situations where it is useful seeing the HTML source code of the page which corresponds to the <code>page</code> variable. This is possible adding the command <code>puts page.html</code> to the test.

Another option is providing the test with the command <code>save_and_open_page</code>, which saves and opens the page in a default browser. In linux, you will have to defined the browser using the environment variable <code>BROWSER</code>. For instance, you can define Chrome in the department computers with the command:

    export BROWSER='/usr/bin/chromium-browser'

The definition will be enforced only in the shell where you make it. If you want to make it stable, add it in the file ~/.bashrc

Add a test for a situation where there are three breweries in the database:

```ruby
  it "lists the existing breweries and their total number" do
    breweries = ["Koff", "Karjala", "Schlenkerla"]
    breweries.each do |brewery_name|
      FactoryGirl.create(:brewery, name:brewery_name)
    end

    visit breweries_path

    expect(page).to have_content "Number of breweries: #{breweries.count}"

    breweries.each do |brewery_name|
      expect(page).to have_content brewery_name
    end
  end
```

Also add a test to make sure that you can access a brewery page by clicking a link in the page with breweries. Make use of Capybara <code>click_link</code> method, which helps to click on a page links.

```ruby
  it "allows user to navigate to page of a Brewery" do
    breweries = ["Koff", "Karjala", "Schlenkerla"]
    year = 1896
    breweries.each do |brewery_name|
      FactoryGirl.create(:brewery, name: brewery_name, year: year += 1)
    end

    visit breweries_path

    click_link "Koff"

    expect(page).to have_content "Koff"
    expect(page).to have_content "established in 1897"
  end
```

The test wil go through if the page form is the same as in the texts. In case of problems you should add the command <code>save_and_open_page</code> in the page and check visually the contents of the page opened by the test.

In the two last tests, the beginning is the same – you create three breweries first and then navigate to the breweries page.

You find below the refactored result, where the tests containing the same initiations are moved to their own describe chunk, where the <code>before :each</code> chunk has been defined to initiate them.

```ruby
require 'rails_helper'

describe "Breweries page" do
  it "should not have any before been created" do
    visit breweries_path
    expect(page).to have_content 'Listing breweries'
    expect(page).to have_content 'Number of breweries: 0'

  end

  describe "when breweries exists" do
    before :each do
      @breweries = ["Koff", "Karjala", "Schlenkerla"]
      year = 1896
      @breweries.each do |brewery_name|
        FactoryGirl.create(:brewery, name: brewery_name, year: year += 1)
      end

      visit breweries_path
    end

    it "lists the breweries and their total number" do
      expect(page).to have_content "Number of breweries: #{@breweries.count}"
      @breweries.each do |brewery_name|
        expect(page).to have_content brewery_name
      end
    end

    it "allows user to navigate to page of a Brewery" do
      click_link "Koff"

      expect(page).to have_content "Koff"
      expect(page).to have_content "established in 1897"
    end

  end
end
```

Notice that the <code>before :each</code> inside the describe chunk is executed once before each test defined under describe and **each test starts in a situation where the database is empty**.

## Testing user functionality

Move to user functionality, create the file features/users_spec.rb for this. Get started with a test to check whether users can log in the system:

```ruby
require 'rails_helper'

describe "User" do
  before :each do
    FactoryGirl.create :user
  end

  describe "who has signed up" do
    it "can signin with right credentials" do
      visit signin_path
      fill_in('username', with:'Pekka')
      fill_in('password', with:'Foobar1')
      click_button('Log in')

      expect(page).to have_content 'Welcome back!'
      expect(page).to have_content 'Pekka'
    end
  end
end
```

The test demonstrate an interaction with the form, the command <code>fill_in</code> looks for a text field for the username and inputs the parameter value.  As you might have guessed, <code>click_button</code> searches for a button in the page and clicks on it.

Notice the <code>before :each</code> chunk in the test, which uses FactoryGirl to create a User object before each test. Signing up would not work without the object, because the database is reset before each test execution.

More examples about various topics such as how to look for page elements using forms can be found in Capybara documentation, in the section The DSL, see https://github.com/jnicklas/capybara#the-dsl.

Implement a couple of tests more for user. The input of a wrong password should redirect back to the sign-in page:

```ruby
  describe "who has signed up" do
    # ...

    it "is redirected back to signin form if wrong credentials given" do
      visit signin_path
      fill_in('username', with:'Pekka')
      fill_in('password', with:'wrong')
      click_button('Log in')

      expect(current_path).to eq(signin_path)
      expect(page).to have_content 'Username and/or password mismatch'
    end
  end
```

The tests uses the method <code>current_path</code> which returns the path where the test execution has led to when the method is called. The method helps making sure the user is redirected back to the sign-in page if signing in failed.

It is not always so clear to what extent you should test your application business logic through browser-level tests. At least the logic to find out the user object favourite beer, brewery, and beer style should be tested with unit tests.

User-level tests can be used for instance to make sure pages show the same situation that there is in the database. So for instance, in your brewery page test you generated three breweries and tested that they are all rendered in the brewery list.

It makes sense to test that you can add and remove things on the pages, too. The test below for instance will make sure when a new user registers, the number of users in the system increases by one:

```ruby
  it "when signed up with good credentials, is added to the system" do
    visit signup_path
    fill_in('user_username', with:'Brian')
    fill_in('user_password', with:'Secret55')
    fill_in('user_password_confirmation', with:'Secret55')

    expect{
      click_button('Create User')
    }.to change{User.count}.by(1)
  end
```

Notice that the form fields were defined in the <code>fill_in</code> methods slightly differently than in the sign-in form. The fields ID can and should always be checked by looking at the page source code choosing _view page source_.

The test expects that clicking the _Create user_ button will cause the number of saved users in the database to increase by one. The syntax is brilliant, but it will take some time before Rspec's strongly expressive language will start to feel familiar.

You will have to take into consideration a small detail, that is, the method <code>expect</code> can be given parameters in two ways.
If the method has to test a value, the value is given between brakets, like <code>expect(current_path).to eq(signin_path)</code>. Instead, if it tests the impact of an operation (like the one above, <code>click_button('Create User')</code>) on the value of an application object (<code>User.count</code>), the operation to execute is given to <code>expect</code> in a code chunk.

Read more about this in Rspec documentation https://www.relishapp.com/rspec/rspec-expectations/v/2-14/docs/built-in-matchers

So the last test checked whether the operation executed at browser level created an object in the database. Should you make a separate test to see whether a user name can sign in the system? Maybe. After all, the previous test did not questioned whether the user object was saved in the database correctly.

The scope for testing is so wide, however, that a complete analysis is impossible and tests should be written in first place for things which might break.

Create a new test for beer rating. Create the file spec/features/ratings_spec.rb right for the test.

```ruby
require 'rails_helper'

describe "Rating" do
  let!(:brewery) { FactoryGirl.create :brewery, name:"Koff" }
  let!(:beer1) { FactoryGirl.create :beer, name:"iso 3", brewery:brewery }
  let!(:beer2) { FactoryGirl.create :beer, name:"Karhu", brewery:brewery }
  let!(:user) { FactoryGirl.create :user }

  before :each do
    visit signin_path
    fill_in('username', with:'Pekka')
    fill_in('password', with:'Foobar1')
    click_button('Log in')
  end

  it "when given, is registered to the beer and user who is signed in" do
    visit new_rating_path
    select('iso 3', from:'rating[beer_id]')
    fill_in('rating[score]', with:'15')

    expect{
      click_button "Create Rating"
    }.to change{Rating.count}.from(0).to(1)

    expect(user.ratings.count).to eq(1)
    expect(beer1.ratings.count).to eq(1)
    expect(beer1.average_rating).to eq(15.0)
  end
end
```

The test builds its brewery, two beers and a user with the method <code>let!</code> instead of <code>let</code> which we used earlier. In fact, the version without exclamative mark does not execute the operation immediatly, but only once the code refers to the object explicitely. The object <code>beer1</code> is mentioned only at the end of the code, so if you had created it with the method <code>let</code>, you would have run into a problem creating the rating, because its beer would have not existed in the database yet, and the corresponding select element would not have been found.


The code contained in the <code>before</code> chunk of the test helps users to sign in the system. Most probably, the same code chunk will be useful in various different test files. You had better extract the test code needed in various different place and make a module, which can be included in all test files which need it. Create a module in the file /spec/support/helpers/own_test_helper.rb and put the sign-in code there:

```ruby
module OwnTestHelper

  def sign_in(credentials)
    visit signin_path
    fill_in('username', with:credentials[:username])
    fill_in('password', with:credentials[:password])
    click_button('Log in')
  end
end
```

So the method <code>sign_in</code> takes a hash parameter with the user name and password.

Start to use the module method in your tests:

```ruby
require 'rails_helper'

include OwnTestHelper

describe "Rating" do
  let!(:brewery) { FactoryGirl.create :brewery, name:"Koff" }
  let!(:beer1) { FactoryGirl.create :beer, name:"iso 3", brewery:brewery }
  let!(:beer2) { FactoryGirl.create :beer, name:"Karhu", brewery:brewery }
  let!(:user) { FactoryGirl.create :user }

  before :each do
    sign_in(username:"Pekka", password:"Foobar1")
  end
```

and

```ruby
require 'rails_helper'

include OwnTestHelper

describe "User" do
  before :each do
    FactoryGirl.create :user
  end

  describe "who has signed up" do
    it "can signin with right credentials" do
      sign_in(username:"Pekka", password:"Foobar1")

      expect(page).to have_content 'Welcome back!'
      expect(page).to have_content 'Pekka'
    end

    it "is redirected back to signin form if wrong credentials given" do
      sign_in(username:"Pekka", password:"wrong")

      expect(current_path).to eq(signin_path)
      expect(page).to have_content 'username and password do not match'
    end
  end

  it "when signed up with good credentials, is added to the system" do
    visit signup_path
    fill_in('user_username', with:'Brian')
    fill_in('user_password', with:'Secret55')
    fill_in('user_password_confirmation', with:'Secret55')

    expect{
      click_button('Create User')
    }.to change{User.count}.by(1)
  end
end
```

Letting an auxiliary method be in charge of sign-in implementation will also improve the test clarity, and if the sign-up page functionality changes later on, maintaining the tests will be easy, because changes will be required only in one place.

**Attention:** if you see the error message <code>uninitialized constant OwnTestHelper (NameError)</code> move the definition
<code>include OwnTestHelper</code> to the end of the document <code>rails_helper.rb</code>.

**Attention2:** if the error message remains still, you can copy the defined auxiliary method straight at the end of the file <code>rails_helper.rb</code>.

> ## Exercise 5
>
> Make a test to verify that you can add a beer to the system through the www-page, if the beer name receives a valid value (meaning that it is not empty). Also make a test to verify that the browser is redirected to the beer creation page and is shown the appropriate error message, if the beer name is not valid, and that in this case, nothing is stored in the database.
>
> **ATTENTION:** your code might contain a bug in these situations, when you try to create an beer with an invalid name. Check out the fuctionality through your browser. The cause is explained at the beginning of week, https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#muutama-huomio. Fix the bug in your code.
>
> Keep in mind you can use the command <code>save_and_open_page</code> if you run into problems!


> ## Exercise 6
>
> Make a test to verify that the ratings in the database are shown in the page _ratings_ together with their amount. If their amount is not shown, fix this.
>
> *Hint**: one way you can make your test is creating ratings in the database with FactoryGirl first. Then you can test the content of the page 'ratings'.
>
> Keep in mind you can use the command <code>save_and_open_page</code> if you run into problems!


> ## Exercise 7
>
> Make a test to verify the user ratings are shown in his page. So the user page will have to show all the user personal ratings but not the ratings made by other users.
>
> Remember that you will have to give <code>user_path(user)</code> to the <code>visit</code> method. This will help the method to define the path and to navigate to the <code>user</code> page.

> ## Exercise 8
>
>  Make a test to make sure when a user removes their own rating, the rating will be removed from the database.
>
> If there are various links with the same name, <code>click_lik</code> will not work. You will have to specify what link to choose, see http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders and [tämä](http://stackoverflow.com/questions/6733427/how-to-click-on-the-second-link-with-the-same-text-using-capybara-in-rails-3), for instance.


> ## Exercise 9
>
> If you did the exercises 3-4, extend the user page so that it will show the user favourite style as well as favourite brewery. Also, make capybara tests for this. Tests which require more complex computetions do not require tests, because the unit tests ensure the functionality well enough. 


## Latest RSPec syntax styles

As you found out while writing your first tests, Rspec has got various ways to express one same thing. Do now some unit tests for your Brewery model. Get started by generating a test layout with the command

    rails generate rspec:model brewery

Write first in the "old-fashoned", now also deprecated <code>should</code> syntax (see the test https://github.com/rspec/rspec-expectations/blob/master/Should.md), which makes sure the <code>create</code> sets up the brewery name and founding year correctly, and that the obect is stored in the database.

```ruby
require 'rails_helper'

describe Brewery do
  it "has the name and year set correctly and is saved to database" do
    brewery = Brewery.create name:"Schlenkerla", year:1674

    brewery.name.should == "Schlenkerla"
    brewery.year.should == 1674
    brewery.valid?.should == true
  end
end
```

The last condition – whether the brewery is valid and stored in the database – is expressed in a bad way. Because the brewery method <code>valid?</code> returns a true-false value, you can also express the same thing in the following way (see http://rubydoc.info/gems/rspec-expectations/RSpec/Matchers):

```ruby
  it "has the name and year set correctly and is saved to database" do
    brewery = Brewery.create name:"Schlenkerla", year:1674

    brewery.name.should == "Schlenkerla"
    brewery.year.should == 1674
    brewery.should be_valid
  end
```

When you use a <code>be_something</code> predicate matcher, rspec expects the object has a true-false method called <code>something?</code>, which is like a "magic trick" made thanks to the conventions.

The expression <code>brewery.should be_valid</code> is nearer to natural language, so it is definitely more favourable. If you use should negation – the method <code>should_not</code> – you will be able to create a test to check whether breweries can be stored without a name:

```ruby
  it "without a name is not valid" do
    brewery = Brewery.create  year:1674

    brewery.should_not be_valid
  end
```

Also the <code>brewery.should be_invalid</code> form would work exactly in the same way.

We used the <code>expect</code> syntax above instead of should (see http://rubydoc.info/gems/rspec-expectations/) which has taken over should (Rspec developers still use almost only should in their 2010 book http://pragprog.com/book/achbd/the-rspec-book!). Our test would look like the one below, with expect:

```ruby
  it "has the name and year set correctly and is saved to database" do
    brewery = Brewery.create name:"Schlenkerla", year:1674

    expect(brewery.name).to eq("Schlenkerla")
    expect(brewery.year).to eq(1674)
    expect(brewery).to be_valid
  end

  it "without a name is not valid" do
    brewery = Brewery.create  year:1674

    expect(brewery).not_to be_valid
  end
```

The line <code>expect(brewery.year).to eq(1674)</code> could also be written in the form <code>expect(brewery.year).to be(1674)</code>; differently, <code>expect(brewery.name).to be("Schlenkerla")</code> would not work, the error message tells you something about the problem:

```ruby
  1) Brewery has the name and year set correctly and is saved to database
     Failure/Error: expect(brewery.name).to be("Schlenkerla")

       expected #<String:44715020> => "Schlenkerla"
            got #<String:47598800> => "Schlenkerla"

       Compared using equal?, which compares object identity,
       but expected and actual are not the same object. Use
       `expect(actual).to eq(expected)` if you don't care about
       object identity in this example.
```

So <code>be</code> requires two same objects, it is not enough that their contents are the same. According to Ruby, there is only one integer which is 1674, so 'be' will work with years, but there can be an infinite amound of strings containing the word "Sclenkerla," so you will have to use the <code>eq</code> matcher to compare strings.

It is possible to polish tests even better by using the syntax brought by Rspec 2 (https://www.relishapp.com/rspec/rspec-core/v/2-11/docs). In all your tests conditions, the _subject of your tests_ was the same – the object stored in the variable <code>brewery</code>. The new <code>subject</code> syntax makes it possible to define the subject of your tests only once, and then you will not need to refer to it explicitly. The following test is refactored using the new syntax:

```ruby
  describe "when initialized with name Schlenkerla and year 1674" do
    subject{ Brewery.create name: "Schlenkerla", year: 1674 }

    it { should be_valid }
    its(:name) { should eq("Schlenkerla") }
    its(:year) { should eq(1674) }
  end
```

The text is more compact than before and it reads extremely fluently. What's more, even the test report generated in document format looks very natural:

```ruby
➜  ratebeer git:(master) ✗ rspec spec/models/brewery_spec.rb -fd

Brewery
  without a name is not valid
  when initialized with name Schlenkerla and year 1674
    should be valid
    name
      should eq "Schlenkerla"
    year
      should eq 1674

Finished in 0.03309 seconds
```

**Attention:** the <code>its</code> syntax is not used any more in rspec core starting from Rspec version 3, and it will need the following gem to be set up and work:

    gem 'rspec-its'

More about the subject syntax at
https://www.relishapp.com/rspec/rspec-core/v/2-11/docs/subject

Advises to write well with Rspec can be found at
http://betterspecs.org/

## Test coverage

The tests line coverage measures the percentage of the program code lines that is executed when tests are executed. It is easy to measure the test coverage on Rails with the gem _simplecov_, see https://github.com/colszowka/simplecov

Get started with the gem by adding the following line to the Gemfile test scope

    gem 'simplecov', require: false

**Attention** instead of a normal <code>bundle install</code> command I personally had to execute the command <code>bundle update</code> at this point, in order to set up all the gem versions that were mutually fit.

In order to start using simplecov, you should the following code **in the first two lines** of the file rails_helper.rb:

```ruby
require 'simplecov'
SimpleCov.start('rails')
```

The run the tests (see the note above if you have problems here)

```ruby
➜  ratebeer git:(master) ✗ rspec spec
.............................

Finished in 1.29 seconds (files took 3.63 seconds to load)
29 examples, 0 failures
Coverage report generated for RSpec to /Users/mluukkai/kurssirepot/ratebeer/coverage. 146 / 201 LOC (72.64%) covered.
```

The tests line coverage is 72.64 percent. You find a more detailed report opening the file coverage/index.html with your browser. As it is shown by the picture, there are still large parts of the program which are tested poorly, especially as far as the controllers are concerned:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w4-1.png)

If you take a closer look at the report, you will notice that some classes are not even mentioned in the report! For instance, the beer club controller or method are not mentioned there. If there are no tests for a class, its class code will be left completely untuched in Simplecov report!

If you add the following lines somewhere in the test

```ruby
BeerClub
BeerClubsController
```

the classes code will be loaded during the test and simplecov will keep them in its report. The figures change slightly:

```ruby
➜  ratebeer git:(master) ✗ rspec spec
.............................

Finished in 1.3 seconds (files took 3.9 seconds to load)
29 examples, 0 failures
Coverage report generated for RSpec to /Users/mluukkai/kurssirepot/ratebeer/coverage. 158 / 234 LOC (67.52%) covered.
```

Wide-ranging tests does not mean that you are testing smart things, of course. Because it is easy to measure, it is better than nothing and it shows the most evident issues, at least.

> ## Exercise 10
>
> Start to use Simplecov in your code. Make sure all the essential code – models, controllers, and methods – are taken into consideration in the report!


## Continuous integration

With the term [continuous integration](http://martinfowler.com/articles/continuousIntegration.html) we mean the custom where the program developers integrate their code changes into a common project as often as possible. The idea making sure the program developing version works all the time and eliminating in this way the difficult, separate integration stage. The continuous integration requires a wide-ranging group of automatic tests to work. It is common when it comes to continuous integration to use a centralized server, which pays attention to the repository where the developing version is located. When developers integrate their code in the developing version, the integration server sees the change, builds the code, and runs the tests. If the tests fail, the integration server reports this in a way or in another to whom it may concern.

Travis https://travis-ci.org/ is a continuous integration service which follows the SaaS (Software as a Service) principle and which has been quickly favoured for Open Source projects.

Github Rails projects can be set up easily to be scrutinized by Travis

> ## Exercise 11
>
> In the repository root, create the configuration file .travis.yml for Travis which the following contents (ATTENTION! Put your Ruby version next to ```rvm:``` ):
> 
>```ruby
>language: ruby
>
>rvm:
>  - 2.0.0
>
>script:
>  - bundle exec rake db:migrate --trace
>  - RAILS_ENV=test bundle exec rake db:migrate --trace
>  - bundle exec rake db:test:prepare
>  - bundle exec rspec -fd spec/
>```
>
> Then click on "sign in with github" from Travis page and write your username.
>
> Go to the top right corner where there is your name, and choose "accounts". Link the opened view and the continuous integration of your ratebeer reporitory.
>
> When you push your code to Github next time, Travis will execute a build script automatically, defining that the tests should be executed. You'll receive an e-mail if the build status changes.
>
> Add a link to TravisCI page of the application in your repository READDME.md file (**attention:** the file ending has to be md!):
>
>```ruby
>[![Build Status](https://travis-ci.org/mluukkai/ratebeer-public.png)](https://travis-ci.org/mluukkai/ratebeer-public)
>```
> Notice that the last part of the link is the same as the one of your project GitHub repository, meaning that the one above refers to the Github repository https://github.com/mluukkai/ratebeer-public
>
> All the people who have to see the application status will be able to see it in this way, and the probability tests will not be broken will increase!

## Continuous delivery

Continuous delivery is a practice one more step beyond the continuous integration (see http://en.wikipedia.org/wiki/Continuous_delivery). They both have in common a continuous deployment, that is the idea to deploy the code to an environment like the production environment or even straight to production in the best case, every time that the code is integrated. 

Especially for Web application, continuous deployment can be an operation which does not require too much effort.

> ## Exercise 12
>
> Implement a continuous deployment to Heroku in your application using Travis-CI. Configure the migrations too, so that they will be executed together with the deployment.
>
> You find helping guidelines at
http://about.travis-ci.org/docs/user/deployment/heroku/
and http://about.travis-ci.org/blog/2013-07-09-introducing-continuous-deployment-to-heroku/
>
> **ATTENTION** it is heartly suggested that you make the configuration using [travis command line tool](http://blog.travis-ci.com/2013-01-14-new-client/)! Notice that after setting it up, you will have to restart the console.
>
> **ATTENTION2:** There have been problems of compatibility between Travis and Heroku from time to time. Look into the error messages and find out where is the problem, try to deploy again after some time (for instance, in a couple of hours). Don't get stuck with this!

## Code quality metrics

In addition to the test coverage, you should also pay attention to the code quality. Codeclimate (https://codeclimate.com) is a SaaS which allows you to generate various quality metrics for your Rails code.

> ## Exercise 13
>
>Codeclimate is free for opensource projects. Register your project by pressing on the rather anonimous link "Add an OS repo" at https://codeclimate.com/pricing.
>
>Codeclimate will complain about the repetitions in your code. It refers however to the quite bad code created by Rails scaffold, so you can leave it where it is.
>
>Link the quality metric report to your repository README file, too:
>
>```ruby
>[![Code Climate](https://codeclimate.com/github/mluukkai/ratebeer-public.png)](https://codeclimate.com/github/mluukkai/ratebeer-public)
>```
>
> Now codeclimate will also put enough pressure on you as the application developer to keep high quality code all the time!
>
> Notice that the last part of the link is the same as the one for your project Github repository. So the one below links to the Github repository https://github.com/mluukkai/ratebeer-public

In addition to Codeclimate static analysis, you should follow a regular coding style when you develop a Rails application. You find a style guide developed by Rails community at https://github.com/bbatsov/rails-style-guide

The amount of services to help the application developer life increases from day to day. Instead of or in addition to Simplecov, you can delegate the test coverage report to Coveralls https://coveralls.io/ -nimiselle pilvipalvelulle.

## Actions for signed-in users

Leave tests for a moment and go back to a couple of the previous themes. In week 2, you defined your application with http basic authentication so that users could delete breweries only with the admin password. [In week 3](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko3.md#vain-omien-reittausten-poisto) you defined your application functionality so that deleting ratings was not possible for others than the user who created that rating. Instead, things like creating, removing, and editing beer groups and beers are possible even without signing up, so far.

Put http basic authentication aside, and change your application so that beers, breweries, and beer groups can be created, edited and deleted only by signed-in users.

Get started by removing the http basic authentication. So remove the following line from the brewery controller

    before_action :authenticate, only: [:destroy]

as well as the method <code>authenticate</code>. Anyone can again remove breweries, now.

Start to add a protection.

It is easy to remove beers, beer clubs, and breweries editing and creation links from the sight if users are not signed in the system.

For instance, you can remove the beers creation link for non-signed-in users from the end of the page of the view views/beers/index.html.erb:

```erb
<% if not current_user.nil? %>
  <%= link_to('New Beer', new_beer_path) %>
<% end %>
```

So the creation link is shown only if the <code>current_user</code> is <code>nil</code>. And you can make use of the more compact form of if:

```erb
<%= link_to('New Beer', new_beer_path) if not current_user.nil? %>
```

Now, the <code>link_to</code> method will be executed – that is, the link code will be rendered – only if the if condition is true. 'If not' conditions don't make a too good Ruby's code, a better option would be using <code>unless</code>:

```erb
<%= link_to('New Beer', new_beer_path) unless current_user.nil? %>
```

So the link is rendered __unless__ the <code>current_user</code> is <code>nil</code>.

Actually, <code>unless</code> would be useless now, because <code>nil</code> is interpreted as false in Ruby, so the neatest form for the command would be

```erb
<%= link_to('New Beer', new_beer_path) if current_user %>
```

You will remove the adding, removing, and editing links soon, before doing it however, have a look at the protection at controller level. In fact, even though you removed all links to restriced actions, nothing prevents users from making a straight HTTP request to the application, and in this way doing an action which should be restricted to signed-in users.

So you will still have to make sure at controller level that if users try for some reason to do a forbidden action straight with HTTP, the action will not be executed.

You decide to lead users to the signed-in page if they try to do restricted actions.

Define the following method for the class <code>ApplicationController</code>:

```ruby
  def ensure_that_signed_in
    redirect_to signin_path, notice:'you should be signed in' if current_user.nil?
  end
```

So if users call the medhod without being signed-in, they are redirected to the signed-in page. Because the method is located in the class <code>ApplicationController</code> and all the controllers inherit this class, the method will be available for all controllers.

Add the method as a "before" filter (see http://guides.rubyonrails.org/action_controller_overview.html#filters and https://github.com/mluukkai/WebPalvelinohjelmointi2015/wiki/viikko-2#yksinkertainen-suojaus) for beer, brewery, and beer club controllers for all the methods except for index and show:

```ruby
class BeersController < ApplicationController
  before_action :ensure_that_signed_in, except: [:index, :show]

  #...
end
```

For instance, when a beer is being created Rails executes the filter <code>ensure_that_signed_in</code> before the method <code>create</code>, so the filter redirects to the sign-in page users who did not sign-in. If the user had signed in the system, the filter would not have done anything, and the new beer would have been created normally.

Try out that the changes work with your browser. So non-signed-in users are redirected to the sign-in page when they do any action which is restriced with the "before" filter, but signed-in users can access the page smoothly.

> ## Exercise 14
>
> Using "before" filters, prevent non-signed-in users from doing any action connected to breweries and beer clubs except listing them and showing the information of singular resources (so the methods <code>show</code> and <code>index</code>)
>
> When you have made sure that the functionality works fine, you can remove from the view the superfluous creating, removing, and editing links for non-signed-in users.

> ## Exercise 15
>
> Some of the program tests which you have done before the extentions in exercise 14 are broken. Fix the tests

## Polishing the application face

If you want you can polish the application views. For instance, you can remove the resource removing and editing links from the listing page:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w4-2.png)

and add the removing link to the page of singular resources:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w4-3.png)

these changes will not be essential and will not be undertaken in the following weeks either.

## Tehtävien palautus

Commit all your changes and push the code to Github. Deploy to the newest version of Heroku, too.

You should mark that you have returned the exercises at http://wadrorstats2015.herokuapp.com/


