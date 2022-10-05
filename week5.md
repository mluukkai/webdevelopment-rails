
You will continue to develop your application from the point you arrived at the end of week 4. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week [from the course repository](https://github.com/mluukkai/WebPalvelinohjelmointi2015/tree/master/malliv/viikko4). If you already got most of the previous week exercises done, it might be easier if you complement your own answer with the help of the material.

If you start working this week on the base of last week sample answer, copy the folder from the course repository (assuming you have alrady cloned it) and make a new repository of the folder with the application.

**Attention:** some Mac users have found problems with the pg-gem which Heroku needs. Gems are not needed locally and we defined to set them only in the production environment. If you have problems, you can set gems by adding the following expression to <code>bundle install</code>:

    bundle install --without production

This setting will be remembered later on, so a simple `bundle install` will be enough if you want to set up new dependences.


## Mashup: bar search

A great part of modern Internet services makes use of open interfaces which provides useful data to enrich applications functionality.

Interfaces for beers are also available, try to search for beer at http://www.programmableweb.com/ 

The best interface among the ones available seems to be http://www.programmableweb.com/api/brewery-db but its use is limited to a 400-day trial, and you should not use this time. Try Beermapping API (http://www.programmableweb.com/api/beer-mapping ja http://beermapping.com/api/), instead, which makes it possible to search for beer restaurants.

Applications which make use of beermaping API need a singular API key. You can retrieve a key at http://beermapping.com/api/request_key, a corresponding procedure is in use for the larger part of modern free interfaces.

The API's services are listed at the page http://beermapping.com/api/reference/

For instance, you can find out the beer restaurants in a defined location by making an HTTP get request for the address <code>http://beermapping.com/webservice/loccity/[apikey]/[city]<location></code>

The location is picked as a part of the URL.

You can make requests with the browser or from the command line with the curl program. You'll find out beer restaurants in Espoo in the following way:

```ruby
mbp-18:ratebeer mluukkai$ curl http://beermapping.com/webservice/loccity/96ce1942872335547853a0bb3b0c24db/espoo
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>12411</id><name>Gallows Bird</name><status>Brewery</status><reviewlink>http://beermapping.com/maps/reviews/reviews.php?locid=12411</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=12411&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=12411&amp;d=1&amp;type=norm</blogmap><street>Merituulentie 30</street><city>Espoo</city><state></state><zip>02200</zip><country>Finland</country><phone>+358 9 412 3253</phone><overall>91.66665</overall><imagecount>0</imagecount></location></bmp_locations>mbp-18:ratebeer mluukkai$
```

As you'll notice, the response is in XML. This is a bit outdated because the currently most popular format to exchange information among Web services is Json.

If you use your browser, you'll see the returned XML in a form which can be read more easily for humans:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-1.png)

**ATTENTION: do not use the API key which is shown here but register one for your own.**

**ATTENTION2:** the service is _extremely_ slow at times. Instead, you can use the 'cache memory service' at which provides the http://stark-oasis-9187.herokuapp.com/api/. This provides the same data and was made just for the course. For instance, you find the data about Helsinki from the cache service at the URL [http://stark-oasis-9187.herokuapp.com/api/helsinki]
(http://stark-oasis-9187.herokuapp.com/api/helsinki).

If you look for a town which was had been retrieven before, the cache memory server will return the result it had stored. Instead, if you look for a town which is unknown to the cache memory server, it will ask for information from the Beermapping service first. The operation will take much longer in such cases. The cache memory server has not been tested much, so you might find bugs. If so, report about it.

Make now the search funtionality for your application beer restaurants.

Create a page for this at the address places, so go to route.rb and define

    get 'places', to: 'places#index'

and created the controller:

```ruby
class PlacesController < ApplicationController
  def index
  end
end
```

plus the view app/views/places/index.html.erb, which will only look like a search box at first:

```erb
<h1>Beer places search</h1>

<%= form_tag places_path do %>
  city <%= text_field_tag :city, params[:city] %>
  <%= submit_tag "Search" %>
<% end %>
```

The form sends HTTP POST requests to places_path. So define an appropriate route for it in routes.rb

    post 'places', to:'places#search'

In this way, you defined that the method name is <code>search</code>. Extend the controller like below:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    render :index
  end
end
```

THe idea is that the <code>search</code> method retrieves a brewery list from beermapping API. The breweries will then be added to index.html and that's why the method <code>search</code> renders the view template <code>index</code> at the end.

The <code>search</code> method has to make an HTTP request from the controller to the beermapping API page. The best way to make HTTP requests in Ruby is using the HTTParty gem, see https://github.com/jnunemaker/httparty. Add the following to your Gemfile:

    gem 'httparty'

Set up the gem by running the usual command from the command line, <code>bundle install</code>

Try to look for Helsinki restaurants by hand from the console now (remember to restart your console):

```ruby
2.0.0-p451 :001 > api_key = "96ce1942872335547853a0bb3b0c24db"
2.0.0-p451 :002 > url = "http://beermapping.com/webservice/loccity/#{api_key}/"
2.0.0-p451 :003 > HTTParty.get url+"helsinki"
```

**ATTENTION:** you will be able to use the cache memory server from now on, defining <code>url = http://stark-oasis-9187.herokuapp.com/api/</code>

The call will return an object of the class <code>HTTParty::Response</code>. The object can be enquired about the headers concerning the answer:

```ruby
2.0.0-p451 :004 > response = HTTParty.get url+"helsinki"
2.0.0-p451 :005 > response.headers
 => {"date"=>["Sat, 07 Feb 2015 12:20:01 GMT"], "server"=>["Apache"], "expires"=>["Mon, 26 Jul 1997 05:00:00 GMT"], "last-modified"=>["Sat, 07 Feb 2015 12:20:01 GMT"], "cache-control"=>["no-store, no-cache, must-revalidate", "post-check=0, pre-check=0"], "pragma"=>["no-cache"], "vary"=>["Accept-Encoding"], "content-length"=>["4887"], "connection"=>["close"], "content-type"=>["text/xml"]}
2.0.0-p451 :006 >
```

and the HTTP call status code:

```ruby
2.0.0-p451 :006 > response.code
 => 200
```

See http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html, the status code is 200 now, which is fine, meaning that the call has succeeded.

The answer object method <code>parsed_response</code> will return the data as Ruby's hash:

```ruby
2.0.0-p451 :007 > response.parsed_response
 => {"bmp_locations"=>{"location"=>[{"id"=>"6742", "name"=>"Pullman Bar", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=6742", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm", "street"=>"Kaivokatu 1", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"+358 9 0307 22", "overall"=>"72.500025", "imagecount"=>"0"}, {"id"=>"6743", "name"=>"Belge", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=6743", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6743&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6743&d=1&type=norm", "street"=>"Kluuvikatu 5", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"+358 10 766 35", "overall"=>"67.499925", "imagecount"=>"1"}, {"id"=>"6919", "name"=>"Suomenlinnan Panimo", "status"=>"Brewpub", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=6919", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6919&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6919&d=1&type=norm", "street"=>"Rantakasarmi", "city"=>"Helsinki", "state"=>nil, "zip"=>"00190", "country"=>"Finland", "phone"=>"+358 9 228 5030", "overall"=>"69.166625", "imagecount"=>"0"}, {"id"=>"12408", "name"=>"St. Urho's Pub", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=12408", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=12408&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=12408&d=1&type=norm", "street"=>"Museokatu 10", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"+358 9 5807 7222", "overall"=>"95", "imagecount"=>"0"}, {"id"=>"12409", "name"=>"Kaisla", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=12409", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=12409&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=12409&d=1&type=norm", "street"=>"Vilhonkatu 4", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"+358 10 76 63850", "overall"=>"83.3334", "imagecount"=>"0"}, {"id"=>"12410", "name"=>"Pikkulintu", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=12410", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=12410&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=12410&d=1&type=norm", "street"=>"Klaavuntie 11", "city"=>"Helsinki", "state"=>nil, "zip"=>"00910", "country"=>"Finland", "phone"=>"+358 9 321 5040", "overall"=>"91.6667", "imagecount"=>"0"}, {"id"=>"18418", "name"=>"Bryggeri Helsinki", "status"=>"Brewpub", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=18418", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=18418&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=18418&d=1&type=norm", "street"=>"Sofiankatu 2", "city"=>"Helsinki", "state"=>nil, "zip"=>"FI-00170", "country"=>"Finland", "phone"=>"010 235 2500", "overall"=>"0", "imagecount"=>"0"}, {"id"=>"18844", "name"=>"Stadin Panimo", "status"=>"Brewery", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=18844", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=18844&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=18844&d=1&type=norm", "street"=>"Kaasutehtaankatu 1, rakennus 6", "city"=>"Helsinki", "state"=>nil, "zip"=>"00540", "country"=>"Finland", "phone"=>"09 170512", "overall"=>"0", "imagecount"=>"0"}, {"id"=>"18855", "name"=>"Panimoravintola Bruuveri", "status"=>"Brewpub", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=18855", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=18855&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=18855&d=1&type=norm", "street"=>"Fredrikinkatu 63AB", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"09 685 66 88", "overall"=>"0", "imagecount"=>"0"}]}}
2.0.0-p451 :008 >
```

Even though the server returns the answer in XML format, the gem HTTParty parses it, and and makes it possible that it is handled straight in the best form as Ruby hash.

You can get the restaurant table returned by the call in the following way:

```ruby
2.0.0-p451 :013 > places = response.parsed_response['bmp_locations']['location']
2.0.0-p451 :014 > places.size => 9
```

So, it knows 9 places in Helsinki. Inspect the first one:

```ruby
2.0.0-p451 :015 > places.first
 => {"id"=>"6742", "name"=>"Pullman Bar", "status"=>"Beer Bar", "reviewlink"=>"http://beermapping.com/maps/reviews/reviews.php?locid=6742", "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5", "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm", "street"=>"Kaivokatu 1", "city"=>"Helsinki", "state"=>nil, "zip"=>"00100", "country"=>"Finland", "phone"=>"+358 9 0307 22", "overall"=>"72.500025", "imagecount"=>"0"}
 2.0.0-p451 :016 > places.first.keys
 => ["id", "name", "status", "reviewlink", "proxylink", "blogmap", "street", "city", "state", "zip", "country", "phone", "overall", "imagecount"]
2.0.0-p451 :017 >
```

The last command <code>locations.first.keys</code> tells you the fields which belong to restaurants.

Create an object only to present the breweries, and call it by the name <code>Place</code>. Put the class in the models folder.

```ruby
class Place
  include ActiveModel::Model

  attr_accessor :id, :name, :status, :reviewlink, :proxylink, :blogmap, :street, :city, :state, :zip, :country, :phone, :overall, :imagecount
end
```

Because that is not a "normal" class which inherits <code>ActiveRecord::Base</code> you will have to define object and attributes with the help of the method <code>attr_accessor</code>. The method creates "a getter and and setter" for each symbol in parameter, which means the methods to read and update the attribute value.

The object has been provided with an attribute for all the keys returned by beermapping for each restaurant.

The class contains the method <code>ActiveModel::Model</code> (see http://api.rubyonrails.org/classes/ActiveModel/Model.html), which makes it possible to initialize all the attributes in the constructor using the hash returned by the API. This means that the data returned by the API will help you to create Place objects in the following way:

```ruby
2.0.0-p451 :019 > baari = Place.new places.first
 => #<Place:0x000001035a2040 @id="6742", @name="Pullman Bar", @status="Beer Bar", @reviewlink="http://beermapping.com/maps/reviews/reviews.php?locid=6742", @proxylink="http://beermapping.com/maps/proxymaps.php?locid=6742&d=5", @blogmap="http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm", @street="Kaivokatu 1", @city="Helsinki", @state=nil, @zip="00100", @country="Finland", @phone="+358 9 0307 22", @overall="72.500025", @imagecount="0">
2.0.0-p451 :020 > baari.name
 => "Pullman Bar"
2.0.0-p451 :021 > baari.street
 => "Kaivokatu 1"
2.0.0-p451 :022 >
```

So write the code to initialize the controller. Hard-code the search criteria so that it starts from Helsinki and create only one Place object for the first place which is retrieven:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "96ce1942872335547853a0bb3b0c24db"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    # tai vaihtoehtoisesti
    # url = 'http://stark-oasis-9187.herokuapp.com/api/'

    response = HTTParty.get "#{url}helsinki"
    places_from_api = response.parsed_response["bmp_locations"]["location"]
    @places = [ Place.new(places_from_api.first) ]

    render :index
  end
end
```

Modify app/views/places/index.html.erb so that it shows the restaurants which have been found:

```erb
<h1>Beer places search</h1>

<%= form_tag places_path do %>
  city <%= text_field_tag :city, params[:city] %>
  <%= submit_tag "Search" %>
<% end %>

<% if @places %>
  <ul>
    <% @places.each do |place| %>
      <li><%=place.name %></li>
    <% end %>
  </ul>
<% end %>
```

The code looks like working (notice that you will have to restart Rails server so that the gem HTTParty will be set up).

Expand the code to show all breweries and to make use of a form parameter to search against the locality:

```ruby
  def search
    api_key = "96ce1942872335547853a0bb3b0c24db"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"

    @places = response.parsed_response["bmp_locations"]["location"].inject([]) do | set, place |
      set << Place.new(place)
    end

    render :index
  end
```

The application works, but it will display an error if the locality does not have restaurants.

If you use the debugger, you will see what the locality list returned by the API looks like in such cases:

```ruby
{"id"=>nil, "name"=>nil, "status"=>nil, "reviewlink"=>nil, "proxylink"=>nil, "blogmap"=>nil, "street"=>nil, "city"=>nil, "state"=>nil, "zip"=>nil, "country"=>nil, "phone"=>nil, "overall"=>nil, "imagecount"=>nil}
```

So the return value is an hash. But if the search finds beers, the return value is a table which contains hashes. Fix the code taking this into consideration. The code will also take into consideration the case where the API returns a hash which does not correspond to an unexistent place anyway. This is the case when a town has only one restaurant.

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "96ce1942872335547853a0bb3b0c24db"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"
    places_from_api = response.parsed_response["bmp_locations"]["location"]

    if places_from_api.is_a?(Hash) and places_from_api['id'].nil?
      redirect_to places_path, :notice => "No places in #{params[:city]}"
    else
      places_from_api = [places_from_api] if places_from_api.is_a?(Hash)
      @places = places_from_api.inject([]) do | set, location|
        set << Place.new(location)
      end
      render :index
    end
  end

end
```

The code looks quite bad so far, but we'll come back to this issue in a moment. Make sure the page shows more information about the bars. Define the keys which should be shown as static methods of the Place class:

```ruby
class Place
  include ActiveModel::Model
  attr_accessor :id, :name, :status, :reviewlink, :proxylink, :blogmap, :street, :city, :state, :zip, :country, :phone, :overall, :imagecount

  def self.rendered_fields
    [:id, :name, :status, :street, :city, :zip, :country, :overall ]
  end
end
```

below the improved code for index.html.erb:

```erb
<p id="notice"><%= notice %></p>

<%= form_tag places_path do %>
  city <%= text_field_tag :city, params[:city] %>
  <%= submit_tag "Search" %>
<% end %>

<% if @places %>
  <table>
    <thead>
      <% Place.rendered_fields.each do |f| %>
        <td><%=f %></td>
      <% end %>
    </thead>
    <% @places.each do |place| %>
      <tr>
        <% Place.rendered_fields.each do |f| %>
          <td><%= place.send(f) %></td>
        <% end %>
      </tr>
    <% end %>
  </table>
<% end %>
```

Your application hides a small issue still. If you try to look for New York beer restaurants you will run into troubles. The space has to be replaced with the code %20 in the URL. The change should not be made 'by hand', because the space is not the only character which has to be coded into the URL. As you might have thought, Rails provides a ready-made solution for this, the method <code>ERB::Util.url_encode</code>. Try the method out from the console:

```ruby
2.0.0-p451 :022 > ERB::Util.url_encode("St John's")
 => "St%20John%27s"
2.0.0-p451 :023 >
```

Implement the code change by replacing the HTTP GET request line with the following:

```ruby
    response = HTTParty.get "#{url}#{ERB::Util.url_encode(params[:city])}"
```

> ## Exercise 1
>
> Implement the code above in your program. Also add a link to the beer restaurants search page in the navigation bar.

## Refactoring your Places controller

Rails controllers should not include application logic. It is a best practice to put external APIs in their own class. Place the class into the lib folder (in the file beermapping_api.rb):

```ruby
class BeermappingApi
  def self.places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.inject([]) do | set, place |
      set << Place.new(place)
    end
  end

  def self.key
    "96ce1942872335547853a0bb3b0c24db"
  end
end
```

So the class defines a static method which returns a table of the beer restaurants which have been found in the towns defined by the parameter. If no restaurant is found, the table will be empty. The APLI class is not in its best format yet, because you cannot know completely what other methods you need.

**ATTENTION:** if you haven't done [exercise 15 of week 2](https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko2.md#teht%C3%A4v%C3%A4-15) or if you placed the module of the exercise into the folder _app/models/concerns_, go to the class <code>Application</code> in the file _config/application.rb and add the line <code>config.autoload_paths += Dir["#{Rails.root}/lib"]</code> in the class definition so that Rails will load the lib folder code and make it available for the application classes.

The controller will be looking neat, by now:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    @places = BeermappingApi.places_in(params[:city])
    if @places.empty?
      redirect_to places_path, notice: "No locations in #{params[:city]}"
    else
      render :index
    end
  end
end
```

## Testing beer restaurant search

You will want to create Rspec tests for the functionality you've implemented. The new functionality makes use of external services. Tests should be written in any case without making use of the external services. Luckily, it is easy to replace an external interface with Rails stub component.

You will want to devide the tests in two parts. The class <code>BeermappingApi</code> encapsulates the external interface, so replace its functionality by hard coding a new one with the help of Stub. The test will check whether the places page works properly, assuming that the <code>BeermappingApi</code> components works properly.

Test the <code>BeermappingApi</code> component first with a singular test written with Rspec.

So get started with your tests on the functionality of the web page places. Create a file for the test, /spec/features/places_spec.rb

```ruby
require 'rails_helper'

describe "Places" do
  it "if one is returned by the API, it is shown at the page" do
    allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
      [ Place.new( name:"Oljenkorsi", id: 1 ) ]
    )

    visit places_path
    fill_in('city', with: 'kumpula')
    click_button "Search"

    expect(page).to have_content "Oljenkorsi"
  end
end
```

The test gets started with an interesting command:

```ruby
    allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
      [ Place.new( name:"Oljenkorsi", id: 1 ) ]
    )
```

The command "hard codes" a table containing one Place object in answer to the method <code>places_in</code> of the class <code>BeermappingApi</code>, as if the method was called by the parameter "kumpula".

When the tests make an HTTP request to the places controller, and as the controller calls the API method <code>places_in</code>, instead of executing the real code the places controller is returned a hard-coded answer.

In case you find the following error while running your tests

```ruby
mbp-18:ratebeer mluukkai$ rspec spec/features/places_spec.rb
/Users/mluukkai/.rbenv/versions/2.0.0-p247/lib/ruby/gems/2.0.0/gems/activerecord-4.0.2/lib/active_record/migration.rb:379:in `check_pending!': Migrations are pending; run 'bin/rake db:migrate RAILS_ENV=test' to resolve this issue. (ActiveRecord::PendingMigrationError)
…
```

the reason is that you haven't executed all the database migrations in the test environment. You will fix the problem with the command <code>rake db:test:prepare</code>. If you run into any error concerning the gem versions (as it happened to me personally), execute <code>bundle update</code>.

> ## Exercise 2
>
> Extend your tests to cover the following exceptions:
> * if the API returns various beer restaurants, all of them are shown on the page
> * if the API does not find any beer restaurant in town (so if the return value is an empty table), the page should show the message "No locations in _place name_"
>
> The message that no location was found should be implemente _with a notice_, see  https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko3.md#kirjautumisen-hienos%C3%A4%C3%A4t%C3%B6%C3%A4

Let's move to testing the class <code>BeermappingApi</code>. The class makes an HTTP GET request for the Beermapping service with the help of the HTTParty library. You could stub the HTTParty get method like in the previous exercise. This would not be nice, though, because the method returns an <code>HTTPartyResponse</code> object and creating that by hand with stub is not the most beautiful thing to do.

A better option is using _webmock_ https://github.com/bblimke/webmock/, which makes it possible to stubbing at the level of the library used by HTTParty.

Get started with the gem by adding the line <code>gem 'webmock'</code> to the Gemfile **test-scope**:


```ruby
group :test do
    # ...
    gem 'webmock'
end
```

**ATTENTION: webmock has to be defined _only_ into test-scope, otherwise it will prevent all the HTTP requests made by your application!**

Run <code>bundle install</code>.

Add also the following line to the file ```spec/rails_helper.rb```:

```ruby
require 'webmock/rspec'
```

Using the webmock library is easy, altogether. For instance, the command below stubs the GET request information about 'Lapin kulta' (which is defined with regexp <code>/.*</code>) in XML form to _each_ URL:

```ruby
stub_request(:get, /.*/).to_return(body:"<beer><name>Lapin kulta</name><brewery>Hartwall</brewery></beer>", headers:{ 'Content-Type' => "text/xml" })
```

So if you called <code>HTTParty.get("http://www.google.com")</code> after the command, you'd see the following

```xml
<beer>
  <name>Lapin kulta</name>
  <brewery>Hartwall</brewery>
</beer>
```

So you need the right "hard-coded" data for tests, the data which represents the XML returned by the Beermapping service HTTP GET request.

A way to generate a test input is to ask the interface itself for it, so make an HTTP GET request with the <code>curl</code> command from the command line:

```ruby
mbp-18:ratebeer mluukkai$ curl http://beermapping.com/webservice/loccity/96ce1942872335547853a0bb3b0c24db/espoo
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>12411</id><name>Gallows Bird</name><status>Brewery</status><reviewlink>http://beermapping.com/maps/reviews/reviews.php?locid=12411</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=12411&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=12411&amp;d=1&amp;type=norm</blogmap><street>Merituulentie 30</street><city>Espoo</city><state></state><zip>02200</zip><country>Finland</country><phone>+358 9 412 3253</phone><overall>91.66665</overall><imagecount>0</imagecount></location></bmp_locations>
```

Now you could copy-paste the information returned in XML form by the HTTP reqest to your test. If you want to be sure you place the XML in the right string, you should use a quite particular syntax
see http://blog.jayfields.com/2006/12/ruby-multiline-strings-here-doc-or.html where the string is place between <code><<-END_OF_STRING</code> and <code>END_OF_STRING</code> väliin.

You find below the test code which should be placed into spec/lib/beermapping_api_spec.rb (deciding to place the code in the lib subfolder because the test destination is an auxiliary class in the lib folder):

```ruby
require 'rails_helper'

describe "BeermappingApi" do
  it "When HTTP GET returns one entry, it is parsed and returned" do

    canned_answer = <<-END_OF_STRING
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>12411</id><name>Gallows Bird</name><status>Brewery</status><reviewlink>http://beermapping.com/maps/reviews/reviews.php?locid=12411</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=12411&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=12411&amp;d=1&amp;type=norm</blogmap><street>Merituulentie 30</street><city>Espoo</city><state></state><zip>02200</zip><country>Finland</country><phone>+358 9 412 3253</phone><overall>91.66665</overall><imagecount>0</imagecount></location></bmp_locations>
    END_OF_STRING

    stub_request(:get, /.*espoo/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

    places = BeermappingApi.places_in("espoo")

    expect(places.size).to eq(1)
    place = places.first
    expect(place.name).to eq("Gallows Bird")
    expect(place.street).to eq("Merituulentie 30")
  end

end
```

So the test first defines that the HTTP GET request for the URL ending in the string "espoo" (which was defined through the regexp <code>.*espoo/</code>) should return an hardcoded XML; it defines that the information returned with the header should be in XML form. Without this definition the HTTParty library will not know how to parse correctly the data of the HTTP request.


The test itself happens directly after checking the table returned by the BeermappingApi method <code>places_in</code>.

*Attention:* in the test you only stubbed the HTTP GET calls for the URLs that ended with "espoo" (<code>/.*espoo/</code>). If the test execution causes any other kind of HTTP call, the test will point this out:

```ruby
) BeermappingApi When HTTP GET returns no entries, an empty array is returned
     Failure/Error: places = BeermappingApi.places_in("kumpula")
     WebMock::NetConnectNotAllowedError:
       Real HTTP connections are disabled. Unregistered request: GET http://beermapping.com/webservice/loccity/96ce1942872335547853a0bb3b0c24db/kumpula

       You can stub this request with the following snippet:

       stub_request(:get, "http://beermapping.com/webservice/loccity/96ce1942872335547853a0bb3b0c24db/kumpula").
         to_return(:status => 200, :body => "", :headers => {})
```

As you'll understand from the error message, you can also stub the HTTP call for the singular URL string with the help of the comand <code>stub_request</code>. The same test can also contain various <code>stub_request</code> calls, all defining answers for different URL requests.

> ## Exercise 3
>
> Extent the tests to include the cases below
> * if HTTP GET does not return any place, it should return an empty table
> * if HTTP GET returns various places, the method <code>places_in</code> should return a table of Place objects containing all the restaurants in XML form returned by the HTTP call.
>
> The stubbed answers should be formed again with the help of the curl command with the API requests

Stubbing and mocking methods and whole objects is a vast area of studies. You can read more on the topic in connection to Rspec at the following link http://rubydoc.info/gems/rspec-mocks/

Names like stub or mock objects or "stubbing and mocking" are used rather carelessly. Luckily, Rails community uses the terms properly. To tell a long story short, stubs are object where method answers have been hard-coded beforehand. Mock also provide hard-coded answers as stubs, but in addition to doing it, Mocks help you to define your expectations on how should their methods be called. If the objects which have to be tested do not call Mock methods as expected, this will produce an error.

More about Mocks and Stubs at: http://martinfowler.com/articles/mocksArentStubs.html

## Execution capacity optimization

Your application will currently be making requests to the beermapping service every time that the restaurants of a city are retrieved. You could improve the application memorizing recent searches.

Rails provides application with cache simple to use based on the key-value combination principle. Try on your console:

```ruby
2.0.0-p451 :001 > Rails.cache.write "avain", "arvo"
 => true
2.0.0-p451 :002 > Rails.cache.read "avain"
 => "arvo"
2.0.0-p451 :003 > Rails.cache.read "kumpula"
 => nil
2.0.0-p451 :004 > Rails.cache.write "kumpula", Place.new(name:"Oljenkorsi")
 => true
2.0.0-p451 :005 > Rails.cache.read "kumpula"
 => #<Place:0x00000104628608 @name="Oljenkorsi">
```

You can store whatever you want in cache. And the interface is really simple, see http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html

In addition to the methods <code>read</code> and <code>write</code> Rails cache proves the method <code>fetch</code> which is great in some situations. In addition to the key which has to be retrieved from the cache memory, the method is given a code chunk which is executed and stored into the key value _if_ the key value was empty.

For instance, the command <code>Rails.cache.fetch("first_user") { User.first }</code> retrieves the object stored in the key *first_user* from cache. If no value was stored in the key, it executes the command <code>User.first</code>, and the object returned will be set as the key value. See an example below:


```ruby
2.0.0-p451 :006 > Rails.cache.fetch("first_user") { User.first }
  User Load (0.7ms)  SELECT  "users".* FROM "users"   ORDER BY "users"."id" ASC LIMIT 1
 => #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 18:37:23", password_digest: "$2a$10$A6KEp02KHLMrpEkij9zcKu/wOjD4h4lsgC1drWwIy2O...">
2.0.0-p451 :007 > Rails.cache.fetch("first_user") { User.first }
 => #<User id: 1, username: "mluukkai", created_at: "2015-01-24 14:20:10", updated_at: "2015-01-24 18:37:23", password_digest: "$2a$10$A6KEp02KHLMrpEkij9zcKu/wOjD4h4lsgC1drWwIy2O...">
2.0.0-p451 :008 >
```

The first method call will search from the database and save the object in the cache memory. The fllowing call will receive the key object straight form the cache memory.

Rails cache saves key-value pairs in the file system by default. How the cache stores objects is being configured however, see http://guides.rubyonrails.org/caching_with_rails.html#cache-stores

Storing cache data in the file system during production is not optimal in terms of execution capability. A better option is [Memcached](http://memcached.org/) for instance; read more at https://devcenter.heroku.com/articles/building-a-rails-3-application-with-memcache

**Attention:** because our tests will soon start to test the code which makes use of Rails.cache, you'd better configure your cache to use the _central memory_ instead of file system to store information during tests. You can do it by adding the following line to the file _config/environments/test.rb_:

```ruby
config.cache_store = :memory_store
```

If you don't make such change, tests which make use of cache will not work in Travis because Travis uses a readonly file system.

Modify the class <code>BeermappingApi</code> so that it will save the request results in the cache memory. If a request concerns a city which is available in cache, the result is returned from cache.

```ruby
class BeermappingApi
  def self.places_in(city)
    city = city.downcase
    Rails.cache.fetch(city) { fetch_places_in(city) }
  end

  private

  def self.fetch_places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.inject([]) do | set, place |
      set << Place.new(place)
    end
  end

  def self.key
    "96ce1942872335547853a0bb3b0c24db"
  end
end
```

The lowercase city name is used as key.
The method <code>fetch</code> which returns the information about beer restaurants of a locality from cache _if_ the can be found in cache. Otherwise, the code contained in the second parameter (<code>fetch_places_in(city)</code>) will be exectuted, retriving information and storing it into cache.

If you do a search for New York beer restaurants twice in a row, you'll see that the answer will be returned much faster the second time.

You have access to the data in the application cache memory also from console:

```ruby
2.0.0-p451 :010 > Rails.cache.read("helsinki").map(&:name)
 => ["Pullman Bar", "Belge", "Suomenlinnan Panimo", "St. Urho's Pub", "Kaisla", "Pikkulintu", "Bryggeri Helsinki", "Stadin Panimo", "Panimoravintola Bruuveri"]
2.0.0-p451 :011 >
```

It is also possible to delete the value stored in a defined key by hand from the console, in case you need:

```ruby
2.0.0-p451 :011 > Rails.cache.delete("helsinki")
 => true
2.0.0-p451 :012 > Rails.cache.read("helsinki")
 => nil
2.0.0-p451 :013 >
```

## Outdated data

The problem with cache memory is when it comes to outdated data. So sometimes one may add restaurants to the beermapping page, and your cache memory will maintain the old data. Then you should make sure that the cache memory will not contain too old data.

One option is clearing the cache memory data with the command:

    Rails.cache.clear

A better solution is defining an expiring lifetime for the data and save it in the cache memory.

> ## Exercise 4
>
> ### This is not the most important exercise of the week, so do not get stuck with it if you get problems
>
>  Define the expiring lifetime for the restaurant data that are saved in the cache memory, 1 week for instance. When you test the exercise, you should use a shorter lifespam however, like one minute.
>
> Passing the exercise does not require major changes in your code, you only need to fix _one_ line in fact. You find useful hints at http://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-store. Other information to manage the time settings at http://guides.rubyonrails.org/active_support_core_extensions.html#time
>
> **Attention:** as usual, you will have to test the expiring lifetime settings by hand from console!
>
> **Attention2:** if you mess up with the cache memory, remember <code>Rails.cache.clear</code> and <code>Rails.cache.delete avain</code>

## Tests and cache

In exercise 3, you made tests for the class <code>Beermappingapi</code> with the help of Webmock. It is god to point out that cache memory affects tests and separately you may want to test the situations where data are not found from the cache memory (cache miss) and where they are already in cache (cache hit):

```ruby
require 'spec_helper'

describe "BeermappingApi" do

  describe "in case of cache miss" do

    before :each do
      Rails.cache.clear
    end

    it "When HTTP GET returns one entry, it is parsed and returned" do
      canned_answer = <<-END_OF_STRING
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>13307</id><name>O'Connell's Irish Bar</name><status>Beer Bar</status><reviewlink>http://beermapping.com/maps/reviews/reviews.php?locid=13307</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=13307&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=13307&amp;d=1&amp;type=norm</blogmap><street>Rautatienkatu 24</street><city>Tampere</city><state></state><zip>33100</zip><country>Finland</country><phone>35832227032</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
      END_OF_STRING

      stub_request(:get, /.*tampere/).to_return(body: canned_answer, headers: {'Content-Type' => "text/xml"})

      places = BeermappingApi.places_in("tampere")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("O'Connell's Irish Bar")
      expect(place.street).to eq("Rautatienkatu 24")
    end
  end

  describe "in case of cache hit" do

    it "When one entry in cache, it is returned" do
      canned_answer = <<-END_OF_STRING
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>13307</id><name>O'Connell's Irish Bar</name><status>Beer Bar</status><reviewlink>http://beermapping.com/maps/reviews/reviews.php?locid=13307</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=13307&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=13307&amp;d=1&amp;type=norm</blogmap><street>Rautatienkatu 24</street><city>Tampere</city><state></state><zip>33100</zip><country>Finland</country><phone>35832227032</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
      END_OF_STRING

      stub_request(:get, /.*tampere/).to_return(body: canned_answer, headers: {'Content-Type' => "text/xml"})

      # ensure that data found in cache
      BeermappingApi.places_in("tampere")

      places = BeermappingApi.places_in("tampere")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("O'Connell's Irish Bar")
      expect(place.street).to eq("Rautatienkatu 24")
    end
  end
end
```

The test is very redundant and it should be refactored, but won't stop for now.

**One more thing to keep in mind:** because you are testing a code which makes use of Rails.cache, you'd better configuring the cache so that it makes use of the __central memory__ instead of the file system when it saves during tests. You can implement this adding the following line in the file _config/environments/test.rb_

```ruby
config.cache_store = :memory_store
```

If you don't implement this change, tests that make use of cache will not work in Travis, because Travis makes use of a readonly file system.

## Saving application-specific data

The API key is written in your application code so far. This is not too smart however. There are many options to save the configuration information in Rails, see for instance http://quickleft.com/blog/simple-rails-app-configuration-settings

The best option to save application-specific and not too complex data are environment variables. See the example below:

Set up <code>APIKEY</code> to the environment variable from the command line first.

```ruby
mbp-18:ratebeer mluukkai$ export APIKEY="96ce1942872335547853a0bb3b0c24db"
```

Rails applications have access to environment variables through the hash variable <code<ENV</code>:

```ruby
2.0.0-p451 :001 > ENV['APIKEY']
 => "96ce1942872335547853a0bb3b0c24db"
2.0.0-p451 :002 >
```

Delete the hard-coded apikey and read it from the environment variable:

```ruby
class BeermappingApi
  # ...

  def self.key
    raise "APIKEY env variable not defined" if ENV['APIKEY'].nil?
    ENV['APIKEY']
  end
end
```

The code contains an exeption which is executed in case the apikey is not found.

The value of the environment variable will have to be defined if you search for beer restaurants. You can define the enviroment variable by starting your application as follows:

```ruby
➜  ratebeer git:(master) ✗ export APIKEY="96ce1942872335547853a0bb3b0c24db"
➜  ratebeer git:(master) ✗ rails s
```

or defining the environment variable together with the start command:

```ruby
➜  ratebeer git:(master) ✗ APIKEY="96ce1942872335547853a0bb3b0c24db" rails s
```

You can define the value of the environment variable (with the export command) in the file which is executed when the shell is started (the file format will be .zshrc, .bascrc or .profile according to the shell).

It's simple to set up the enviroment variable value in Heroku too, see
https://devcenter.heroku.com/articles/config-vars

## More about the controller

Some have been quite confused about the idea behind the <code>show</code> controller methods. You'll find more insights about it by going through the exercise below.

Take a look at a brewery controller. The controller method to show a singular brewery does not contain any code:

```ruby
  def show
  end
```
the view template app/views/breweries/show.html.erb renders by default anyway and it points to the <code>@brewery</code> variable:

```ruby
    <h2><%= @brewery.name %>
    </h2>

    <p>
      <em>Established year:</em>
      <%= @brewery.year %>
    </p>
```

how will the variable get a value? The value is set in the controller as a _before filter_ defined in the method <code>set_brewery</code>.

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]
  #...

  def set_brewery
    @brewery = Brewery.find(params[:id])
  end
end
```

so the controller defines that the code below should be executed always before the method <code>show</code>

```ruby
@brewery = Brewery.find(params[:id])
```

In turn, this loads the brewery object from memory and saves it into a variable for the view.

As you'll realize from the code, the controller retrieves the brewery ID through the has <code>params</code>. How does this happen?

If you look at the application routes either with the command <code>rake routes</code> or through the browser (going to whatever invalid address), you will see that the route information concerning singular breweries looks like this

```ruby
brewery_path	 GET	 /breweries/:id(.:format)	 breweries#show
```

so the form of the URL of a singular brewery is _breweries/42_, and the ending number stands for the brewery ID. As the route definition implies, the brewery ID is give as the value of the <code>:id</code> key in the <code>params</code> hash.

You could aslo define a 'parameter path' by hand. If you added the following chunk of code to routes.rb

```ruby
   get 'panimo/:id', to: 'breweries#show'
```

you would have access to the brewery page from the address http://localhost:3000/panimo/42. Once again, the address would make use of the method <code>show</code> which would retrieve the ID in the same way as the <code>params</code> hash.

Differently, if you wanted to use another controller method and if you defined the route in the following way

```ruby
   get 'panimo/:panimo_id', to: 'breweries#nayta'
```

the controller method could look like the one below:

```ruby
   def nayta
     @brewery = Brewery.find(params[:panimo_id])
     render :index
   end
```

so this time you could define the route so that the brewery ID was referenced through the <code>:brewery_id</code> key of the <code>params</code> hash.

## Ravintolan sivu

> ## Exercises 5 and 6 (this equals to two exercises)
>
> Improve your application so that you can open a page with restaurant information by clicking on restaurant names. The page should also include a map (implemented with iframe, for instance), showing the restaurant location. Notice that the map URL can be found from the restaurant information. Attention: using iframes is not the best option in terms of information security issues. A better option would be using [Googlen Map APIa](https://developers.google.com/maps/) straight away.
>* You'd better follow Rails conventions when you pick the restaurant URL, that is places/:id. Routes.rb will look something like this:
>
>```ruby
> resources :places, only:[:index, :show]
> # mikä generoi samat polut kuin seuraavat kaksi
> # get 'places', to:'places#index'
> # get 'places/:id', to:'places#show'
>
> post 'places', to:'places#search'
> ```
>
>* Attention: the restaurant information are retrieved from cache a bit inderectly when users go to a restaurant page. In order to have access to this information you will have to "remember" the city where the restaurant was found, in addition to its ID – or the result of the last search operation. One way to do this is through sessions, see https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko3.md#k%C3%A4ytt%C3%A4j%C3%A4-ja-sessio
>
> Another way to implement this functionality is the so-called "Locquery Service," as described at the page http://beermapping.com/api/reference/ 
>
> Check whether adding the restaurant page breaks any test. If so, you can try to fix the test, even though it is not essential at this point.

After the exercise, your application will not be much different that the picture:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-2.png)


## Rating beers straight from a beer page

Users have to rate beers from a separate page so far, and the beer is chosen out of a separate rating menu. It would be more natural if rating could happen straight from a beer page.

There are many optional ways to implement this. See below one of the easiest. You'll make use of the <code>form_for</code> helper, creating a form with the help of the object behind it. The **BeersController** show method will need a small change:

```ruby
  def show
    @rating = Rating.new
    @rating.beer = @beer
  end
```

So in case a beer is rated, it creates a rating object linked to it for the view template. The rating object is created with new, so it is not stored in the database. Notice that before executing the method <code>show</code>, a before filter executes a command which retrieves the beer to inspect from the database: <code>@beer = Beer.find(params[:id])</code>


The view template /views/beers/show.html.erb is modified as follows:

```erb
<h2> <%= @beer %> </h2>

<p>
  <strong>Style:</strong>
  <%= @beer.style %>
</p>

<% if @beer.ratings.empty? %>
  <p>beer has not yet been rated!</p>
<% else %>
  <p>has been rated <%= @beer.ratings.count %> times, average score <%= @beer.average_rating %></p>
<% end %>

<% if current_user %>

  <h4>give a rating:</h4>

  <%= form_for(@rating) do |f| %>
    <%= f.hidden_field :beer_id %>
    score: <%= f.number_field :score %>
    <%= f.submit %>
  <% end %>

  <%= link_to 'Edit', edit_beer_path(@beer) %>

<% end %>
```

If you want that the form sends the beer ID, the field <code>beer_id</code> will have to be added into the form. You don't want that users are able to manipulate the field however, so it should be defined as <code>hidden_field</code> in the form.

Because the form is created with the helper <code>form_for</code>, it will be sent to <code>ratings_path</code> automatically with an HTTP POST request, which means that it is the rating controller <code>create</code> method which takes care of sending the form. The controller will works with no changes needed!

There is a small issue in this solution. If users try to give an invalid rating:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-3.png)

the controller (that is, the rating controller <code>create</code> method) will render a new rating form instead of the beer view:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-4.png)

