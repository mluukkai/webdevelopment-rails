You will continue to develop your application from the point you arrived at the end of week 5. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week from the submission system.

## About Tests

A part of this week exercises may break some of the tests from the previous weeks. You can mark that you have done the exercises even though tests are broken. It's up to you whether you want to fix them or not.


## A reminder on debugger

On week 2 we got to know  [debugger](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#debugger).
If you haven't yet gotten into the hait of using it, here's a quick recap on its use.

Using debugger is very simple. You just write  <code>binding.pry</code> (or for the sglihtly less well working Rails native debugger: <code>binding.break</code> ) to _any_ part of your code. An example:

```ruby
class PlacesController < ApplicationController
   # ...

  def search
    city = params[:city]
    binding.pry
    @places = BeermappingApi.places_in(city)
    if @places.empty?
      redirect_to places_path, notice: "No locations in #{city}"
    else
      @weather = ApixuApi.weather_in(city)
      session[:city] = city
      render :index
    end
  end
```

Here we survey the BeermappingApi using section of the code. When the application searches for a beer restaurant the debugger opens a console session at the point marked in the code:

```ruby
From: /myapp/app/controllers/places_controller.rb:13 PlacesController#search:

    10: def search
    11:   @city = params[:city].downcase
    12:   binding.pry
 => 13:   @places = BeermappingApi.places_in(@city)
    14:   @weather = Weather.current(@city)
    15:
    16:   if @places.empty?
    17:     redirect_to places_path, notice: "No locations in #{@city}"
    18:   else
    19:     session[:last_city] = @city
    20:     render :index, status: 418
    21:   end
    22: end

[1] pry(#<PlacesController>)> params
=> #<ActionController::Parameters {"authenticity_token"=>"n7tewb4WlQqBhhr0dc_hFWc5r2VCiBIroM4q0N1AkYn7pXcRdjA61k98XguiVPm3QRmNShjzoMZ-Hy7KbQ9WZg", "city"=>"helsinki", "commit"=>"Search", "controller"=>"places", "action"=>"search"} permitted: false>
[2] pry(#<PlacesController>)>
```

We could now for example check that the contenst of the <code>params</code> hash
 is as we exoect it to be.

Execute the next command and see whether the result is as expected. The next command can be executed with the <code>ne</code> command.

```ruby
From: /myapp/app/controllers/places_controller.rb:14 PlacesController#search:

    10: def search
    11:   @city = params[:city].downcase
    12:   binding.pry
    13:   @places = BeermappingApi.places_in(@city)
 => 14:   @weather = Weather.current(@city)
    15:
    16:   if @places.empty?
    17:     redirect_to places_path, notice: "No locations in #{@city}"
    18:   else
    19:     session[:last_city] = @city
    20:     render :index, status: 418
    21:   end
    22: end

[3] pry(#<PlacesController>)> @places.size
=> 12
[4] pry(#<PlacesController>)> @places.first.name
=> "Pullman Bar"
[5] pry(#<PlacesController>)> exit
```

The last command lets the program progress normally.

Again, the debugger can be started from _any_ part of the application code, also from tests and even views. Try launching debugger while rendering the beer creation form:

```erb
From: /myapp/app/views/beers/_form.html.erb:15 #<Class:0x00007ffb824e7ac0>#_app_views_beers__form_html_erb__2870933239970559054_132200:

    10:       </ul>
    11:     </div>
    12:   <% end %>
    13:
    14:   <% binding.pry %>
 => 15:
    16:   <div>
    17:     <%= form.label :name, style: "display: block" %>
    18:     <%= form.text_field :name %>
    19:   </div>
    20:

[1] pry(#<#<Class:0x00007ffb824e7750>>)> @styles.size
  Style Count (3.8ms)  SELECT COUNT(*) FROM "styles"
  ↳ (pry):7
=> 8
[2] pry(#<#<Class:0x00007ffb824e7750>>)> @styles.first
  Style Load (3.6ms)  SELECT "styles".* FROM "styles" ORDER BY "styles"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ↳ (pry):8
=> #<Style:0x00007ffb83062cd8
 id: 1,
 name: "European pale lager",
 description:
  "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.",
 created_at: Thu, 01 Sep 2022 11:49:42.556514000 UTC +00:00,
 updated_at: Thu, 01 Sep 2022 14:06:48.157892000 UTC +00:00>
[3] pry(#<#<Class:0x00007ffb824e7750>>)>
```

 <code><% binding.pry %></code> has been added to the beer creation form view template. Even the helper method <code>options_from_collection_for_select</code> can be called from the debugger 

```ruby
[3] pry(#<#<Class:0x00007ffb824e7750>>)> options_from_collection_for_select(@styles, :id, :name, selected: @beer.style_id)
  Style Load (3.9ms)  SELECT "styles".* FROM "styles"
  ↳ (pry):9
=> "<option value=\"1\">European pale lager</option>\n<option value=\"2\">Pale Ale</option>\n<option value=\"3\">Porter</option>\n<option value=\"4\">Weizen</option>\n<option value=\"5\">watery</option>\n<option value=\"6\">IPA</option>\n<option value=\"7\">lowalcohol</option>\n<option value=\"8\">Lowalcohol</option>"
```

Once more: **When you have problems, instead of guessing, use the debugger!**

Throughout this course, the importance of using the Rails console as a development tool has been emphasized. So **when you are doing something even slightly untrivial, first test it in the console.** In some cases it might be even better to do the testing in the console launched by the debugger as then you can work in exactly the context you are writing the code for. This way you can access eg. variables <code>params</code>, <code>sessions</code> and other execution context dependent data.


## Bootstrap

We haven't been paying too much attention to the look of your application so far. The modern trend is that the HTML code only defines the information on the pages, whereas their outlook is defined in the separate CSS files.

We use <em>classes</em> <em>IDs</em> to define elements in HTML, so that the styles defined in the appropriate files go to the right spots on the page.

Already few weeks ago, you defined that the application layout navigation bar is located in the div element, which is given the class "navibar":

```erb
<div class="navibar">
  <%= link_to 'breweries', breweries_path %>
  <%= link_to 'beers', beers_path %>
  <%= link_to 'styles', styles_path %>
  <%= link_to 'ratings', ratings_path %>
  <%= link_to 'users', users_path %>
  <%= link_to 'clubs', beer_clubs_path %>
  <%= link_to 'places', places_path %>
  |
  <% if not current_user.nil? %>
    <%= link_to current_user.username, current_user %>
    <%= link_to 'rate a beer', new_rating_path %>
    <%= link_to 'join a club', new_membership_path %>
    <%= link_to 'signout', signout_path, method: :delete %>
  <% else %>
    <%= link_to 'signin', signin_path %>
    <%= link_to 'signup', signup_path %>
  <% end %>
</div>
```

In week 2, we defined a style for the navigation bar by adding the following features to the file application.css, in app/assets/stylesheats/:

```css
.navibar {
  padding: 10px;
  background: #efefef;
}
```

A developer could design the whole page to look like they want using a CSS, if they have  the appropriate eye and skills for it.

It is not even necessary to reinvent the wheel when it comes to the page design. Bootstrap <a href="http://getbootstrap.com/">http://getbootstrap.com/</a> is a styling library which contains a huge amount of CSS files and javascript intended for designing web pages. Among other web page styling libraries, Bootstrap has for a while now found favour with web page designers. At its time, Bootstrap was the first widely popular style library. Later a great number of other libraries have emerged, a few to mention: [Material UI](https://mui.com/material-ui/customization/how-to-customize/) and the more recent [Tailwind CSS](https://tailwindcss.com/)

Get started with bootstrapping your application with the gem <https://github.com/twbs/bootstrap-rubygem>. Add the following to your Gemfile:

```ruby
gem 'bootstrap', '~> 5.2.0'
gem 'jquery-rails'
gem 'mini_racer'
```

Set up the gems by running <code>bundle install</code>. After install, you need to restart the application.

Following the gem's [installation guide] (https://github.com/twbs/bootstrap-rubygem#a-ruby-on-rails) add the following to the **START** of file _app/javascript/application.js_

```
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

Also change the file name ending of _app/assets/stylesheets/application.css_ to _scss_ and add a row to the end of that file

```
@import "bootstrap";
```

Now, when you open your application in a browser (having restarted the application) you can already see a slight change for example in the fonts.


## Navbar

With Bootstrap the user interface is built of components defined as CSS classes. An example of a Bootstrap component is [navbar](https://getbootstrap.com/docs/5.2/components/navbar/). With navbar you can style the navigation bar of your application.

Change the contents of file _app/views/layouts/application.html.erb_  into:

```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Ratebeer</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <div class="container-fluid">
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item">
              <%= link_to 'breweries', breweries_path, { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'beers', beers_path , { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'ratings', ratings_path , { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'users', users_path , { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'clubs', beer_clubs_path , { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'places', places_path, { class: "nav-link" }  %>
            </li>
            <li class="nav-item">
              <%= link_to 'styles', styles_path , { class: "nav-link" } %>
            </li>
            |
            <% if current_user %>
              <li class="nav-item">
                <%= link_to "#{current_user.username}", current_user , { class: "nav-link" } %>
              </li>
              <li class="nav-item">
                <%= link_to "Rate a beer", new_rating_path , { class: "nav-link" } %>
              </li>
              <li class="nav-item">
                <%= link_to "Join a club", new_membership_path , { class: "nav-link" } %>
              </li>
              <li class="nav-item">
                <%= link_to "Sign out", signout_path, class: "nav-link", data: {turbo_method: :delete} %>
              </li>
            <% else %>
              <li class="nav-item">
                <%= link_to "Sign up", signup_path , { class: "nav-link" } %>
              </li>
              <li class="nav-item">
                <%= link_to "Sign in", signin_path , { class: "nav-link" } %>
              </li>
            <% end %>        
          </ul>
        </div>
      </div>
    </nav>

    <%= yield %>
  </body>
</html>
```

Bootstrap documentation is not the most legible documentation out there but with some head scratching, you can form a navigation bar whose contents match the previous one.

Even though the new navigation bar code is longer and messier than the previous version, there is a notable advantage. When viewing the application from a "big" screen, the navbar is shown normally:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-0a.png)

But if we view the app from a smaller screen, say, mobile device, instead of the navbar an icon is shown and tapping this icon drops down the navigation bar:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-0b.png)

A navigation bar formed with Bootstrap is _responseve_, it adjusts itself according to screen size.

### Grid

Not only Bootstrap easily helps you to create a responsive navigation bar, Bootstrap's grid will allow you to divide the page into different sections, see  https://getbootstrap.com/docs/5.2/layout/grid/


In the bottom of file  _app/views/layout/application.html.erb_ edit the part that refers to the rendering of individual view templates:

```erb
<%= yield %>
```

Change it into:

```erb
<div class="container">
  <div class="row">
    <div class="col-sm-8">
      <%= yield %>
    </div>
    <div class="col-sm-4">
      <img
        src="http://www.cs.helsinki.fi/u/mluukkai/wadror/pint.jpg"
        width="200"
        style="padding-top:30px"
      >
    </div>
  </div>
</div>
```


In the Bootstrap container (the part containing the contents of actual pages) we add a row that is split into two columns: an 8 wide column, into which the contents of a page will be embeded into and a 4 wide column in which we will display a picture, no matter which page the user currently is on. 

The page background is good now, and you can use Bootstrap styles and components on your pages.

### Notification

Several application views contain the ĺine

```erb
<p id="notice"><%= notice %></p>
```
With which the user is shown different notoifications such as _Beer was successfully created._

Notifications can be styled with Bootstrap's [alert](https://getbootstrap.com/docs/5.2/components/alerts/) component:

```erb
<% if notice %>
  <div class="alert alert-primary" role="alert">
    <%= notice %>
  </div>
<% end %>
```

Instead of editing every page containing the notification code, it is better to move the notification logic to a file _app/views/layout/application.html.erb_

```erb
<div class="container">
  <% if notice %>
    <div class="alert alert-primary" role="alert">
      <%= notice %>
    </div>
  <% end %>

  <div class="row">
    ...
  </div>
</div>
```

and remove it from other view files like _app/views/beers/index.html.erb_

If you use Visual Studio Code you can use the _replace in files_ functions to remove now redundant `<p id="notice"><%= notice %></p>` commands.

### More components

Bootstrap offers many different components. For example you can create stylish tables by using bootstrap's component https://getbootstrap.com/docs/5.2/content/tables/. You can use Bootstrap's default layout by adding the class <code>table</code> to the table HTML code, as it follows below:

```erb
<table class="table">
  ...
</table>
```

You want to add the class <code>table-hover</code> too. Thanks to this, if you move the mouse pointer on a line, this will become bold. The table class definition will be

```erb
<table class="table table-hover">
  ...
</table>
```

> ## Exercise 1
>
> The page listing all the beers becomes quite cumbersome when the number of beers grows. Make the beer page use a bootstrap styled [table](https://www.w3schools.com/html/html_tables.asp), see https://getbootstrap.com/docs/5.2/content/tables/
>
> If you edit a beer row using _partials_ file, remember to take this into consideration in other files. In this exercise it is recommendable to stop using the <code>_beer.html.erb</code> partial in rendering the beer table. Instead form the entire table in file <code>views/beers/index.html.erb</code>

After the exercise your application can look something like this

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/beers-bootstrapped.png)



Bootstrap also provides you with button styles https://getbootstrap.com/docs/5.2/components/buttons/

Use the blue button which is defined by the class pair <code>btn btn-primary</code>. Below an example where the class has been added a button for beer rating:

```erb
<h4>give a rating:<h4>

<%= form_with(model: @rating) do |form| %>
  <%= form.hidden_field :beer_id %>
  score: <%= form.number_field :score %>
  <%= form.submit "Create rating", class:"btn btn-primary" %>
<% end %>
```

The class can also be added to links which you want to look like buttons:

```erb
<%= link_to('New Beer', new_beer_path, class:'btn btn-primary') if current_user %>
```

> ## Exercise 2

> Add styles to your application for at least a couple of button and links. You may want to choose the style <code>btn btn-danger</code> for the delete operation.

> ## Exercise 3
>
> The application forms are still pretty ugly. Make at least the form for creating new beer clubs more stylish with Bootsrap [form](https://getbootstrap.com/docs/5.2/forms/overview/) styling components
>
> Note: If you use form helper methods like `select`, you might have to give the class in `html_options` hash. See [documentation](https://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-select). For example, for select the class would be given like this: `<%= f.select :field, choices, {}, { :class => "class-here" } %>`
>
> You can decide the style yourself. One way to style the form is:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-3a.png)


> ## Excercise 4
> Change the navigation bar so that when users sign in, their signed-in user actions are contained in a drop down menu like in the picture below.
>
>You find guidelines from the [navbar documentation](https://getbootstrap.com/docs/5.2/components/navbar/) from examples containing _dropdown_ elements.

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-3c.png)

> If your dropdowns don't seem to work, make sure the requires and imports are in the right order in the file <code>application.js</code>, requires before imports.

> ## Exercise 5
>
> Make part of your Web site fashionable using some Bootstrap component. You can mark this exercise if you spend at least 15 minutes to improve the outlook of your pages. 

## Brewery activity

Some of the breweries have gone out of bussiness and you want to distinguish them from the active breweries in the list. Add a boolean column to the brewery database that tells whether they are active. Create a migration:

    rails g migration AddActivityToBrewery active:boolean


Attention: because the migration name starts with the word Add and ends in the object name, that is Brewery, and because it contains the information about the column to be added, the right migration code is generated automatically. It is however smart to always check the generated code.

```ruby
class AddActivityToBrewery < ActiveRecord::Migration[7.0]
  def change
    add_column :breweries, :active, :boolean
  end
end
```


Execute the migration; then go to your console to mark by hand all the breweries in the database as active:

```ruby
> Brewery.all.each{ |b| b.active=true; b.save }
```

Go and create a new brewery so that your database will contain an inactive brewery too.

Then change the brewery page so that next to the brewery name, it tells if the brewery is inactive:

```erb
 <h2>
  <%= brewery.name %>
  <% if not brewery.active  %>
    <span class="badge bg-secondary">retired</span>
  <% end %>
</h2>
```


It makes sense to allow setting up the brewery activity from the forms to create and edit them. Add an activity setting checkbox to views/breweries/_form-html.erb:

```erb
<div>
  <%= form.label :active, style: "display: block" %>
  <%= form.check_box :active %>
</div>
```

Try this out. You will see that changing the activity does not work, however.

The problem is that the attribute <code>active</code> is not among the attributes which are allowed for mass assignment.

Inspect the brewery controller a bit. Creating breweries and editing their information both retrieve the brewery information with the <code>brewery_params</code> method.

```ruby
def create
  @brewery = Brewery.new(brewery_params)

  # ...
end

def update
  # ...
  if @brewery.update(brewery_params)
  # ...
end

def brewery_params
  params.require(:brewery).permit(:name, :year)
end
```

As you've seen in [week 2](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#saving-a-rating) every attribute intended for mass assignment has to be explicitely allowed through the method <code>permit</code>. Change the method <code>brewery_params</code> as it follows:

```ruby
def brewery_params
  params.require(:brewery).permit(:name, :year, :active)
end
```

You want to show the active and inactive breweries separately in the breweries list. A straight-forward solution is saving the active and inactive ones with the controller using separate variables:

```ruby
def index
  @active_breweries = Brewery.where(active: true)
  @retired_breweries = Brewery.where(active: [nil, false])
end
```

The value <code>active</code> of the field can either be explicitally set as <code>false</code> or <code>nil</code>; we had to add both options to the last <code>where</code> sentence so that they both refer to inactive breweries.

Copy paste the table in the view twice, for the active and inactive ones:

```erb

<h1>Breweries</h1>

<h2>Active</h2>

<p> Number of active breweries: <%= @active_breweries.count %> </p>

<div id="breweries">
  <% @active_breweries.each do |brewery| %>
    <%= render brewery %>
  <% end %>
</div>

<h2>Retired</h2>

<p> Number of retired breweries: <%= @retired_breweries.count %> </p>

<div id="retired_breweries">
  <% @retired_breweries.each do |brewery| %>
    <%= render brewery %>
  <% end %>
</div>

<p>
<%= link_to "List of beers", beers_path%>
</p>
<%= link_to("New brewery", new_brewery_path, class:"btn btn-primary") if current_user %>
```

The solution works, but there are a couple of options which are even better. Start with the controller first.

The controllers requires a list of both active and inactive breweries. The controller also tells how the two lists are retrived from the database.

You could polish the controller by making so that the class <code>Brewery</code> provides a better interface to find the breweries list. ActiveRecord provides a nice solution for this, scope, see http://guides.rubyonrails.org/active_record_querying.html#scopes

Define two scopes for the breweries, active and inactive:

```ruby
class Brewery < ApplicationRecord
  has_many :beers, dependent: :destroy
  has_many :ratings, through: :beers

  validates :name, presence: true
  validates :year, numericality: { only_integer: true,
                                   greater_than: 1039,
                                   less_than_or_equal_to: ->(_) { Time.now.year } }

  scope :active, -> { where active: true }
  scope :retired, -> { where active: [nil,false] }

  include RatingAverage
end

```

The scope defines a class method which returns all the beers returned according to the search against the scope.

Now the class <code>Brewery</code> provides you not only with all the breweries but also a nice interface with the active the inactive ones:

    Brewery.all      # all breweries
    Brewery.active   # the active ones
    Brewery.retired  # the inactive ones


The controller will be more elegant at this point:

```ruby
def index
  @active_breweries = Brewery.active
  @retired_breweries = Brewery.retired
end
```


The solution is better not only for the clarity but also in terms of responsibility assignment of the objects. It is not too good to make the controller tell <em>how</em> active and retired breweries have to be retrived from the database. Instead, it is natural to make a model responsible for it, because models role is to act as an abstract level between the rest of the application and the database.

Note that ActiveRecord allows operation chaining. You could write:

```ruby
  Brewery.where(active: true).where("year > 2000")
```

and the result would be a SQL query:

```sql
SELECT "breweries".* FROM "breweries" WHERE "breweries"."active" = ? AND (year>2000)
```

ActiveRecord knows to optimize chained method call as one SQL operation. The scope also works as a part of a chain. Eg. You can find all still active breweries that were founded after 2000 with the following one-liner:

```ruby
Brewery.active.where("year > 2000")
```




> ## Exercises 6 – 7 (this equals two exercises)

> Your Ratings page is somehow boring now. Instead of the ratings, modify it to show:
> - The three best beers and breweries based on the average rating scores
> - The five last ratings which were made.
>
> **Hints:**
>
> If a beer/brewery has no ratings, counting the average with method <code>average_rating</code> will most likely cause on error (when ordering by rating). Fix the method so that it can count an average also for beers/breweries with no ratings.
>
>Implement a scope <code>:recent</code> to the class <code>Rating</code>, returning the last five ratings. You find more information on the database request required by scope at http://guides.rubyonrails.org/active_record_querying.html, see order and limit. Try to make the request from the console first!
>
>The scope for the best beer and brewery  won't be so simple to make, because they have to find the objects returned by the scope at database level, and that would require complex SQL.
>
>Instead of the scopes, you can make class-level methods (or static method, to tell it in Java's words) for the classes <code>Brewery</code>, <code>Beer</code>, and <code>User</code>, so that the controller will have access to them. For instance, the brewery method could be something like this:
>
> ```ruby
> class Brewery
>  # ...
>
>  def self.top(n)
>    sorted_by_rating_in_desc_order = Brewery.all.sort_by{ |b| ... }
>    # return n best from the list
>    # how? see http://www.ruby-doc.org/core-2.5.1/Array.html
>  end
> end
> ```
>
> The method is used from the controller as shown below:
>
> ```ruby
>  @top_breweries = Brewery.top 3
> ```
> Attention: beers, styles and breweries <code>top</code> methods are actually made of copy-paste, and using the modules would allow to define a code only in one place. Once you have done all the week exercises, you can try to clean your code!
>
>Do not copy-paste the views code, but use partials when needed instead.

> ## Exercise 8
>
> Now add the three best rated beer styles and the three most active raters (most ratings) to ratings page.

After the exercises, the ratings page could look like:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-4.png)

This might help with page styling: https://getbootstrap.com/docs/5.2/layout/grid/#nesting

## Cleaning the view code with helpers


In week 3 we added the <code>current_user</code> method to the class <code>ApplicationController</code> and we said it was a so called helper method

```ruby
class ApplicationController < ActionController::Base
  # ...
  helper_method :current_user

 end
```

Both controllers and views can use the method to check the identity of users who have signed in. Because the method is defined in the class <code>ApplicationController</code> it is available for all controllers. Being definied as helper method, it is available for views too.

Applications often need auxiliary methods (which are called helper methods in Rails) only for view templates. In such cases, they shouldn't be placed in the class <code>ApplicationController</code> but in the modules in <em>app/helpers/</em>. If an auxiliary method is supposed to be used in more than one view, the correct place for them is <code>application_helper</code>. Instead, if the auxiliary methods are for the pages which depend on only one controller, they should be defined into the helper module corresponding to the controller.

You'll notice that your views have some redundant parts of code. For instance, the show.html.erb templates for beer, style, and brewery all contain very similar code, which creates the links for editing and deleting:

```erb
<% if current_user %>
  <%= link_to 'Edit', edit_brewery_path(@brewery), class:"btn btn-primary"  %>
  <%= link_to 'Destroy', @brewery, method: :delete, data: { confirm: 'Are you sure?' }, class:"btn btn-danger"  %>
<% end %>
```

Separate them in their own helpers, into the module application_helper.rb

```ruby
module ApplicationHelper
  def edit_and_destroy_buttons(item)
    unless current_user.nil?
      edit = link_to('Edit', url_for([:edit, item]), class: "btn btn-primary")
      del = link_to('Destroy', item, method: :delete,
                                     form: { data: { turbo_confirm: "Are you sure ?" } },
                                     class: "btn btn-danger")
      raw("#{edit} #{del}")
    end
  end
end
```

The method creates two HTML link elements with _link_to_ and returns both the links "raw" (see http://apidock.com/rails/ActionView/Helpers/RawOutputHelper/raw), which basically means HTML code, which can embed in the page.

The buttons are added to the beer style pages as below:

```erb
<h2>
  <%= @style.name %>
</h2>

<quote>
  <%= @style.description %>
</quote>

...

<%= edit_and_destroy_buttons(@style) %>
```

The view template really looks much better now.

It would also be possible to separate the buttons code in their own partial, and it is a matter of taste which is the best solution here, whether a helper method or a partial.

> ## Tehtävä 9
>
> Most of the pages show the average rating value. Average values are Decimal types, so sometimes they are printed with too much precision even. Define an auxiliary method <code>round(param)</code> to render the average value of ratings. The medhod should always print its parameter with only <code>one</code> decimal digit precision. Make use of this helper method in the view templates (or at least in some of them).
>
> You can use the Rails method <code>number_with_precision</code> in your helper, see http://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html#method-i-number_with_precision

## Route for changing the brewery status

A moment ago, breweries were added the information about their activity and they were given the possibility to change their activity status from the brewery information editing form. It is quite unrealistic, but think that breweries could stop for some time and start again their activity. In such case, editing the activity status from the brewery information editing form would be a bit cumbersome. In such cases, it would be easier if the list with all breweries had a button to change their status with a click. You could implement this kind of button by embedding a suitable form for each breweries in the list. You want to choose another solution this time, though. In addition to Rails' six default routes, add the new route <code>toggle_activity</code> to the breweries, so that you will be able edit the brewery status with the help of the HTTP POST calls made to this route.

Make the following change to breweries in the file routes.rb as:

```ruby
resources :breweries do
  post 'toggle_activity', on: :member
end
```

If you now run <code>rails routes</code>, you'll see the new route which appeared for brewery:

```ruby
  toggle_activity_brewery POST   /breweries/:id/toggle_activity(.:format)                                                 breweries#toggle_activity
                breweries GET    /breweries(.:format)                                                                     breweries#index
                          POST   /breweries(.:format)                                                                     breweries#create
              new_brewery GET    /breweries/new(.:format)                                                                 breweries#new
             edit_brewery GET    /breweries/:id/edit(.:format)                                                            breweries#edit
                  brewery GET    /breweries/:id(.:format)                                                                 breweries#show
                          PATCH  /breweries/:id(.:format)                                                                 breweries#update
                          PUT    /breweries/:id(.:format)                                                                 breweries#update
                          DELETE /breweries/:id(.:format)                                                                 breweries#destroy
```

You want to add the activity status change functionality to each brewery page. So add the following to the brewery page app/views/breweries/show.html.erb:

```erb
<%= link_to "change activity", toggle_activity_brewery_path(@brewery.id), data: {turbo_method: "post"}, class: "btn btn-primary" %>
```


When you click on the button now, the browser will make an HTTP POST request for the address /breweries/:id/toggle_activity, where ID field is the ID of the brewery you clicked on. Rails routing mechanism tries to call a breweries controller <code>toggle_activity</code> method which does not exist, so this results in an error message. The method can be implement like this:

```ruby
def toggle_activity
  brewery = Brewery.find(params[:id])
  brewery.update_attribute :active, (not brewery.active)

  new_status = brewery.active? ? "active" : "retired"

  redirect_to brewery, notice:"brewery activity status changed to #{new_status}"
end
```

Implementing this functionality was easy, but does it make sence to add  the route <code>toggle_activity</code> in first place? According to the RESTful ideology, it would be more orthodox to use a form to do this, through a PUT request for the path breweries/:id. In any case, you should avoid situations, where a resource status is changed through GET requests. For this reason you defined the path toggle_activity for POST requests.

More about custom routes at http://guides.rubyonrails.org/routing.html#adding-more-restful-actions

>
## Admin user and access management

> ## Exercise 10
>
>Anyone who is signed-in can delete breweries, beers, and beer clubs, so far. Extend the system so that a part of the users are administrators, and delete operations are restricted to them alone.
>
> - Create a new boolean field <code>admin</code> for the User model. The field helps to indicate the users who have admin rights to the system.
> - It's enough that admins can be defined only from console.
> - Make breweries, beers, beer clubs, and styles deletion possible only for aministrators.
>
> **Attention:** because of password validation reasons, turning a user into admin will not be possible from console if the password field has no value:
>
> ```ruby
> > u = User.first
> > u.admin = true
> > u.save
>   (0.1ms)  rollback transaction
> => false
> ```
>
> Editing the value of singular attributes is still possible by bypassing the validation with the method <code>update_attr</code>:
>
> ```ruby
> > u.update_attribute(:admin, true)
> ```
>
>**ATTENTION:** you'd better use a [before filter](https://github.com/mluukkai/webdevelopment-rails/blob/main/week4.md#functions-for-signed-in-users) when you do this


> ## Exercises 11 – 12 (it's worth of two points)
>
> Implement such functionality to let administrators freeze user accounts. Freezing can happen with a button that only administrators see on a user's page. Frozen users can not sign in the system. When they try to sign in, the application should tell that their user name has been frozen, and they should get in touch with admins. Administrators should be able to reactivate frozen user accounts.
>
> You may implement this functionality following the pictures below

The administrator can freeze a user account from the user's page

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-1c.png)

The administrator can see the frozen user accounts from the users view

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-1b.png)

If an user's account is frozen, they won't be able to sign in

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-1x.png)

The administrator can reactivate frozen user names from the user's page

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-1d.png)

> ## Exercise 13
>
> Most likely some of your tests have broken due to this week's changes. Fix the tests.

## Advanced authorization

If your application needs a more diverse authorization, you may want to manage it with the help of the <em>cancan</em> gem, see <a href="https://github.com/CanCanCommunity/cancancan">https://github.com/CanCanCommunity/cancancan</a> and
<a href="http://railscasts.com/episodes/192-authorization-with-cancan">http://railscasts.com/episodes/192-authorization-with-cancan</a>

The Rails cast page is old, and you had better take a look at the Github page of the project. Rails casts provide you with brilliant insights on various topics, so even though they may not be updated with all details any more, you may want to go through them in any case.


<a id="user-content-rails-sovellusten-tietoturvasta" class="anchor" href="#rails-sovellusten-tietoturvasta" aria-hidden="true"><span class="octicon octicon-link"></span></a>Rails applications information security

We haven't said anything about Rails applications information security, so far. It's time to go into the topic now. Rails guides provide a brilliant overview on the most common data security threats for Web application and how you can prepare for them on Rails.

<blockquote>

<a id="user-content-tehtävät-12-14-kolmen-tehtävän-arvoinen" class="anchor" href="#teht%C3%A4v%C3%A4t-12-14-kolmen-teht%C3%A4v%C3%A4n-arvoinen" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercises 12 – 14 (it is worth three points)

Read <a href="http://guides.rubyonrails.org/security.html">http://guides.rubyonrails.org/security.html</a>

The text is long but the topic is relevant. If you want to optimize your time, leave behind sections 4, 5 and 7.4 – 7.8.

You are done with the exercises once you know the following topics

<ul>
<li>SQL injection</li>
<li>CSRF</li>
<li>XSS</li>
<li>smart use of sessions</li>
</ul>

You had better read also the following links, as far as information security is concerned:

<ul>
<li><a href="http://guides.rubyonrails.org/action_controller_overview.html#force-https-protocol">http://guides.rubyonrails.org/action_controller_overview.html#force-https-protocol</a></li>
<li><a href="http://guides.rubyonrails.org/action_controller_overview.html#log-filtering">http://guides.rubyonrails.org/action_controller_overview.html#log-filtering</a></li>
</ul>
</blockquote>

The documents above fail to stress that Rails <em>sanitates</em> (that is, escapes all the script and html tags) by default the input that is rendered on pages, so for instance if you try to input the javascript chunk <code> &lt;script&gt;alert('Evil XSS attack');&lt;/script&gt;</code> to describe the beer style, the code won't be executed, buy it will be rendered on the page 'as text':

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-7.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-7.png" alt="picture" style="max-width:100%;"></a>

If you take a look at the page source code, you'll notice that Rails has switched &lt; and &gt; signs of the HTML tags with the corresponding printing characters, where the input changes into normal text when it comes to the browser:

<div class="highlight highlight-ruby"><pre> <span class="pl-k">&amp;</span>lt;script<span class="pl-k">&amp;</span>gt;alert(<span class="pl-k">&amp;</span><span class="pl-c">#39;Evil XSS attack&amp;#39;);&amp;lt;/script&amp;gt;</span></pre></div>

The default sanitation can be 'disconnected' by making an explicit request with the help of the method <code>raw</code>, so that the contents are rendered on the page as they are. If you changed in the following the way the style description is rendered

<div class="highlight highlight-ruby"><pre><span class="pl-k">&lt;</span>p<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span><span class="pl-k">%=</span> raw(<span class="pl-smi">@style</span>.description) <span class="pl-s"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s">&lt;/p<span class="pl-pds">&gt;</span></span></pre></div>

the javascript code is executed while it is rendered:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-8.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-8.png" alt="picture" style="max-width:100%;"></a>

More info at <a href="http://www.railsdispatch.com/posts/security">http://www.railsdispatch.com/posts/security</a> and <a href="http://railscasts.com/episodes/204-xss-protection-in-rails-3">http://railscasts.com/episodes/204-xss-protection-in-rails-3</a>


<a id="user-content-metaohjelmointia-mielipanimoiden-ja-tyylin-refaktorointi" class="anchor" href="#metaohjelmointia-mielipanimoiden-ja-tyylin-refaktorointi" aria-hidden="true"><span class="octicon octicon-link"></span></a>Metaohjelmointia: mielipanimoiden ja tyylin refaktorointi

In the exercises 3 and 4 of week 4 (see <a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#teht%C3%A4v%C3%A4-3">https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#teht%C3%A4v%C3%A4-3</a>) you implemented the methods to find out a person's favourite brewery and beer style. The following is a straight solution to implement the methods <code>favorite_style</code> and <code>favorite_brewery</code>:

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">User</span>
  <span class="pl-c"># ...</span>

  <span class="pl-k">def</span> <span class="pl-en">favorite_brewery</span>
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?
    brewery_ratings <span class="pl-k">=</span> rated_breweries.inject([]) <span class="pl-k">do </span>|<span class="pl-smi">ratings</span>, <span class="pl-smi">brewery</span>|
      ratings <span class="pl-k">&lt;&lt;</span> {
        <span class="pl-c1">brewery:</span> brewery,
        <span class="pl-c1">rating:</span> rating_of_brewery(brewery) }
    <span class="pl-k">end</span>

    brewery_ratings.sort_by { |<span class="pl-smi">brewery</span>| brewery[<span class="pl-c1">:rating</span>] }.last[<span class="pl-c1">:brewery</span>]
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">favorite_style</span>
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?
    style_ratings <span class="pl-k">=</span> rated_styles.inject([]) <span class="pl-k">do </span>|<span class="pl-smi">ratings</span>, <span class="pl-smi">style</span>|
      ratings <span class="pl-k">&lt;&lt;</span> {
        <span class="pl-c1">style:</span> style,
        <span class="pl-c1">rating:</span> rating_of_style(style) }
    <span class="pl-k">end</span>

    style_ratings.sort_by { |<span class="pl-smi">style</span>| style[<span class="pl-c1">:rating</span>] }.last[<span class="pl-c1">:style</span>]
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">rated_breweries</span>
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.brewery }.uniq
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">rated_styles</span>
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.style }.uniq
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">rating_of_brewery</span>(<span class="pl-smi">brewery</span>)
    ratings_of_brewery <span class="pl-k">=</span> ratings.select <span class="pl-k">do </span>|<span class="pl-smi">r</span>|
      r.beer.brewery <span class="pl-k">==</span> brewery
    <span class="pl-k">end</span>
    ratings_of_brewery.map(<span class="pl-k">&amp;</span><span class="pl-c1">:score</span>).sum <span class="pl-k">/</span> ratings_of_brewery.count
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">rating_of_style</span>(<span class="pl-smi">style</span>)
    ratings_of_style <span class="pl-k">=</span> ratings.select <span class="pl-k">do </span>|<span class="pl-smi">r</span>|
      r.beer.style <span class="pl-k">==</span> style
    <span class="pl-k">end</span>
    ratings_of_style.map(<span class="pl-k">&amp;</span><span class="pl-c1">:score</span>).sum <span class="pl-k">/</span> ratings_of_style.count
  <span class="pl-k">end</span></pre></div>

Take a look at the method you can use to find out the best breweries. You have two auxiliary methods. The user's rated breweries (that is, the breweries where a user has rated at least one beer) can be accessed in the following way (see <a href="http://www.google.com">http://www.google.com</a> if you don't know the idea behind the map command):

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">rated_breweries</span>
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.brewery }.uniq
  <span class="pl-k">end</span></pre></div>

Another auxiliary method finds out the average value of a particular brewery (see the methods select and map, and check <a href="http://www.google.com">http://www.google.com</a> if you need):

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">rating_of_brewery</span>(<span class="pl-smi">brewery</span>)
    ratings_of_brewery <span class="pl-k">=</span> ratings.select <span class="pl-k">do </span>|<span class="pl-smi">r</span>|
      r.beer.brewery <span class="pl-k">==</span> brewery
    <span class="pl-k">end</span>
    ratings_of_brewery.map(<span class="pl-k">&amp;</span><span class="pl-c1">:score</span>).sum <span class="pl-k">/</span> ratings_of_brewery.count
  <span class="pl-k">end</span></pre></div>

The method to find the best brewery goes through all breweries first and it finds out the average value of their ratings. The result is a table like <code> [ { brewery: 'Koff', rating:10}, {brewery: 'Stadinpanimo', 27 }, {brewery:'Schlenkerla', rating:35}, {brewery:Karjala, rating:18}]</code> (the table does not contain brewery names but brewery objects, actually). The table is sorted against the value of <code>rating</code>. The method returns the first brewery of the last item of the sorted table, in our example the Schlenkerla object.

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite_brewery</span>
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?

    brewery_ratings <span class="pl-k">=</span> rated_breweries.inject([]) <span class="pl-k">do </span>|<span class="pl-smi">ratings</span>, <span class="pl-smi">brewery</span>|
      ratings <span class="pl-k">&lt;&lt;</span> {
        <span class="pl-c1">brewery:</span> brewery,
        <span class="pl-c1">rating:</span> rating_of_brewery(brewery) }
    <span class="pl-k">end</span>

    brewery_ratings.sort_by { |<span class="pl-smi">brewery</span>| brewery[<span class="pl-c1">:rating</span>] }.last[<span class="pl-c1">:name</span>]
  <span class="pl-k">end</span></pre></div>

The inject command which creates the table <code>brewery_ratings</code> is nothing but a compact way to write this:

<div class="highlight highlight-ruby"><pre>  brewery_ratings <span class="pl-k">=</span> []

  rated_breweries.each <span class="pl-k">do </span>|<span class="pl-smi">brewery</span>|
    object <span class="pl-k">=</span> {
      <span class="pl-c1">brewery:</span> brewery,
      <span class="pl-c1">rating:</span> rating_of_brewery(brewery)
    }
    brewery_ratings <span class="pl-k">&lt;&lt;</span> object
  <span class="pl-k">end</span></pre></div>

Notice that <code>favorite_style</code> works according the <em>exactly</em> same principle, and the method itself as well as the auxiliary methods it makes use of are actually only copy-paste of the code to find the best brewery.

Because our software has a wide range of tests, it's easy to refactor the copy-paste. Have a look at the following auxiliary methods first:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">rated_breweries</span>
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.brewery }.uniq
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">rated_styles</span>
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.style }.uniq
  <span class="pl-k">end</span></pre></div>

The only difference in the methods is with the code chunk of the <code>map</code> method, where the method calls a beer object that belongs to the rating. The method to call can also be given as parameter. In such case, instead of writing an explicit call, the method is called with the help of the <code>send</code> method:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">rated</span>(<span class="pl-smi">category</span>)
    ratings.map{ |<span class="pl-smi">r</span>| r.beer.send(category) }.uniq
  <span class="pl-k">end</span></pre></div>

The method can now be used in the following way:

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">037</span> <span class="pl-k">&gt;</span> u <span class="pl-k">=</span> <span class="pl-c1">User</span>.first
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :038 <span class="pl-k">&gt;</span> u.rated <span class="pl-c1">:style</span>
=&gt; [<span class="pl-c">#&lt;Style id: 1, name: "Lager", description: "Similar to the Munich Helles story, many European ...", created_at: "2015-02-07 18:12:03", updated_at: "2015-02-12 13:24:47"&gt; ...]</span>
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :039 <span class="pl-k">&gt;</span> u.rated <span class="pl-c1">:brewery</span>
 =&gt; [<span class="pl-c">#&lt;Brewery id: 1, name: "Koff", year: 1897, created_at: "2015-01-11 14:29:22", updated_at: "2015-02-12 14:02:01", active: true&gt;, #&lt;Brewery id: 4, name: "BrewDog", year: 2007, created_at: "2015-01-17 13:11:51", updated_at: "2015-02-12 14:02:01", active: true&gt;, #&lt;Brewery id: 3, name: "Weihenstephaner", year: 1042, created_at: "2015-01-11 14:29:22", updated_at: "2015-02-12 14:02:01", active: true&gt;, #&lt;Brewery id: 2, name: "Malmgard", year: 2001, created_at: "2015-01-11 14:29:22", updated_at: "2015-02-12 14:07:55", active: true&gt;]</span></pre></div>

You can give a parameter to the method to compute the average value of style and brewery ratings according to category, as before:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">rating_of</span>(<span class="pl-smi">category</span>, <span class="pl-smi">item</span>)
    ratings_of_item <span class="pl-k">=</span> ratings.select <span class="pl-k">do </span>|<span class="pl-smi">r</span>|
      r.beer.send(category) <span class="pl-k">==</span> item
    <span class="pl-k">end</span>
    ratings_of_item.map(<span class="pl-k">&amp;</span><span class="pl-c1">:score</span>).sum <span class="pl-k">/</span> ratings_of_item.count
  <span class="pl-k">end</span></pre></div>

So, first of all it looks for the ratings for the brewery or beer style that was given as parameter. After that, it computes the average value in the normal way.

The method works as espected:

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">051</span> <span class="pl-k">&gt;</span> lager <span class="pl-k">=</span> <span class="pl-c1">Style</span>.first
 =&gt; <span class="pl-c">#&lt;Style id: 1, name: "Lager", description: "Similar to the Munich Helles story, many European ..." &gt;</span>
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">052</span> <span class="pl-k">&gt;</span> u.rating_of(<span class="pl-c1">:style</span>, lager)
 =&gt; <span class="pl-c1">18</span></pre></div>

With the new auxiliary methods, you can easily create a method to find out users favourite brewery and favourite style, according to what parameter you give to the method:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite</span>(<span class="pl-smi">category</span>)
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?

    category_ratings <span class="pl-k">=</span> rated(category).inject([]) <span class="pl-k">do </span>|<span class="pl-smi">set</span>, <span class="pl-smi">item</span>|
      set <span class="pl-k">&lt;&lt;</span> {
        <span class="pl-c1">item:</span> item,
        <span class="pl-c1">rating:</span> rating_of(category, item) }
    <span class="pl-k">end</span>

    category_ratings.sort_by { |<span class="pl-smi">item</span>| item[<span class="pl-c1">:rating</span>] }.last[<span class="pl-c1">:item</span>]
  <span class="pl-k">end</span></pre></div>

Try out the console:

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">063</span> <span class="pl-k">&gt;</span> u.favorite <span class="pl-c1">:brewery</span>
 =&gt; <span class="pl-c">#&lt;Brewery id: 2, name: "Malmgard", year: 2001, created_at: "2015-01-11 14:29:22", updated_at: "2015-02-12 14:07:55", active: true&gt;</span>
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">065</span> <span class="pl-k">&gt;</span> u.favorite <span class="pl-c1">:style</span>
 =&gt; <span class="pl-c">#&lt;Style id: 3, name: "Baltic porter", description: "Porters of the late 1700's were quite strong compa...", created_at: "2015-02-07 18:12:03", updated_at: "2015-02-07 18:24:28"&gt;</span></pre></div>

The methods to find out favourite style and brewery can then be modified so that they will delegate their functionality to new methods:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite_brewery</span>
    favorite <span class="pl-c1">:brewery</span>
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">favorite_style</span>
    favorite <span class="pl-c1">:style</span>
  <span class="pl-k">end</span></pre></div>

In addition to getting rid of the copy-paste, the solution brings along also another benefit. If you define a new "attribute," like colour, you will simultaneously get a method to find the favourite colour:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite_color</span>
    favorite <span class="pl-c1">:color</span>
  <span class="pl-k">end</span></pre></div>


<a id="user-content-method_missing" class="anchor" href="#method_missing" aria-hidden="true"><span class="octicon octicon-link"></span></a>method_missing

In fact, it would aslo be possible to use methods like <code>favorite_style</code> and <code>favorite_brewery</code>  without defining them explicitly.

Comment away all the methods in your code for a second.

If you called an inexistent method of an object (not being defined in the class itself, in the parent classes or in any module contained in its class or parent classes), like

<div class="highlight highlight-ruby"><pre>jD4h4lsgC1drWwIy2O...<span class="pl-s"><span class="pl-pds">"</span>, admin: true, is_frozen: nil&gt;</span>
<span class="pl-s">2.0.0-p451 :069 &gt; u.paras_bisse</span>
<span class="pl-s">NoMethodError: undefined method `paras_bisse' for #&lt;User:0x000001059cb0c0&gt;</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/activemodel-4.1.5/lib/active_model/attribute_methods.rb:435:in `method_missing'</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/activerecord-4.1.5/lib/active_record/attribute_methods.rb:208:in `method_missing'</span>
<span class="pl-s">  from (irb):69</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/railties-4.1.5/lib/rails/commands/console.rb:90:in `start'</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/railties-4.1.5/lib/rails/commands/console.rb:9:in `start'</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/railties-4.1.5/lib/rails/commands/commands_tasks.rb:69:in `console'</span>
<span class="pl-s">2.0.0-p451 :070 &gt;</span></pre></div>

the result is that the Ruby translator calls a method called <code>method_missing</code> of that class, which has an unknown method name as parameter. All Ruby's classes inherit the class <code>Object</code> which defines the method <code>method_missing</code>. Classes may need to overwrite this method and get "methods" which are not existent, but that work as normal methods as far as the caller knows.

Rails uses <code>method_missing</code> internally in many situations. You can not overwrite it straight, you will have to delegate the <code>method_missing</code> calls to the parent class unless you want to handle them yourself.

Have a try and define the following <code>method_missing</code> for the class <code>User</code>:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">method_missing</span>(<span class="pl-smi">method_name</span>, <span class="pl-k">*</span><span class="pl-smi">args</span>, <span class="pl-k">&amp;</span><span class="pl-smi">block</span>)
    puts <span class="pl-s"><span class="pl-pds">"</span>nonexisting method <span class="pl-pse">#{</span><span class="pl-s1">method_name</span><span class="pl-pse"><span class="pl-s1">}</span></span> was called with parameters: <span class="pl-pse">#{</span><span class="pl-s1">args</span><span class="pl-pse"><span class="pl-s1">}</span></span><span class="pl-pds">"</span></span>
    <span class="pl-k">return</span> <span class="pl-k">super</span>
  <span class="pl-k">end</span></pre></div>

And see what happens:

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">072</span> <span class="pl-k">&gt;</span> u.paras_bisse
nonexisting method paras_bisse was called with <span class="pl-c1">parameters:</span> []
<span class="pl-c1">NoMethodError:</span> undefined method <span class="pl-s"><span class="pl-pds">`</span>paras_bisse' for #&lt;User:0x000001016af8e0&gt;</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/activemodel-4.1.5/lib/active_model/attribute_methods.rb:435:in <span class="pl-pds">`</span></span>method_missing<span class="pl-s"><span class="pl-pds">'</span></span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/activerecord-4.1.5/lib/active_record/attribute_methods.rb:208:in `method_missing<span class="pl-pds">'</span></span>
  from <span class="pl-k">/</span><span class="pl-c1">Users</span><span class="pl-k">/</span>mluukkai<span class="pl-k">/</span>kurssirepot<span class="pl-k">/</span>ratebeer<span class="pl-k">/</span>app<span class="pl-k">/</span>models<span class="pl-k">/</span>user.<span class="pl-c1">rb:</span><span class="pl-c1">30</span><span class="pl-c1">:in</span> <span class="pl-s"><span class="pl-pds">`</span>method_missing'</span>
<span class="pl-s">2.0.0-p451 :073 &gt;</span></pre></div>

As you see above, <code>method_missing</code> has been executed. You can overwrite it in the following way:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">method_missing</span>(<span class="pl-smi">method_name</span>, <span class="pl-k">*</span><span class="pl-smi">args</span>, <span class="pl-k">&amp;</span><span class="pl-smi">block</span>)
    <span class="pl-k">if</span> method_name <span class="pl-k">=~</span> <span class="pl-sr"><span class="pl-pds">/</span>^favorite_<span class="pl-pds">/</span></span>
      category <span class="pl-k">=</span> method_name[<span class="pl-c1">9</span>..<span class="pl-k">-</span><span class="pl-c1">1</span>].to_sym
      <span class="pl-v">self</span>.favorite category
    <span class="pl-k">else</span>
      <span class="pl-k">return</span> <span class="pl-k">super</span>
    <span class="pl-k">end</span>
  <span class="pl-k">end</span></pre></div>

All the method calls that start with <code>favorite_</code> and that are unknown will be handled in the following way: the part after the underscore will be separated and the obect will be called the method <code>favorite</code> so that the part after the underscore stands for the parameter defining the category.

Now the methods <code>favorite_brewery</code> and <code>favorite_style</code> "exist" and work properly:

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">076</span> <span class="pl-k">&gt;</span> u <span class="pl-k">=</span> <span class="pl-c1">User</span>.first
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :<span class="pl-c1">077</span> <span class="pl-k">&gt;</span> u.favorite_brewery.name
 =&gt; <span class="pl-s"><span class="pl-pds">"</span>Malmgard<span class="pl-pds">"</span></span>
<span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :078 <span class="pl-k">&gt;</span> u.favorite_style.name
  =&gt; <span class="pl-s"><span class="pl-pds">"</span>Baltic porter<span class="pl-pds">"</span></span></pre></div>

The problem now is that whatever method that starts with favorite_ "would work", but it would cause an error which is far from being optimal.

<div class="highlight highlight-ruby"><pre><span class="pl-c1">2.0</span>.<span class="pl-c1">0</span><span class="pl-k">-</span>p451 :079 <span class="pl-k">&gt;</span> u.favorite_movie
<span class="pl-c1">NoMethodError:</span> undefined method <span class="pl-s"><span class="pl-pds">`</span>movie' for #&lt;Beer:0x00000105a18690&gt;</span>
<span class="pl-s">  from /Users/mluukkai/.rvm/gems/ruby-2.0.0-p451/gems/activemodel-4.1.5/lib/active_model/attribute_methods.rb:435:in <span class="pl-pds">`</span></span>method_missing<span class="pl-s"><span class="pl-pds">'</span></span></pre></div>

Ruby provides various opportunities to define what <code>favorite_</code> methods are accepted. You could implement for instance this very Ruby-like way to define it:

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">User<span class="pl-e"> &lt; ActiveRecord::Base</span></span>
  <span class="pl-k">include</span> <span class="pl-c1">RatingAverage</span>

  favorite_available_by <span class="pl-c1">:style</span>, <span class="pl-c1">:brewery</span>

  <span class="pl-c"># ...</span>
<span class="pl-k">end</span></pre></div>

This is not the time to go too much deeper into it. This would be useful only if favorite_ methods could be used in other calsses too.

You can end here the implementation with method_missing and go back to the versions that had been commented away at the beginning of the chapter.

If the things that have been explained in this chapter are interesting for you, you can continue with the following:

<ul>
<li><a href="http://rubymonk.com/learning/books/5-metaprogramming-ruby-ascent">http://rubymonk.com/learning/books/5-metaprogramming-ruby-ascent</a></li>
<li><a href="http://rubymonk.com/learning/books/2-metaprogramming-ruby">http://rubymonk.com/learning/books/2-metaprogramming-ruby</a></li>
<li><a href="https://github.com/sathish316/metaprogramming_koans">https://github.com/sathish316/metaprogramming_koans</a></li>
<li>also the book <a href="http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104">Eloquent Ruby</a> is a good reference on the topic</li>
</ul>


<a id="user-content-tehtävien-palautus" class="anchor" href="#teht%C3%A4vien-palautus" aria-hidden="true"><span class="octicon octicon-link"></span></a>Tehtävien palautus

Commit all the changes that you have done and push the code in Github. Deploy also the newest version into Heroku.

You should mark that you have returned the exercises at <a href="http://wadrorstats2015.herokuapp.com/">http://wadrorstats2015.herokuapp.com/</a>
</article>
  </div>

</div>

<a href="#jump-to-line" rel="facebox[.linejump]" data-hotkey="l" style="display:none">Jump to Line</a>
<div id="jump-to-line" style="display:none">
  <form accept-charset="UTF-8" action="" class="js-jump-to-line-form" method="get"><div style="margin:0;padding:0;display:inline"><input name="utf8" type="hidden" value="&#x2713;" /></div>
    <input class="linejump-input js-jump-to-line-field" type="text" placeholder="Jump to line&hellip;" autofocus>
    <button type="submit" class="btn">Go</button>
</form></div>

        </div>

      </div><!-- /.repo-container -->
      <div class="modal-backdrop"></div>
    </div><!-- /.container -->
  </div><!-- /.site -->


    </div><!-- /.wrapper -->

      <div class="container">
  <div class="site-footer" role="contentinfo">
    <ul class="site-footer-links right">
        <li><a href="https://status.github.com/" data-ga-click="Footer, go to status, text:status">Status</a></li>
      <li><a href="https://developer.github.com" data-ga-click="Footer, go to api, text:api">API</a></li>
      <li><a href="https://training.github.com" data-ga-click="Footer, go to training, text:training">Training</a></li>
      <li><a href="https://shop.github.com" data-ga-click="Footer, go to shop, text:shop">Shop</a></li>
        <li><a href="https://github.com/blog" data-ga-click="Footer, go to blog, text:blog">Blog</a></li>
        <li><a href="https://github.com/about" data-ga-click="Footer, go to about, text:about">About</a></li>

    </ul>

    <a href="https://github.com" aria-label="Homepage">
      <span class="mega-octicon octicon-mark-github" title="GitHub"></span>
</a>
    <ul class="site-footer-links">
      <li>&copy; 2015 <span title="0.03953s from github-fe129-cp1-prd.iad.github.net">GitHub</span>, Inc.</li>
        <li><a href="https://github.com/site/terms" data-ga-click="Footer, go to terms, text:terms">Terms</a></li>
        <li><a href="https://github.com/site/privacy" data-ga-click="Footer, go to privacy, text:privacy">Privacy</a></li>
        <li><a href="https://github.com/security" data-ga-click="Footer, go to security, text:security">Security</a></li>
        <li><a href="https://github.com/contact" data-ga-click="Footer, go to contact, text:contact">Contact</a></li>
    </ul>
  </div>
</div>


    <div class="fullscreen-overlay js-fullscreen-overlay" id="fullscreen_overlay">
  <div class="fullscreen-container js-suggester-container">
    <div class="textarea-wrap">
      <textarea name="fullscreen-contents" id="fullscreen-contents" class="fullscreen-contents js-fullscreen-contents" placeholder=""></textarea>
      <div class="suggester-container">
        <div class="suggester fullscreen-suggester js-suggester js-navigation-container"></div>
      </div>
    </div>
  </div>
  <div class="fullscreen-sidebar">
    <a href="#" class="exit-fullscreen js-exit-fullscreen tooltipped tooltipped-w" aria-label="Exit Zen Mode">
      <span class="mega-octicon octicon-screen-normal"></span>
    </a>
    <a href="#" class="theme-switcher js-theme-switcher tooltipped tooltipped-w"
      aria-label="Switch themes">
      <span class="octicon octicon-color-mode"></span>
    </a>
  </div>
</div>



    

    <div id="ajax-error-message" class="flash flash-error">
      <span class="octicon octicon-alert"></span>
      <a href="#" class="octicon octicon-x flash-close js-ajax-error-dismiss" aria-label="Dismiss error"></a>
      Something went wrong with that request. Please try again.
    </div>


      <script crossorigin="anonymous" src="https://assets-cdn.github.com/assets/frameworks-5c08de317e4054ec200d36d3b1361ddd3cb30c05c9070a9d72862ee28ab1d7f9.js"></script>
      <script async="async" crossorigin="anonymous" src="https://assets-cdn.github.com/assets/github/index-bfb27c17548d6c05bbe36a476673446098f04766b4a2ce445e2ae9d5374622ff.js"></script>
      
      

  </body>
</html>