A possible solution would be checking what address the create method came from and then rendering the right page based on the address found. You don't need to implement this change now, however.

First fix a bigger issue. You will see from the picture above, that a beer can not be chosen anymore if the rating validation fails (in this case, we tried to rate the beer _Huvila Pale Ale_ and we can choose _iso 3_).

The reason behind this is that the method <code>options_from_collection_for_select</code> which generates the options of the drop-down menu is not told what options it should choose by default, and so it picks the first object of the collection. You can specify the default options giving the method its fourth parameter:

```erb
    options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id) %>
```

So modify the view template app/views/ratings/new.html.erb to look like below:

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

    <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id) %>
    score: <%= f.number_field :score %>

    <%= f.submit %>
<% end %>
```

In fact, other application forms have the same problem. Check what happens for instance when you edit beer information. Fix the form if you want.

> ## Exercise 7
>
> Make it possible to join beer clubs straight from beer club pages.
>
> You should stick to the same implementation principle as for the ratings on beer pages, so add a form in the page for beer clubs which should help you to create a new <code>Membership</code> object which belongs to the beer club and to the singed-in user. The form does not need anything else than a 'submit' button:
>
>```erb
>  <%= form_for(@membership) do |f| %>
>     <%= f.hidden_field :beer_club_id %>
>     <%= f.submit value:"join the club" %>
>  <% end %>
>```

And now some small fashonable changes when joining beer clubs

> ## Exercise 8
>
> The joining button should not be displayed if users are not signed in the system or if the user is already a member of the club.
>
> Change your code (the appropriate method of the membership controller) so that after joining a beer club the browser points to the beer club page and the page shows a message similar to the picture below.

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-5.png)

> ## Exercise 9
>
> Extend your application functionality so that members can leave a beer club.
>
> Add a button for the beer club page which allows to leave the club. The button should be visible only if the sign-in user goes to a beer page where he is a member already. When they press the button, users memberships expires and they are redirected back to their own page. The page should show a message reporting the successful action, as the pictures below explain better.
>
> Hint: this functionality can be implemented in the same way as for joining a club, that is with a form on the beer club page. The HTTP method used in the form should be defined as "delete":
>
>```erb
>    <%= form_for(@membership, method: "delete") do |f| %>
>       <%= f.hidden_field :beer_club_id %>
>       <%= f.submit value: "end the membership" %>
>    <% end %>
>```
>
> In order to use the form, the value of the <code>@membership</code> variable in the controller should be the object for the user club.

If the user is the club member, the page should show a button to leave the club:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-5a.png)

After leaving a club, the user should be redirected to their page and a message should appear:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-5b.png)


## Migrations

You have been using Rails migrations already from the first week. Now it's time to dig deeper into the topic.

> ## Exercise 10
>
> Read more at http://guides.rubyonrails.org/migrations.html

## Beer style

> ## Exercises 11 – 13 (this equals three exercises)
>
> Expand your application so that the beer style won't be a string any more; instead, the styles should be saved in the database. Each beer style should also have a textual description. The style description type should be defined as <code>text</code>; in fact, the default size of a field defined with the type <code>string</code> is only 255 characters.
>
> Afte the change, the beer-style relationship should be like this:

![picture](http://yuml.me/30b291af)

> Notice, that the <code>style</code> attribute which currently belongs to beer should be deleted so that there will be no association conflict between the accessor to be generated and the old field.
>
> It might be a bit challenging to make the change so that beers are linked automatically to the right style database tables.
> This will also work if you implement the change with various steps, for instance:
> * create an database table for the styles
> * create a table line for each style name which is found in the _beers_table (this will work by end from the console)
> * rename the _beers_ table style column, called it something like _old_style_ (do it with migrations)
> * work by hand on your console and connect beers and the _style_ objects with the help of the old_style column
> * remove _old_style_ from the beer table with the help of migrations
>
> **Notice that you should update Heroku instance simultaneously!**
>
> It will be even faster if you do all the steps above within only one migration.
>
> Suggestion: you can train migrating data before you do it. Copy the database, that is the file _db/development.sqlute3_, and if you mess up with the migration, you can always recover the old data. Byebug might also turn up useful when you make the migration.
>
> You can also move to the new styles in the database more directly deleting the _style_ column from the beers and setting up the beer styles from the console, for instance.
>
> After the change has been implemented, when beers are created, their style will be chosen from a ready-made list as it is for breweries. Also add a link to the navigation bar to the style page.
>
> The style page should be added a list with all the beers with that style.

The beer style page will look something like this, after you have completed the exercise:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w5-6.png)

**ATTENTION:** make sure that it is still possible to create beers after the extention! You will have to change a few things, and maybe the most difficult to notice is the beer controller help method <code>beer_params</code>.

A good beer style list and their pictures is found at the address http://beeradvocate.com/beer/style/

> ## Exercise 14
>
> Saving the styles in the database will break most of the tests. Update your tests. Notice that you will have to fix also the FactoryGirl factories.
>
> Even though the broken tests are many, keep calm. Solve the problems one test after the other, the same problems are accomulated in various different places and updating the tests will not be too difficult at the end.

## Tehtävien palautus

Commit all your changes and push the code to Github. Deploy to the newest version of Heroku, too.

You should mark that you have returned the exercises at http://wadrorstats2015.herokuapp.com/
