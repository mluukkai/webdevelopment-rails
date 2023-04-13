You will continue to develop your application from the point you arrived at the end of week 6. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week from the submission system. 

<a id="user-content-muistutus-debuggerista" class="anchor" href="#muistutus-debuggerista" aria-hidden="true"><span class="octicon octicon-link"></span></a>Muistutus debuggerista

## Reminder on debugger
You ran into  [debugger](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#debugger) on week 2 and also got an reminder [last week](https://github.com/mluukkai/webdevelopment-rails/blob/main/week6.md#a-reminder-on-debugger). 

Once more: **When you have problems, instead of guessing, use the debugger!**

Throughout this course, the importance of using the Rails console as a development tool has been emphasized. So **when you are doing something even slightly untrivial, first test it in the console.** In some cases it might be even better to do the testing in the console launched by the debugger as then you can work in exactly the context you are writing the code for. This way you can access eg. variables <code>params</code>, <code>sessions</code> and other execution context dependent data.


## About tests

Part of this week's exercises may break some of the tests from the previous weeks. You can mark the exercises done despite breaking the tests. Fixing them and GitHub Actions is optional.

## Different orders

You want to implement now a new functionality to sort your beer list against the different columns. Forward the information about the order you want to the controller as parameter of an HTTP request. Change the table in <code>app/views/beers/index.html.erb</code> like this:

```erb
<table class="table table-striped  table-hover">
  <thead>
    <tr>
      <th><%= link_to "Name", beers_path(order: "name")%></th>
      <th><%= link_to "Style", beers_path(order: "style")%></th>
      <th><%= link_to "Brewery", beers_path(order: "brewery")%></th>
      <th><%= link_to "Rating", beers_path(order: "rating")%></th>
    </tr>
  </thead>
  ...
</table>
```

so the table titles have now become links which point back to the same page, but in addition, they add the [query parameter](https://en.wikipedia.org/wiki/Query_string) <code>:order</code> to the request, defining the new order.
What happens is that the parameter is passed along the url, attached to the end of it, separated by a question mark. For example if you click the style column, the url becomes _beers?order=style_

The controller can avćcess the parameter through the  <code>params</code> hash. As expected, the value of the parameter defining the order is <code>params[:order]</code>.

Let's extend the beer controller so that it tests whether the request has a parameter, and if so, the beers are sorted in the right order:

```ruby
def index
  @beers = Beer.all

  order = params[:order] || 'name'

  @beers = case order
           when "name" then @beers.sort_by(&:name)
           when "brewery" then @beers.sort_by { |b| b.brewery.name }
           when "style" then @beers.sort_by { |b| b.style.name }
           when "rating" then @beers.sort_by(&:average_rating).reverse
           end
end
```

The code defines that the table should be sorted against the names by default. It will happen like this

```ruby
order = params[:order] || 'name'
```


Normally <code>order</code> will get the value <code>params[:order]</code>. If the parameter <code>:order</code> hasn't been given, (so it's value is <code>nil</code>) the part after <code>||</code> will be picked as value, that is:  _name_.

**Attention 1:** Ruby's command <code>case when</code> is used to sort the beers

```ruby
@beers = case order
  when 'name' then @beers.sort_by{ |b| b.name }
  when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
  when 'style' then @beers.sort_by{ |b| b.style.name }
end
```

and it works by default in the same way as the code below

```ruby
  @beers =
  if order == 'name'
    @beers.sort_by{ |b| b.name }
  elsif orded == 'brewery'
    @beers.sort_by{ |b| b.brewery.name }
  elsif orded == 'style'
    @beers.sort_by{ |b| b.style.name }
  end
```

**Attention 2:** in the example the beers are first retrived from the database and then they are sorted in central memory. The beer list could also be sorted at database level:

```ruby
# oluet nimen perusteella järjestettynä
Beer.order(:name)

# oluet panimoiden nimien perusteella järjestettynä
Beer.includes(:brewery).order("breweries.name")

# oluet tyylin nimien perusteella järjestettynä
Beer.includes(:style).order("style.name")
```

<blockquote>

## Exercise 1

Change the list page so that beer clubs can be sorted in alphabetic order against their names or or cities or by their foundation year. The name order is the default one.

_ATTENTION_ if you haven't implemented beer clubs in your application, you can do this exercise for breweries page (and assume that both active and inactive breweries are sorted the same way)

</blockquote>


## Functionality implemented on the browser

Your solutions to sort the beers list are quite good. The problem comes with performance, because after sorting it always makes a call to the server, which generates a page to show in the new order.

The sorting functionality could also be implemented with javascript on the browser. Even though this course is about server functionality, you will now see an example of how the sorting functionality could be implemented on the browser. In this solution the browser only provides a list of the beers in json form, and the browser executes the javascript code and takes care of creating the table listing the beers.

You won't overwrite the already existing beer list, the functionality of the beers page. Instead, you will create a completely new page at the address beerlist with that functionality. Make a route for the page in the file routes.rb:

    get 'beerlist', to: 'beers#list'

So use the <code>list</code> method that is contained in the beer controller. The method doesn't need to do anything:

```ruby
class BeersController < ApplicationController
  before_action :ensure_that_signed_in, except: [:index, :show, :list]
  # muut before_actionit ennallaan

  def list
  end

  ...
end
```

**Attention** The method `list` was added to those methods that don't require executing the <code>ensure_that_signed_in</code> method before it, meaning that viewing the beer list generated with javascript doesn't require logging in! 

Also the view views/beers/list.html.erb is minimalist:

```erb
<h2>Beers</h2>

<div id="beers"></div>
```

SO the view only places a div element on the page, and giving it "beer" as ID (that is a reference to access the element).

As expected, nothing else than an h2 element will be seen at http://localhost:3000/beerlist.

Start now to write the action logic implementation with Javascript.

The Javascript code that your Rails application needs should be placed in the folder app/javascript/custom. Create the file _utils.js_ in the folder:

```javascript
const hello = () => {
  document.getElementById("beers").innerText = "Hello from JavaScript";
  console.log("hello console!");
}

export { hello };
```

In addition, you need to implement the _hello_ function into your application. Do it by adding into file app/javascript/application.js the following lines:

```javascript
import { hello } from "custom/utils";

hello();
```

Also take javascript (located in the _custom_ folder) into use in the application's importmap, meaning, add to <code>config/importmap.rb</code> file:

```rb
pin_all_from "app/javascript/custom", under: "custom"
```

If you open the page again again now, Javascript will first search for the element with the id  <code>beers</code>, after which it   will set up the text "hello from javascript" as its text. The next command writes the greeting to the Javascript console.

The console is an <strong>extremely important</strong> tool as far as Javascript programming in the browser is concerned. You can open the console in Chrome from the tools tap or pressing ctrl,shift,j (in linux) or alt,cmd,i (in mac):

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w7-1.png)

<strong>You want to keep the console open all the time when you program with Javascript!</strong>

Javascript looks cryptic at first, because of its various anonymous functions. The code in file application.js  defines that when the page is loaded the _hello_ function in utils.js is executed.

If you check the address http://localhost:3000/beers.json with your browser, you will see the beer information in a textual json form (see http://en.wikipedia.org/wiki/JSON, http://www.json.org):

```ruby
[{"id":10,"name": "Extra Light Triple Brewed","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:47:54.117Z","updated_at": "2022-09-05T10:17:39.414Z","url": "http://localhost:3000/beers/10.json"},{"id":6,"name": "Hefeweizen","style":{"id":4,"name": "German hefeweizen","description": "A south German style of wheat beer (weissbier) typically made with a ratio of 50 percent barley to 50 percent wheat. Sometimes the percentage of wheat is even higher. \"Hefe\" means \"with yeast,\" hence the beer's unfiltered and cloudy appearance. The particular ale yeast used produces unique esters and phenols of banana and cloves with an often dry and tart edge, some spiciness, and notes of bubblegum or apples. Hefeweizens are typified by little hop bitterness, and a moderate level of alcohol. Often served with a lemon wedge (popularized by Americans), to cut the wheat or yeasty edge, some may find this to be either a flavorful snap or an insult that can damage the beer's taste and head retention.","created_at": "2022-09-05T10:17:39.361Z","updated_at": "2022-09-05T10:36:17.788Z"},"brewery_id":3,"created_at": "2018-09-01T16:41:53.522Z","updated_at": "2022-09-05T10:17:39.406Z","url": "http://localhost:3000/beers/6.json"},{"id":7,"name": "Helles","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":3,"created_at": "2018-09-01T16:41:53.525Z","updated_at": "2022-09-05T10:17:39.408Z","url": "http://localhost:3000/beers/7.json"},{"id":16,"name": "Helles","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":3,"created_at": "2018-09-08T10:56:52.592Z","updated_at": "2022-09-05T10:17:39.420Z","url": "http://localhost:3000/beers/16.json"},{"id":4,"name": "Huvila Pale Ale","style":{"id":2,"name": "American Pale Ale","description": "Originally British in origin, this style is now popular worldwide and the use of local or imported ingredients produces variances in character from region to region. American versions tend to be cleaner and hoppier (with the piney, citrusy Cascade variety appearing frequently) than British versions, which are usually more malty, buttery, aromatic, and balanced. Pale Ales range in color from deep gold to medium amber. Fruity esters and diacetyl can vary from none to moderate, and hop aroma can range from lightly floral to bold and pungent. In general, expect a good balance of caramel malt and expressive hops with a medium body and a mildly bitter finish. ","created_at": "2022-09-05T10:17:39.359Z","updated_at": "2018-09-22T12:07:42.742Z"},"brewery_id":2,"created_at": "2018-09-01T16:41:53.516Z","updated_at": "2022-09-05T10:17:39.396Z","url": "http://localhost:3000/beers/4.json"},{"id":9,"name": "IVB","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:46:01.643Z","updated_at": "2022-09-05T10:17:39.412Z","url": "http://localhost:3000/beers/9.json"},{"id":1,"name": "Iso 3","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:41:53.508Z","updated_at": "2022-09-05T10:17:39.384Z","url": "http://localhost:3000/beers/1.json"},{"id":2,"name": "Karhu","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:41:53.511Z","updated_at": "2022-09-05T10:17:39.389Z","url": "http://localhost:3000/beers/2.json"},{"id":8,"name": "Lite","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:45:09.037Z","updated_at": "2022-09-05T10:17:39.410Z","url": "http://localhost:3000/beers/8.json"},{"id":14,"name": "Nanny State","style":{"id":6,"name": "Low alcohol beer","description": "Low Alcohol Beer is also commonly known as Non Alcohol (NA) beer, despite containing small amounts of alcohol. Low Alcohol Beers are generally subjected to one of two things: a controlled brewing process that results in a low alcohol content, or the alcohol is removed using a reverse-osmosis method which passes alcohol through a permeable membrane. They tend to be very light on aroma, body, and flavor.","created_at": "2022-09-05T10:17:39.362Z","updated_at": "2018-09-22T12:11:57.808Z"},"brewery_id":5,"created_at": "2018-09-06T14:30:50.585Z","updated_at": "2022-09-05T10:17:39.418Z","url": "http://localhost:3000/beers/14.json"},{"id":23,"name": "Panimomestarin IPA","style":{"id":5,"name": "American IPA","description": "Today's American IPA is a different soul from the IPA style first reincarnated in the 1980s. More flavorful and aromatic than the withering English IPA, its color can range from very pale golden to reddish amber. Hops are the star here, and those used in the style tend to be American with an emphasis on herbal, piney, and/or fruity (especially citrusy) varieties. Southern Hemisphere and experimental hops do appear with some frequency though, as brewers seek to distinguish their flagship IPA from a sea of competitors. Bitterness levels vary, but typically run moderate to high. Medium bodied with a clean, bready, and balancing malt backbone, the American IPA has become a dominant force in the marketplace, influencing brewers and beer cultures worldwide.","created_at": "2022-09-05T10:17:39.361Z","updated_at": "2018-09-22T12:09:23.686Z"},"brewery_id":1,"created_at": "2018-09-22T10:33:04.353Z","updated_at": "2018-09-22T10:33:04.353Z","url": "http://localhost:3000/beers/23.json"},{"id":13,"name": "Punk IPA","style":{"id":5,"name": "American IPA","description": "Today's American IPA is a different soul from the IPA style first reincarnated in the 1980s. More flavorful and aromatic than the withering English IPA, its color can range from very pale golden to reddish amber. Hops are the star here, and those used in the style tend to be American with an emphasis on herbal, piney, and/or fruity (especially citrusy) varieties. Southern Hemisphere and experimental hops do appear with some frequency though, as brewers seek to distinguish their flagship IPA from a sea of competitors. Bitterness levels vary, but typically run moderate to high. Medium bodied with a clean, bready, and balancing malt backbone, the American IPA has become a dominant force in the marketplace, influencing brewers and beer cultures worldwide.","created_at": "2022-09-05T10:17:39.361Z","updated_at": "2018-09-22T12:09:23.686Z"},"brewery_id":5,"created_at": "2018-09-06T14:30:33.589Z","updated_at": "2022-09-05T10:17:39.416Z","url": "http://localhost:3000/beers/13.json"},{"id":22,"name": "Sink the Bismarck","style":{"id":3,"name": "Baltic Porter","description": "Porters of the late 1700's were quite strong compared to today's standards, easily surpassing 7 percent alcohol by volume. Some English brewers made a stronger, more robust version, to be shipped across the North Sea that they dubbed a Baltic Porter. In general, the style's dark brown color covered up cloudiness and the smoky, roasted brown malts and bitter tastes masked brewing imperfections. Historically, the addition of stale ale also lent a pleasant acidic flavor to the style, which made it quite popular. These issues were quite important given that most breweries at the time were getting away from pub brewing and opening up production facilities that could ship beer across the world.","created_at": "2022-09-05T10:17:39.360Z","updated_at": "2018-09-22T12:08:13.953Z"},"brewery_id":5,"created_at": "2018-09-22T10:09:59.120Z","updated_at": "2018-09-22T10:09:59.120Z","url": "http://localhost:3000/beers/22.json"},{"id":21,"name": "Trans European Lager","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2022-09-05T10:42:19.312Z","updated_at": "2022-09-05T10:42:19.312Z","url": "http://localhost:3000/beers/21.json"},{"id":3,"name": "Tuplahumala","style":{"id":1,"name": "European pale lager","description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at": "2022-09-05T10:17:39.358Z","updated_at": "2022-09-05T10:35:04.921Z"},"brewery_id":1,"created_at": "2018-09-01T16:41:53.513Z","updated_at": "2022-09-05T10:17:39.392Z","url": "http://localhost:3000/beers/3.json"},{"id":5,"name": "X Porter","style":{"id":3,"name": "Baltic Porter","description": "Porters of the late 1700's were quite strong compared to today's standards, easily surpassing 7 percent alcohol by volume. Some English brewers made a stronger, more robust version, to be shipped across the North Sea that they dubbed a Baltic Porter. In general, the style's dark brown color covered up cloudiness and the smoky, roasted brown malts and bitter tastes masked brewing imperfections. Historically, the addition of stale ale also lent a pleasant acidic flavor to the style, which made it quite popular. These issues were quite important given that most breweries at the time were getting away from pub brewing and opening up production facilities that could ship beer across the world.","created_at": "2022-09-05T10:17:39.360Z","updated_at": "2018-09-22T12:08:13.953Z"},"brewery_id":2,"created_at": "2018-09-01T16:41:53.519Z","updated_at": "2022-09-05T10:17:39.400Z","url": "http://localhost:3000/beers/5.json"}]
```

You can improve the readability of a Json page by copying the page contents into the [jsonlint](http://jsonlint.com/) service:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w7-3.png)

A better option is installing a plugin on the browser, which can be understand json. A good choice is Chrome's [jsonview](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc), the plugin shapes json nicely in the browser:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w7-4.png)

A closer look shows that each singular json beer reminds of Ruby's hash:

```ruby
{
  "id":10,"name": "Extra Light Triple Brewed",
  "style":{
    "id":1,"name": "European pale lager",
    "description": "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.",
    "created_at": "2022-09-05T10:17:39.358Z",
    "updated_at": "2022-09-05T10:35:04.921Z"
  },
  "brewery_id":1,
  "created_at": "2018-09-01T16:47:54.117Z",
  "updated_at": "2022-09-05T10:17:39.414Z","url": "http://localhost:3000/beers/10.json"}
```

How can Rails return the results in json instead of HTML when needed?

Try to get the list of all raitings in json, so try out the address http://localhost:3000/ratings.json

You'll get an error message:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w7-4b.png)

So it's not that jsons are created completely automatically, you created the whole code for the rating by hand, and as you saw in the error message, there was no suitable template for the format 'json'.

You'll see that you can find a set of templates ending in _json.jbuilder in the views folder of the resources created with scaffold, like the beer views folder. As you may have guessed, Rails makes use of them if you want the resource in json form.

Take example from the template app/views/beers/index.json.jbuilder and make the following json.jbuilder template for the ratings (the file is app/views/ratings/index.json.jbuilder):

```ruby
json.array! @ratings, partial: "ratings/rating", as: :rating
```

Additionally, you need a _partial_ file for the ratings. Again, take beer template app/views/beers/\_beer.json.jbuilder as an example and create file app/views/ratings/\_rating.json.jbuilder

```ruby
json.extract! rating, :id, :score
json.url rating_url(rating, format: :json)
```


and now you can get the ratings in json at the address http://localhost:3000/ratings.json

```ruby
[{"id":31,"score":34},{"id":30,"score":42},{"id":27,"score":40},{"id":25,"score":12},{"id":24,"score":10}]
```

_Attention_ The variable <code>@ratings</code> is used in the jbuilder template and so must be defined in the `index` controller method. After last week's refactoring it is not defined anymore.

In the json.jbuilder template, you could easily define that the ratings json should also show the beer information concerning the rating:

```ruby
json.extract! rating, :id, :score, :beer
json.url rating_url(rating, format: :json)
```


More about jbuilderista at https://github.com/rails/jbuilder.

In addition to Json's jbuilder template, another way to return data in json form would be to use a <code>respond_to</code> command, which is used by some methods generated with scaffolds. In such case, there would be no need for the json jbuilder template, and the controller would look like below

```ruby
def index
  @ratings = Rating.all

  respond_to do |format|
    format.html { } # renderöidään oletusarvoinen template
    format.json { render json: @ratings }
  end
end
```

Using Jbuilder templates is definitely a better choice, in this case, creating the json "view" – that is the resource representation – is done completely separately from the controller. It is not among the controller's tasks to form the response outlook, whether it was a json or an HTML response.

Go back now to the beers page. When you create the page in javascript, the idea is to fetch the beers in json form from the server in fact, and to render them as you want with the help of javascript.

Change your javascript code as below:

```javascript
const handleResponse = (data) => {
  document.getElementById("beers").innerText = `oluita löytyi ${data.length}`;
};

const beers = () => {
  fetch("beers.json")
    .then((response) => response.json())
    .then(handleResponse);
};

export { beers };
```

The hello fucntion is renamed as beers (remmebr to change the name also in export and application.js import!). 

The beers function uses the [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) method available for the browser to fetch the json form beers from the address beers.json. The data returned by fetch is accessible by calling the method twice. The first call causes that the beers are parsed separately into json format from the data returned to the browser. The second call asks the function _handleResponse_ to handle the data. The handleResponse adds the number of beers to the page. You can combine text and variables in javascript, like you can in Ruby, except that in javascript you use the dollar symbol and ` instead of normal apostrophies.

Behind the slightly odd looking syntax _[then](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)_ is the fact that the function _fetch_ returns a so called [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) and the actual returned data must be taken from the promise with the _then_ function.

So after getting the beers from the server, the function should generate the HTML code for listing them and add it to the page.

Change the javascript code now to list only the beer names at first:

```javascript
const handleResponse = (beers) => {
  const beerList = beers.map((beer) => `<li>${beer.name}</li>`);

  document.getElementById("beers").innerHTML = `<ul> ${beerList.join("")} </ul>`;
};
```

The code defines the local table variable <code>beerList</code> and goes through the list <code>beers</code> that it received as parameter. By using the _map_ function, you can create a new table directly from the return value of the function.  For each beer a HTML element is returned into `beerList`. The element is like this:

```erb
<li>Extra Light Triple Brewed</li>
```

At the end, ul tags are added to the beginning and the end of the list, and the list elements are joined with the join method. The HTML code generated from this is added to the element with the <code>beers</code> ID.

In this way you have got a simple list of the names of the beers on the page.

What if you wanted to sort the beers? To make this happen, refactor first the code as follows:

```javascript
const BEERS = {};

const handleResponse = (beers) => {
  BEERS.list = beers;
  BEERS.show();
};

BEERS.show = () => {
  const beerList = BEERS.list.map((beer) => `<li>${beer.name}</li>`);

  document.getElementById("beers").innerHTML = `<ul> ${beerList.join("")} </ul>`;
};

```

You defined now the object <code>BEERS</code>, that receives the beer list from the server in the attribute <code>BEERS.list</code>. The method <code>BEERS.show</code> creates an HTML list of <code>BEERS.list</code> objects and places them in the view.

In this way, the beer list from the server remains "in memory" in the browser variable <code>BEERS.list</code> and the list can be resorted when needed, and it can be shown to users in new orders without that Web page needing to communicate with the server.

Add a clickable text to the page, allowing to sort the beers on the page in reversed order:

```erb
<p id="reverse">reverse!</p>
<div id="beers"></div>
```

Then add a click handler to the link in javascript, to sort the beers in descending order when the link is clicked and to show them in the beers elements in the page:

```javascript
BEERS.reverse = () => {
  BEERS.list.reverse();
};

const beers = () => {
  document.getElementById("reverse").addEventListener("click", (e) => {
    e.preventDefault();
    BEERS.reverse();
    BEERS.show();
  });

  fetch("beers.json")
    .then((response) => response.json())
    .then(handleResponse);
};

export { beers };
```
The click handler of the text is defined in the page's _beers_ function. So when the document has been loaded, the click handler is _registered_ to the element with the id "reverse".

When the link is clicked the event handler first calls the method <code>e.preventDefault</code>. This method prevents the "normal" function, that is, accessing a (now inexistent) link.
After that, methods _reverse_ and _show_ are called to render the beers on the screen in reverse order.

You'll now understand the basics well enough to implement the real functionality.

Change the view as follows:

```erb
<h2>Beers</h2>

<table id="beertable" class="table table-hover">
  <thead>
    <tr>
      <th> <span id="name">Name</span> </th>
      <th> <span id="style">Style</span> </th>
      <th> <span id="brewery">Brewery</span> </th>
    </tr>
  <thead>
  <tbody>
    <div id="beerlist"></div>
  </tbody>
</table>
```

So the three column names have been made into elements that click listeners will be registered to. The table was given the ID <code>beertable</code>.

Change the <code>show</code> method defined in the javascript so that it adds the beer names to the table:

```javascript
const createTableRow = (beer) => {
  const tr = document.createElement("tr");
  const beername = tr.appendChild(document.createElement("td"));
  beername.innerHTML = beer.name;

  return tr;
};

BEERS.show = () => {
  const table = document.getElementById("beertable");

  BEERS.list.forEach((beer) => {
    const tr = createTableRow(beer);
    table.appendChild(tr);
  });
};
```

So first the code sets a reference to the table in the variable <code>table</code>. After this, with the help of the createTableRow helper function, some <code>tr</code> elements are created. The cells of the table, <code>td</code>, are placed inside these elements. The row is returned to the forEach loop where it is set as the table's "child" by using the appendChild method.

Expand the method to show all information of the beer. You'll notice however that in the json-format beers list at <http://localhost:3000/beers.json> the only information about a beer's brewery is the brewery object's id. You would like to see the brewery's name. The beer style's information is already completely available in the json.

This is luckily easy to fix by editing the json-jbuildertemplate that generates the beer list.
The template now looks like this:

```ruby
json.array! @beers, partial: 'beers/beer', as: :beer
```

The template defines that for each beer a json-format depiction is created with the help of _\_beer.json.jbuilder_. The file contents are:

```ruby
json.extract! beer, :id, :name, :style, :brewery_id, :created_at, :updated_at
json.url beer_url(beer, format: :json)
```

The file defines that the fields <em>id</em>, <em>name</em> and <em>brewery_id</em> as well as <em>style</em> should be contained in the json form for each beer; also, style refers to the <code>Style</code> object of the beer. The style object has to be rendered completely in the json form of the beer. You will also get the brewery json form in the same beer json listing if you replace <em>brewery_id</em> with <em>brewery</em>, in the template. So change the template taking care of rendering a single beer json:

```ruby
json.extract! beer, :id, :name, :style, :brewery
```

the last line was deleted, which would have added an URL to the beer's own json form. The timestamp fields were also removed.

Now the table can be generated after adding the next lines into the createTableRow function:

```javascript
const createTableRow = (beer) => {
  const tr = document.createElement("tr");
  tr.classList.add("tablerow");
  const beername = tr.appendChild(document.createElement("td"));
  beername.innerHTML = beer.name;
  const style = tr.appendChild(document.createElement("td"));
  style.innerHTML = beer.style.name;
  const brewery = tr.appendChild(document.createElement("td"));
  brewery.innerHTML = beer.brewery.name;

  return tr;
};
```

The beers list in json form will contain also a lot of useless information, because at the same time, the brewery json forms of each beer brewery and style are rendered completely. You could optimize the the single beer json template so that the beer brewery and style would follow the json form only as far as their name is concerned:

```ruby
json.extract! beer, :id, :name
json.style do
  json.name beer.style.name
end
json.brewery do
  json.name beer.brewery.name
end
```

Now the json-form list sent by the server is much more compact:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w7-5.png)

The links have to be set up now with the event listeners that execute the sorting (you find the final javascript code below):

```javascript
const BEERS = {};

const handleResponse = (beers) => {
  BEERS.list = beers;
  BEERS.show();
};

const createTableRow = (beer) => {
  const tr = document.createElement("tr");
  tr.classList.add("tablerow");
  const beername = tr.appendChild(document.createElement("td"));
  beername.innerHTML = beer.name;
  const style = tr.appendChild(document.createElement("td"));
  style.innerHTML = beer.style.name;
  const brewery = tr.appendChild(document.createElement("td"));
  brewery.innerHTML = beer.brewery.name;

  return tr;
};

BEERS.show = () => {
  document.querySelectorAll(".tablerow").forEach((el) => el.remove());
  const table = document.getElementById("beertable");

  BEERS.list.forEach((beer) => {
    const tr = createTableRow(beer);
    table.appendChild(tr);
  });
};

BEERS.sortByName = () => {
  BEERS.list.sort((a, b) => {
    return a.name.toUpperCase().localeCompare(b.name.toUpperCase());
  });
};

BEERS.sortByStyle = () => {
  BEERS.list.sort((a, b) => {
    return a.style.name.toUpperCase().localeCompare(b.style.name.toUpperCase());
  });
};

BEERS.sortByBrewery = () => {
  BEERS.list.sort((a, b) => {
    return a.brewery.name
      .toUpperCase()
      .localeCompare(b.brewery.name.toUpperCase());
  });
};

const beers = () => {
  document.getElementById("name").addEventListener("click", (e) => {
    e.preventDefault;
    BEERS.sortByName();
    BEERS.show();
  });

  document.getElementById("style").addEventListener("click", (e) => {
    e.preventDefault;
    BEERS.sortByStyle();
    BEERS.show();
  });

  document.getElementById("brewery").addEventListener("click", (e) => {
    e.preventDefault;
    BEERS.sortByBrewery();
    BEERS.show();
  });

  fetch("beers.json")
    .then((response) => response.json())
    .then(handleResponse);
};

export { beers };
```

While calling the event listeners, the newly sorted elements of the BEERS.list will be appended to the table after the already existing ones. This was fixed by adding a line to the start of the BEERS.show function. In this line the pre-existing rows of the class <code>tablerow</code> are retrieved and deleted.

Your Javascript code is linked to each application page. The unfortunate result is that on any site you visit, the Javascript will execute the `beers` function. The application also tries to register the event listeners to every page even though it makes sense to do so only on the page with the beer list.

 Refine your Javascript code so that the `beers` function code is executed only if you are on a page with the table <code>beertable</code>:

```javascript
const beers = () => {
  if (document.querySelectorAll("#beertable").length < 1) return;

  //...

  var request = new XMLHttpRequest();

  request.onload = handleResponse;

  request.open("get", "beers.json", true);
  request.send();
};
```

If the page doesn't contain a element with the id _beertable_, the function execution won't continue. While developing, remember that the id should be unique, no application should contain two identical ids!

The current trend tells us we should move more and more of a Web pages functionality to the browser. The advantage is that you'll make your Web applications remind more and more of desktop applications.


## React

The code of the page you implemented with Javascript to list the beers was decent structure-wise. However, if compared to Rails fluency and effortless coding style, what you wrote was quite heavy and full of annoying and rutine-like details, at times. If the amount of browser executable code keeps growing, it is easy to end up with a messy code base which is hard to read and even harder to expand.

Javascript frontend development frameworks come to the rescue. For a long while, the most popular solution for frontend development has been [React](https://facebook.github.io/react/) which is developed by Facebook. React is a vast subject, in which you can immerse yourself on the Full Stack Web Development course offered by the the department. It is ongoing as a [open university course](https://fullstackopen.github.io/).

>## Exercise 2
>
>Follow the examples above, and use Javascript to implement the page listing all breweries http:localhost:3000/brewerylist
> 
> The page displays the name and founding year of the brewery, number of beers made by the brewery and whether the brewery is active or not. The page <strong>does not</strong> need to separate the expired breweries in their own table.
>
> Sorting of the breweries will be done in the next exercise.
>
>Remember to keep the Javascript console open the whole time while you proceed with the exercise! You can debug by printing to the Javascript console with the command <code>console.log()</code>
>
><strong>ATTENTION:</strong> due to the changes you did last week, the breweries json list http://localhost:3000/breweries.json does not work, because the breweries#index controller is not given the list of all breweries in the variable <code>@breweries</code> any more. Fix the situation.
>
> **ATTENTION2:** Do this exercise little by little, as was done with the beers list in the previous example. Debugging Javascript might be challenging and the **surest way to get overwhelmingly frustrated is to try to do this exercise quickly by copy-pasting the beer list code.**

> ## Exercise 3
>
> Expand the brewery list so that they can be ordered alphabetically by name, by founding year, or by the amount beers made by the brewery.

## Testing browser-side functionality

Make some tests with rspec/capybara for the Javascript beers list. Your starting point is the following file, spec/features/beerlist_page_spec.rb:

```ruby
require 'rails_helper'

describe "Beerlist page" do
  before :all do
    Capybara.register_driver :selenium do |app|
      Capybara::Selenium::Driver.new(app, :browser => :chrome)
    end
  end

  before :each do
    @brewery1 = FactoryBot.create(:brewery, name: "Koff")
    @brewery2 = FactoryBot.create(:brewery, name: "Schlenkerla")
    @brewery3 = FactoryBot.create(:brewery, name: "Ayinger")
    @style1 = Style.create name: "Lager"
    @style2 = Style.create name: "Rauchbier"
    @style3 = Style.create name: "Weizen"
    @beer1 = FactoryBot.create(:beer, name: "Nikolai", brewery: @brewery1, style:@style1)
    @beer2 = FactoryBot.create(:beer, name: "Fastenbier", brewery:@brewery2, style:@style2)
    @beer3 = FactoryBot.create(:beer, name: "Lechte Weisse", brewery:@brewery3, style:@style3)
  end

  it "shows one known beer" do
    visit beerlist_path
    expect(page).to have_content "Nikolai"
  end
end
```

Execute the test with the command <code>rspec spec/features/beerlist_page_spec.rb</code>. You will receive an error message:

```ruby
  1) Beerlist page shows one known beer
     Failure/Error: expect(page).to have_content "Nikolai"
       expected to find text "Nikolai" in "breweries beers styles ratings users clubs places signin signup\nyou should be signed in\nSign in\nusername password"
     # ./spec/features/beerlist_page_spec.rb:18:in `block (2 levels) in <top (required)>'

Finished in 21.17 seconds (files took 5.33 seconds to load)
1 example, 1 failure
```

It looks like that the page does not contain any beers list at all. Check this out with the command <code>save_and_open_page</code> that you should put right before the  <code>expect</code> command. This will open the browser page where capybara has navigated to
(see https://github.com/mluukkai/webdevelopment-rails/blob/main/week4.md#capybara).

And the beer table to show on the page is empty as expected:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-2.png)

You find the reason for this from Capybara documentation https://github.com/jnicklas/capybara#drivers.

> By default, Capybara uses the :rack_test driver, which is fast but limited: it does not support JavaScript, nor is it able to access HTTP resources outside of your Rack application, such as remote APIs and OAuth services. To get around these limitations, you can set up a different default driver for your features.

Fixing this is simple, too. The Javascript tests only need to be added a parameter and they will be executed with the help of Selenium, a test driver which knows Javascript:

```ruby
it "shows the known beers", js:true do
```

Run the tests. You'll run in an error message again:

```ruby
1) Beerlist page shows one known beer
    Failure/Error: visit beerlist_path

    WebMock::NetConnectNotAllowedError:
      Real HTTP connections are disabled. Unregistered request: GET http://127.0.0.1:52187/__identify__ with headers {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=>'Ruby'}

      You can stub this request with the following snippet:

      stub_request(:get, "http://127.0.0.1:52187/__identify__").
        with(
          headers: {
        'Accept'=>'*/*',
        'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3',
        'User-Agent'=>'Ruby'
          }).
        to_return(status: 200, body: "", headers: {})

      ============================================================
```


The reason is that you started to use the WebMock gem in [week 5](https://github.com/mluukkai/webdevelopment-rails/blob/main/week5.md#testing-beer-restaurant-search), blocking the test code HTTP connections by default. The Javascript beer list tries to fetch the beers list in json form from the server, in fact. You get over this if you allow the connections, for instance by editing the <code>before :all</code> code chunk (which initializes the tests):

```ruby
before :all do
  Capybara.register_driver :selenium do |app|
    Capybara::Selenium::Driver.new(app, :browser => :chrome)
  end
  WebMock.allow_net_connect!
end
```

The tests works finally.

When you create page contents with Javascript, these contents do not appear on the page together with the HTML base, but only later on, when the execution of the Javascript return call function. So if you look at the page contents right after navigating to the page, the Javascript won't have managed to form the final page outlook yet. For instance, the following <code>save_and_open_page</code> may open a page, that does not contain any beers yet:

```ruby
it "shows a known beer", js:true do
  visit beerlist_path
  save_and_open_page
  expect(page).to have_content "Nikolai"
end
```

As the page https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends says, Capybara is able to wait for asycronic Javascript calls till the page elements required for the tests have loaded.

It's known that the Javascript should add rows to the page table. You will get the page to look correct by adding the command <code>find('table').find('tr:nth-child(2)')</code> at its beginning. This looks for a table in the page and for the second line inside the table (the table first line is already the table title in the page template):

```ruby
it "shows a known beer", :js => true do
  visit beerlist_path
  find('table').find('tr:nth-child(2)')
  save_and_open_page
  expect(page).to have_content "Nikolai"
end
```


Capybara will now wait, moving to the command to open the page only when the table is loaded (to be more precise, only two lines of the table will be ready for sure).

Executing the test in a real browser is quite slow. You can make them faster by using Chrome's _Headless_ mode, the "UIless version". You can implement the Headless browser by changing the <code>before :all</code> into: 

```ruby
before :all do
  Capybara.register_driver :chrome do |app|
    Capybara::Selenium::Driver.new app, browser: :chrome,
      options: Selenium::WebDriver::Chrome::Options.new(args: %w[headless disable-gpu])
  end

  Capybara.javascript_driver = :chrome
  WebMock.disable_net_connect!(allow_localhost: true)
end
```

After these configuration changes, executing on a normal browser is possible by clearing the contents of <code>Options.new()</code>.

> ## Exercise 4
>
>Implement a test to check that the beers are sorted alphabetically by their name in the beerlist page by default.
>
>The test can be implemented by using the <code>find</code> selector to find the table rows and making sure that each line has the right contents. Because the table has one row for the header, the first actual row can be found like this:
>
> ```ruby
> find('#beertable').first('.tablerow')
> ```
>
>The row contents can be tested as usually with the expect and have_content methods. Capybara command _find_ returns a [Node](https://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders) type object. See the link for instructions on how to handle Node.



> ## Exercise 5
>
>Test the following pieces of functionality
>
> - clicking on the column 'style', the beers are sorted alphabetically by style name
> - clicking on the column 'brewery', the beers are sorted alphabetically by brewery name

## Asset pipeline

The Javascript and style files (and pictures) of Rails applications are managed with the so-called Asset pipeline, see http://guides.rubyonrails.org/asset_pipeline.html

The idea is that the application developer places the Javascript files in the folder <em>app/assets/javascripts</em> and the style files in <em>app/assets/stylesheets</em>. Both can be placed in various different files and subfolders, if needed.

When the application is in the development mode, Rails links all the Javascript and style files (which are defined in the so called Manifest file) together in to the application. If you check the application with the view source property of your browser, you will notice that a large amount of Javascript and style files are linked together there.

The Javascript files to link in to the application are defined in the file <em>app/assets/javascripts/application.js</em>, whose contents look like this now

```javascript
//= require jquery3
//= require popper
//= require bootstrap-sprockets
import "@hotwired/turbo-rails";
import "controllers";
import { beertable } from "custom/utils";

beertable();
```

Even though the require statements look as if they were comments, they are actually "real" commands of the  [sprockets compiler](https://github.com/sstephenson/sprockets) that takes care of asset pipeline. They help define the Javascript files that have to be linked in the application. The file tells to take jquery3, popper, and bootstrap-sprockets. These are all set up in the application through gems.

For execution performance reasons, it is usually better to avoid using too many Javascript and style files in production-use applications. When the application is started in production mode, Sprockets links all the application Javascript and style files into singular, optimised files. You'll notice this if you look at application HTML source code in Fly.io: for instance https://ratebeer22.fly.dev/, it contains now only one js and one css files, and expecially the js file readability is weak for an human.

More about asset pipiline and for instance Javascript linking in Rails applications, at:

- http://railscasts.com/episodes/279-understanding-the-asset-pipeline
- http://railsapps.github.io/rails-javascript-include-external.html

<blockquote>

## Exercises 6-8 (gives three points)

### The exercise is hard, so first make the easier ones. The other exercises don't depend on this.</h3>

Anyone can join as beer club member in your application, so far. Change your application now, so that the membership has to be confirmed by old members before new ones can join.

Some notes

<ul class="task-list">
<li>the best way to implement an unconfirmed membership is that the Membership model is added the boolean field <em>confirmed</em>
</li>
<li>When a club is created, the user who created it should automatically become that club's member</li>
<li>Show a list of the membership applications which haven't been confirmed on the club page</li>
<li>Membership status change can be managed for instance with its own [custom route](https://github.com/mluukkai/webdevelopment-rails/blob/main/week6.md#route-for-changing-the-brewery-status).</li>
</ul>

The exercise may be a bit challenging. [Active Record Associations guide](http://guides.rubyonrails.org/association_basics.html) section **4.3.3 Scopes for has_many**  provides a good tool to make the exercise. Of course, the exercise can also be solved in different ways.

Section **4.3.2.3 :class_name** might be useful as well.
</blockquote>

At the end of the exercise, you application can look something like this. The beer club page shows a list of the membership applications, if the signed-in user is already that beer club's member:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-6.png)


Users' personal pages show the applications which haven't been confirmed yet:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w6-5.png)


## Index to the database

When the user signs in the system, the session controller executes an operation to retrive the user object from the database against the user name:


```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by username: params[:username]

     # ...
  end

end
```

In order to execute the operation, the database has to go through the whole <code>users</code> table. Searches by the object ID are faster, because each table has been indexed against their ID. The index works as with hash tables, providing access to the required database row in "O(1)" time.

Database tables can be added other indexes if needed. Add an index to the <code>users</code> table, making the search against user name faster.

Create a migration for the index

    rails g migration AddUserIndexBasedOnUsername

The migration is the following:

```ruby
class AddUserIndexBasedOnUsername < ActiveRecord::Migration[5.2]
  def change
    add_index :users, :username
  end
end
```

Execute the migration with the command <code>rails db:migrate</code> and the index is ready!

The bad thing about this is that when the system is added a new user or an existing user is deleted, the index has to be edited and this requires time obviously. Adding an index is a tradeoff on what operation you want to optimize, then. In most cases database reading operations happen so much more often than writing operations that the benefits of indexes far outweight the extra work caused by upkeeping them.


## Lazy loading, the n+1 issue, and database request optimisation

The controller to show all beers is simple. The beers are fetched from the database, sorted according to what the parameter of the HTTP call defines, and are assigned to a variable for the template:

```ruby
def index
  @beers = Beer.all

  order = params[:order] || 'name'

  @beers = case order
            when 'name' then @beers.sort_by(&:name)
            when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
            when 'style' then @beers.sort_by{ |b| b.style.name }
            end
end
```

The template shows a table where the beers are listed:

```erb
<% @beers.each do |beer| %>
  <tr>
    <td><%= link_to beer.name, beer %></td>
    <td><%= link_to beer.style, beer.style %></td>
    <td><%= link_to beer.brewery.name, beer.brewery %></td>
  </tr>
<% end %>
</table>
```

Simple and stylish... but not too efficient.

You could take a look at your log file log/development.log to see what happens when users go to the beers page. You will have access to the same piece of information in a fairly better form  through the <em>miniprofiler</em> gem (see https://github.com/MiniProfiler/rack-mini-profiler and http://samsaffron.com/archive/2012/07/12/miniprofiler-ruby-edition)

Getting started with Miniprofiler is easy, you only need to add the following line to your Gemfile

    gem 'rack-mini-profiler'

Execute <code>bundle install</code> and restart your Rails server. When you go to the address http:localhost:300/beers next time, you'll see a timer will have appeared on the upper side of the page. This measures the time used to execute the HTTP request. If you click the number, you'll find a better definition of the time frame:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/profile1.png)

The report shows that <code>Executing action: index</code> – which is the controller method execution – causes one SQL request. Instead, <code>Rendering: beers/index</code> – which is the view template execution – causes notably more SQL requests!

Clicking on the requests you will be able to check their reason:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/profiler2.png)


In contradiction to the earlier report, the controller makes only one request

```ruby
SELECT "beers".* FROM "beers";
```

You see that the rendering of the view template performs the following requests several times:

```ruby
SELECT  "styles".* FROM "styles" WHERE "styles"."id" = ? LIMIT ?;

SELECT  "breweries".* FROM "breweries" WHERE "breweries"."id" = ? LIMIT ?;

SELECT AVG("ratings"."score") FROM "ratings" WHERE "ratings"."beer_id" = ?; 
```

In fact, a request to the tables of both <code>styles</code> and <code>breweries</code> is made for each singular beer.

The reason is that Activerecord is set up on <em>lazy loading</em> by default, and an object's fields are fetched from the database only when they are referred to. This is reasonable sometimes, if an object is related to a huge amount of objects and not all of them are needed straight from the beginning. When you access the page of all beers, lazy loading is not the best idea, because you know for sure that you have to show the brewery and style names for each beer, and these pieces of information can be found only from the brewery and style database tables.

You can guide the SQL generated from the requests with ActiveRecord method parameters. For instance, the following tells that breweries associated with the beers have to be fetched from the database as well:

```ruby
def index
  @beers = Beer.includes(:brewery).all
  # ...
end
```

With the help of Miniprofiler, you'll see that the controller execution now causes two requests:

```ruby
SELECT "beers".* FROM "beers";
SELECT "breweries".* FROM "breweries" WHERE "breweries"."id" IN (?, ?, ?, ?);
```

The view template execution causes request that are for example like this:

```ruby
SELECT  "styles".* FROM "styles" WHERE "styles"."id" = ? LIMIT ?;
SELECT AVG("ratings"."score") FROM "ratings" WHERE "ratings"."beer_id" = ?; 
```

When the view is rendered, the styles have to still be fetched from the database now, each with their own SQL request.

Optimize the controller in a way that all the styles and ratings needed are also read from the database at once:

```ruby
def index
  @beers = Beer.includes(:brewery, :style, :ratings).all

  # ...
end
```

You notice that while the number of requests has dropped, the request for finding the beer rating average is still repeated for every beer.

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/profiler3.png)

This is because you have defined that calculating the average is done with SQL:


```ruby
module RatingAverage
  extend ActiveSupport::Concern

  def average_rating
    # this generates SQL
    ratings.average(:score).to_f
  end
end
```

That means that having already fetched the ratings oesn't help here. We could make use of the beer ratings fetched with the _includes_ command in the avarage calculation by making it happen in the central memory instead of in SQL:

```ruby
module RatingAverage
  extend ActiveSupport::Concern

  def average_rating
    # Count and save based on the fetched ratings objects (associated to a beer)
    rating_count = ratings.size
    
    return 0 if rating_count == 0
    ratings.map{ |r| r.score }.sum / rating_count
  end
end
```

The controller execution now triggers less requests and rendering the view only a single one.

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/profiler4.png)

Miniprofiler shows that the request is

```ruby
SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1
```

and the reason is

```ruby
app/controllers/application_controller.rb:7:in `current_user'
```

that is, the reference to the signed-in user made with the `current_user` variable by the view. This isn't really a problem though. 

You managed to optimise the number of SQL requests and thus the loading time of the page! Decreasing the number on SQL requests is nice in the sense that it is constant and doesn't depend on the number of beers in the system.

What you just experienced is called the n+1 problem (see http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations): when fetching a list of beers with one database request, each object of the list causes a new database search, and instead of only one search, around n+1 searches actually happen.

Before the next exercise, make the user view_ views/users.html.erb_ a bit simpler:

```ruby
<h1>Users</h1>

<div id="users">
  <% @users.each do |user| %>
    <p>
      <%= link_to(user.username, user) %>
      <p>Has made <%= "#{user.ratings.size}"%> ratings, average rating <%= "#{user.average_rating}" %></p>
      <% if user.closed? %>
        <span class="badge text-bg-danger">account closed</span>
      <% end %>
    </p>
  <% end %>
</div>

```

Note, that the _if_ condition of <code>if user.closed</code> works depending on how you have named things in the code during an exercise on week 5. If necessary, you ran remove the whole condition.


>## Exercise 9
>
>There is a n+1 problem in the users page. Fix the problem eager loading the required objects when the users are fetched, like in the exercise above. Make sure that the optimisation works with miniprofiler.


<strong>Attention:</strong> if the table was added also the favourite beer column

```ruby
<% if user.favorite_beer %>
  <p>Favourite beer: <%= "#{user.favorite_beer.name}"%></p>
<% end %>
```

the situation would be a bit harder in terms of SQL optimisation. The latest version of your method was this:

```ruby
def favorite_beer
  return nil if ratings.empty?
  ratings.order(score: :desc).limit(1).first.beer
end
```

Now, not even eager loading will help, because the method call causes an SQL request in any case. Instead, if you implemented the beer ratings in the method central memory (as you did at the beginning of week 4):

```ruby
def favorite_beer
  return nil if ratings.empty?
  ratings.sort_by{ |r| r.score }.last.beer
end
```

the method call <em>would not</em> cause any database operations <em>if</em> the ratings are eager loaded when the method is called.

It may also make sense to keep two versions of the method in some situations to optimise its performance, one that executes the operation at database level and the other which does it in central memory.


## Server caching functionality

Create some more data in your database. Copy the following code to the file db/seeds.db

<div class="highlight highlight-ruby"><pre>users <span class="pl-k">=</span> <span class="pl-c1">200</span>           <span class="pl-c"># if your computer is slow, 100 will be enough 100</span>
breweries <span class="pl-k">=</span> <span class="pl-c1">100</span>       <span class="pl-c"># if your computer is slow, 50 will be enough</span>
beers_in_brewery <span class="pl-k">=</span> <span class="pl-c1">40</span>
ratings_per_user <span class="pl-k">=</span> <span class="pl-c1">30</span>

(<span class="pl-c1">1</span>..users).each <span class="pl-k">do </span>|<span class="pl-vo">i</span>|
  <span class="pl-s3">User</span>.create! <span class="pl-c1">username:</span><span class="pl-s1"><span class="pl-pds">"</span>user_<span class="pl-pse">#{</span><span class="pl-s2">i</span><span class="pl-pse"><span class="pl-s2">}</span></span><span class="pl-pds">"</span></span>, <span class="pl-c1">password:</span><span class="pl-s1"><span class="pl-pds">"</span>Passwd1<span class="pl-pds">"</span></span>, <span class="pl-c1">password_confirmation:</span><span class="pl-s1"><span class="pl-pds">"</span>Passwd1<span class="pl-pds">"</span></span>
<span class="pl-k">end</span>

(<span class="pl-c1">1</span>..breweries).each <span class="pl-k">do </span>|<span class="pl-vo">i</span>|
  <span class="pl-s3">Brewery</span>.create! <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Brewery_<span class="pl-pse">#{</span><span class="pl-s2">i</span><span class="pl-pse"><span class="pl-s2">}</span></span><span class="pl-pds">"</span></span>, <span class="pl-c1">year:</span><span class="pl-c1">1900</span>, <span class="pl-c1">active:</span><span class="pl-c1">true</span>
<span class="pl-k">end</span>

bulk <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create! <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>bulk<span class="pl-pds">"</span></span>, <span class="pl-c1">description:</span><span class="pl-s1"><span class="pl-pds">"</span>cheap, not much taste<span class="pl-pds">"</span></span>

<span class="pl-s3">Brewery</span>.all.each <span class="pl-k">do </span>|<span class="pl-vo">b</span>|
  n <span class="pl-k">=</span> rand(beers_in_brewery)
  (<span class="pl-c1">1</span>..n).each <span class="pl-k">do </span>|<span class="pl-vo">i</span>|
    beer <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.create! <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>beer <span class="pl-pse">#{</span><span class="pl-s2">b.id</span><span class="pl-pse"><span class="pl-s2">}</span></span> -- <span class="pl-pse">#{</span><span class="pl-s2">i</span><span class="pl-pse"><span class="pl-s2">}</span></span><span class="pl-pds">"</span></span>, <span class="pl-c1">style:</span>bulk
    b.beers <span class="pl-k">&lt;&lt;</span> beer
  <span class="pl-k">end</span>
<span class="pl-k">end</span>

<span class="pl-s3">User</span>.all.each <span class="pl-k">do </span>|<span class="pl-vo">u</span>|
  n <span class="pl-k">=</span> rand(ratings_per_user)
  beers <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.all.shuffle
  (<span class="pl-c1">1</span>..n).each <span class="pl-k">do </span>|<span class="pl-vo">i</span>|
    r <span class="pl-k">=</span> <span class="pl-s3">Rating</span>.<span class="pl-k">new</span> <span class="pl-c1">score:</span>(<span class="pl-c1">1</span><span class="pl-k">+</span>rand(<span class="pl-c1">50</span>))
    beers[i].ratings <span class="pl-k">&lt;&lt;</span> r
    u.ratings <span class="pl-k">&lt;&lt;</span> r
  <span class="pl-k">end</span>
<span class="pl-k">end</span></pre></div>

The file uses the version with the exclamation mark (<code>create!</code>) to create objects instead of the normal <code>create</code> methods. The difference between the two is can be seen when an object cannot be created successfully. The method without exclamation mark returns <code>nil</code> in such cases, whereas the other sends an exeption. Sending exeptions is a better options in seeding, otherwise the unsuccessful creation will be left unnoticed.

<strong>Make a copy of the old database <em>db/development.db</em></strong>, so that you may return to the old situation in case you compromise the performance. You can take the old database up again by changing its name again and calling it development.db

<strong>Attention:</strong> this might not be the best way to test the performance of real Rails applications, more about the topic at <a href="http://guides.rubyonrails.org/v3.2.13/performance_testing.html">http://guides.rubyonrails.org/v3.2.13/performance_testing.html</a> (the guides do not include a version updated for Rails 4.)

Executing the seeding with the command

<pre><code>rake db:seed
</code></pre>

Executing the script will take a while.

<strong>Attention:</strong> if executing the script ends in an error, you'd better return to the pre-script database status. A possible issue about executing the script is that duplicate names braking the validation. If you change the command <code>create!</code> to <code>create</code>, the script execution will not interrupt.

Your application will have enough data now, and loading the pages will start being slower.

Try now to see how the performance will be different if you remove the SQL request optimisation comments from beers page, so if you change back the beer controller to look like this:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-c"># @beers = Beer.includes(:brewery, :style).all</span>
    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.all

    order <span class="pl-k">=</span> params[<span class="pl-c1">:order</span>] <span class="pl-k">||</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>

    <span class="pl-k">case</span> order
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>brewery<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.brewery.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>style<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.style.name }
    <span class="pl-k">end</span>
  <span class="pl-k">end</span></pre></div>

Also have a look at the amount of SQL requests executing with and without optimisation. Once again, you'll see the result easily with miniprofiler.

After trying this out, you can return the code to its optimised form.

When the amount of data is huge, optimising the requests alone will not be enough, but you'll have to make up new strategies.

Caching will be an option.

Web application caching can be implemented both for the browser and for the serve (as well as for the proxy between the two). Take a look at server caching. A couple of weeks ago, you cached already the information fetched from the beermapping api 'manually' through Rails.cache. You'll have a look now to some more automatic caching mechanism.

Rails provides server caching solutions at three levels:

<ul class="task-list">
<li>page cache (caching whole pages)</li>
<li>action cache (caching the controller methods results)</li>
<li>fragment cache (caching a page fragment) </li>
</ul>

See <a href="http://guides.rubyonrails.org/caching_with_rails.html">http://guides.rubyonrails.org/caching_with_rails.html</a>

Rails 4 has removed the first two solutions from Rails core, but you can make use of the with the help of different gems. However, you'll only look into the last solution now, that is fragment cache, which is the most diverse caching strategy, in fact.

Caching is not on by default, when yuo execute your application in development mood. In the file <em>config/environment/development.rb</em>, you can turn caching on by giving <code>true</code> to the value of the following line:

<pre><code>config.action_controller.perform_caching = false
</code></pre>

Restart your application now.

Cache how the beers list is viewed.

Fragment cache is possible if you put the part to cache of the view template into the following chunk:

<div class="highlight highlight-erb"><pre><span class="pl-pse">&lt;%</span><span class="pl-s2"> cache <span class="pl-s1"><span class="pl-pds">'</span>key<span class="pl-pds">'</span></span>, <span class="pl-c1">skip_digest:</span> <span class="pl-c1">true</span> <span class="pl-k">do </span></span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
  the view template part to cache
<span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span></pre></div>

As you may have guessed, <code>key</code> os the key where you save the view fragment to cache. The key can be either a string or an object. <code>skip_digest: true</code> refers to the <a href="http://blog.remarkablelabs.com/2012/12/russian-doll-caching-cache-digests-rails-4-countdown-to-2013">view templates versioning</a> which won't be covered now. This means anyway, that the cache should be cleared (with the command <code>Rails.cache.clear</code>) <em>if</em> the view template code is changed.

Adding fragment cache to the beers list views/beers/index.html is easy, you'll cache the dynamic part of the page, the beers table:

<div class="highlight highlight-erb"><pre>&lt;<span class="pl-ent">h1</span>&gt;Beers&lt;/<span class="pl-ent">h1</span>&gt;

<span class="pl-pse">&lt;%</span><span class="pl-s2"> cache <span class="pl-s1"><span class="pl-pds">'</span>beerlist<span class="pl-pds">'</span></span>, <span class="pl-c1">skip_digest:</span> <span class="pl-c1">true</span> <span class="pl-k">do </span></span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>

&lt;<span class="pl-ent">table</span> <span class="pl-e">class</span>=<span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">thead</span>&gt;
    &lt;<span class="pl-ent">tr</span>&gt;
      &lt;<span class="pl-ent">th</span>&gt; <span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to <span class="pl-s1"><span class="pl-pds">'</span>Name<span class="pl-pds">'</span></span>, beers_path(<span class="pl-c1">order:</span><span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>) </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span> &lt;/<span class="pl-ent">th</span>&gt;
      &lt;<span class="pl-ent">th</span>&gt; <span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to <span class="pl-s1"><span class="pl-pds">'</span>Style<span class="pl-pds">'</span></span>, beers_path(<span class="pl-c1">order:</span><span class="pl-s1"><span class="pl-pds">"</span>style<span class="pl-pds">"</span></span>) </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span> &lt;/<span class="pl-ent">th</span>&gt;
      &lt;<span class="pl-ent">th</span>&gt; <span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to <span class="pl-s1"><span class="pl-pds">'</span>Brewery<span class="pl-pds">'</span></span>, beers_path(<span class="pl-c1">order:</span><span class="pl-s1"><span class="pl-pds">"</span>brewery<span class="pl-pds">"</span></span>) </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span> &lt;/<span class="pl-ent">th</span>&gt;
    &lt;/<span class="pl-ent">tr</span>&gt;
  &lt;/<span class="pl-ent">thead</span>&gt;

  &lt;<span class="pl-ent">tbody</span>&gt;
    <span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-vo">@beers</span>.each <span class="pl-k">do </span>|<span class="pl-vo">beer</span>| </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
      &lt;<span class="pl-ent">tr</span>&gt;
        &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.name, beer </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
        &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.style, beer.style </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
        &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.brewery.name, beer.brewery </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
      &lt;/<span class="pl-ent">tr</span>&gt;
    <span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
  &lt;/<span class="pl-ent">tbody</span>&gt;
&lt;/<span class="pl-ent">table</span>&gt;

<span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>

&lt;<span class="pl-ent">br</span>&gt;

<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to(<span class="pl-s1"><span class="pl-pds">'</span>New Beer<span class="pl-pds">'</span></span>, new_beer_path, <span class="pl-c1">class:</span><span class="pl-s1"><span class="pl-pds">'</span>btn btn-primary<span class="pl-pds">'</span></span>) <span class="pl-k">if</span> current_user </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span></pre></div>

If you go to the page now, the page fragment hasn't been saved to memory yet, and loading the page will take as long as it used to before adding the caching:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-9.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-9.png" alt="kuva" style="max-width:100%;"></a>

The rendering time (<code>Rendering: beers/index</code>) was 814 milliseconds, in fact.

The fragment used on the page saves in cache, and opening the following page will be much faster:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-10.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-10.png" alt="kuva" style="max-width:100%;"></a>

So, the view templates renders in three milliseconds!

Attention: the new beer creation link should not be left inside the cached fragment, because the link has to be shown only to users who signed in. The cached part of the page will be shown to everyone in the same way.

If you create a new beer now, you'll see that the page won't contain the new beer information. The reason is however that the page fragment is to be found in cache again. The point is that the expired view fragment should not be shown anymore. In this case, the easiest strategy is making it expired manually from the controller.

Expiration is possible with the command <code>expire_fragment(avain)</code>. The command has to be called from the controller in the points where the beers list contents may change. These points are the controller methods <code>create</code>, <code>update</code> and <code>destroy</code>. The change is easy:

<div class="highlight highlight-erb"><pre>  def create
    expire_fragment('beerlist')
  end

  def update
    expire_fragment('beerlist')
    # ...
  end

  def destroy
    expire_fragment('beerlist')
    # ...
  end</pre></div>

After the changes the page will work as expected!

The page of all beers could also be speeded up a bit. It is the controller which executes now the database operation

<div class="highlight highlight-erb"><pre>    @beers = Beer.includes(:brewery, :style).all</pre></div>

even when the page fragment is found from cache memory. You could test whether the fragment exists with the method <code>fragment_exist?</code>, and execute the database operation only if the fragment does not exist (reminder: unless means the same as if not):

<div class="highlight highlight-erb"><pre>  def index
    unless fragment_exist?( 'beerlist' )
      @beers = Beer.includes(:brewery, :style).all

      order = params[:order] || 'name'

      case order
        when 'name' then @beers.sort_by!{ |b| b.name }
        when 'brewery' then @beers.sort_by!{ |b| b.brewery.name }
        when 'style' then @beers.sort_by!{ |b| b.style.name }
      end
    end
  end</pre></div>

The controller looks more painful but the page is faster:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-11.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-11.png" alt="kuva" style="max-width:100%;"></a>

You could clean up the solution a bit by returning the <code>index</code> method to its original state, and checking whether the fragments exists in a before filter:

<div class="highlight highlight-ruby"><pre>  before_action <span class="pl-c1">:skip_if_cached</span>, <span class="pl-c1">only:</span>[<span class="pl-c1">:index</span>]

  <span class="pl-k">def</span> <span class="pl-en">skip_if_cached</span>
    <span class="pl-k">return</span> render <span class="pl-c1">:index</span> <span class="pl-k">if</span> fragment_exist?( <span class="pl-s1"><span class="pl-pds">'</span>beerlist<span class="pl-pds">'</span></span> )
  <span class="pl-k">end</span></pre></div>

If the before filter <code>skip_if_cached</code> notices that the fragment exists, it renders the view template straight away. In such case, the real controller method won't be executed at all.

Notice however that the page has a little problem. The beers were sorted in different orders by clicking on the columns. Caching has broken this operation anyway!

One way to fix the functionality is adding an order to the fragment key:

<div class="highlight highlight-erb"><pre><span class="pl-pse">&lt;%</span><span class="pl-s2"> cache <span class="pl-s1"><span class="pl-pds">"</span>beerlist-<span class="pl-pse">#{</span><span class="pl-s2"><span class="pl-vo">@order</span></span><span class="pl-pse"><span class="pl-s2">}</span></span><span class="pl-pds">"</span></span>, <span class="pl-c1">skip_digest:</span> <span class="pl-c1">true</span> <span class="pl-k">do </span></span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
   table html
<span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span></pre></div>

The order is stored in the variable <code>@order</code>, in the controller. The changes required by the controller follow below:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">skip_if_cached</span>
    <span class="pl-vo">@order</span> <span class="pl-k">=</span> params[<span class="pl-c1">:order</span>] <span class="pl-k">||</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>
    <span class="pl-k">return</span> render <span class="pl-c1">:index</span> <span class="pl-k">if</span> fragment_exist?( <span class="pl-s1"><span class="pl-pds">"</span>beerlist-<span class="pl-pse">#{</span><span class="pl-s2"><span class="pl-vo">@order</span></span><span class="pl-pse"><span class="pl-s2">}</span></span><span class="pl-pds">"</span></span>  )
  <span class="pl-k">end</span>

  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.includes(<span class="pl-c1">:brewery</span>, <span class="pl-c1">:style</span>).all

    <span class="pl-k">case</span> <span class="pl-vo">@order</span>
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>brewery<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.brewery.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>style<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by!{ |<span class="pl-vo">b</span>| b.style.name }
    <span class="pl-k">end</span>
  <span class="pl-k">end</span></pre></div>

so, because the controller method does not have to be executed necessarily, the order is saved by the before filter.

When the fragment expires, all three orders have to expire:

<div class="highlight highlight-ruby"><pre>   [<span class="pl-s1"><span class="pl-pds">"</span>beerlist-name<span class="pl-pds">"</span></span>, <span class="pl-s1"><span class="pl-pds">"</span>beerlist-brewery<span class="pl-pds">"</span></span>, <span class="pl-s1"><span class="pl-pds">"</span>beerlist-style<span class="pl-pds">"</span></span>].each{ |<span class="pl-vo">f</span>| expire_fragment(f) }</pre></div>

<strong>Attention:</strong> you can call the fragment cache operation from the console:

<div class="highlight highlight-ruby"><pre>irb(main):<span class="pl-c1">043</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> <span class="pl-s3">ActionController</span>::<span class="pl-s3">Base</span>.<span class="pl-k">new</span>.fragment_exist?( <span class="pl-s1"><span class="pl-pds">'</span>beerlist-name<span class="pl-pds">'</span></span> )
<span class="pl-vo">Exist</span> fragment? views<span class="pl-k">/</span>beerlist<span class="pl-k">-</span>name (<span class="pl-c1">0</span>.4ms)
=&gt; <span class="pl-c1">true</span>
irb(main):<span class="pl-c1">044</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> <span class="pl-s3">ActionController</span>::<span class="pl-s3">Base</span>.<span class="pl-k">new</span>.expire_fragment( <span class="pl-s1"><span class="pl-pds">'</span>beerlist-name<span class="pl-pds">'</span></span> )
<span class="pl-vo">Expire</span> fragment views<span class="pl-k">/</span>beerlist<span class="pl-k">-</span>name (<span class="pl-c1">0</span>.6ms)
=&gt; <span class="pl-c1">true</span>
irb(main):<span class="pl-c1">045</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> <span class="pl-s3">ActionController</span>::<span class="pl-s3">Base</span>.<span class="pl-k">new</span>.fragment_exist?( <span class="pl-s1"><span class="pl-pds">'</span>beerlist-name<span class="pl-pds">'</span></span> )
<span class="pl-vo">Exist</span> fragment? views<span class="pl-k">/</span>beerlist<span class="pl-k">-</span>name (<span class="pl-c1">0</span>.1ms)
=&gt; <span class="pl-c1">nil</span></pre></div>

<strong>Attention2:</strong> there is a better debugging tool than the console. In developement mood, the part of the page that hasn't been cached should be given the information of what fragments can be found from cache and what the current page fragment is. Add the following code to the beers page upper part:

<div class="highlight highlight-html"><pre>&lt;<span class="pl-ent">div</span> <span class="pl-e">style</span>=<span class="pl-s1"><span class="pl-pds">"</span>border-style: solid;<span class="pl-pds">"</span></span>&gt;
  beerlist-name: &lt;%= ActionController::Base.new.fragment_exist?( 'beerlist-name' ) %&gt;
  &lt;<span class="pl-ent">br</span>&gt;
  beerlist-style: &lt;%= ActionController::Base.new.fragment_exist?( 'beerlist-style' ) %&gt;
  &lt;<span class="pl-ent">br</span>&gt;
  beerlist-brewery: &lt;%= ActionController::Base.new.fragment_exist?( 'beerlist-brewery' ) %&gt;
  &lt;<span class="pl-ent">br</span>&gt;
  current: &lt;%= "beerlist-#{@order}" %&gt;
&lt;/<span class="pl-ent">div</span>&gt;</pre></div>

The page upper border will contain a box now, telling the cache fragments state, that is, whether a fragment exists or not:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-6.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-6.png" alt="kuva" style="max-width:100%;"></a>

<strong>Attention3:</strong> because the debug box is on the page upper side – before the code that the fragment possibly generates – <strong>the page has to be reloaded always</strong> if you want that the debug box shows the up-to-date fragment situation.

<blockquote>

<a id="user-content-tehtävä-10" class="anchor" href="#teht%C3%A4v%C3%A4-10" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 10

Implement fragment cache to the breweries page. Make sure that cache expires when there's a change on the page contents. You can avoid implementing the exercise 2 modification that helped to sort the breweries in descending order by clicking on the column name again.
</blockquote>

If you wanted to cache a singular beer page, you'd better define the object to be cached as fragment key:

<div class="highlight highlight-ruby"><pre><span class="pl-k">&lt;</span><span class="pl-k">%</span> cache <span class="pl-vo">@beer</span> <span class="pl-k">do </span><span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">    &lt;h2<span class="pl-pds">&gt;</span></span>
      <span class="pl-k">&lt;</span><span class="pl-k">%=</span> <span class="pl-vo">@beer</span>.name <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">    &lt;/h2<span class="pl-pds">&gt;</span></span>

    <span class="pl-k">&lt;</span>p<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span><span class="pl-k">%=</span> link_to <span class="pl-vo">@beer</span>.style, <span class="pl-vo">@beer</span>.style <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">      brewed by</span>
<span class="pl-s1">      &lt;%= link_to @beer.brewery.name, @beer.brewery %<span class="pl-pds">&gt;</span></span>
    <span class="pl-k">&lt;</span><span class="pl-k">/</span>p<span class="pl-k">&gt;</span>

    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">if</span> <span class="pl-vo">@beer</span>.ratings.empty? <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">        &lt;p<span class="pl-pds">&gt;</span></span>beer has <span class="pl-k">not</span> yet been rated! <span class="pl-k">&lt;</span><span class="pl-k">/</span>p<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">else</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">        &lt;p<span class="pl-pds">&gt;</span></span><span class="pl-vo">Has</span> <span class="pl-k">&lt;</span><span class="pl-k">%=</span> pluralize(<span class="pl-vo">@beer</span>.ratings.count,<span class="pl-s1"><span class="pl-pds">'</span>rating<span class="pl-pds">'</span></span>) <span class="pl-s1"><span class="pl-pds">%&gt;</span>, average &lt;%= round(@beer.average_rating) %<span class="pl-pds">&gt;</span></span> <span class="pl-k">&lt;</span><span class="pl-k">/</span>p<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">end</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">&lt;% end %<span class="pl-pds">&gt;</span></span>

<span class="pl-k">&lt;</span>!<span class="pl-k">-</span> cachaamaton osa <span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">if</span> current_user <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">    &lt;h4<span class="pl-pds">&gt;</span></span>give a <span class="pl-c1">rating:</span><span class="pl-k">&lt;</span><span class="pl-k">/</span>h4<span class="pl-k">&gt;</span>

    <span class="pl-k">&lt;</span><span class="pl-k">%=</span> form_for(<span class="pl-vo">@rating</span>) <span class="pl-k">do </span>|<span class="pl-vo">f</span>| <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">        &lt;%= f.hidden_field :beer_id %<span class="pl-pds">&gt;</span></span>
        <span class="pl-c1">score:</span> <span class="pl-k">&lt;</span><span class="pl-k">%=</span> f.number_field <span class="pl-c1">:score</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">        &lt;%= f.submit class:"btn btn-primary"  %<span class="pl-pds">&gt;</span></span>
    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">end</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">    &lt;p<span class="pl-pds">&gt;</span></span><span class="pl-k">&lt;</span><span class="pl-k">/</span>p<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">end</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">&lt;%= edit_and_destroy_buttons(@beer) %<span class="pl-pds">&gt;</span></span></pre></div>

The fragment key will be a string now, which Rails generates by calling the object name <code>cache_key</code>. The method generates a key which unifies the object <em>and</em> creates a date stamp referring to when the object was last modified. If the object fields are modified, the fragment key value is also modified, meaning that the old fragment expires automatically. Below an example about a cache key that is generated automatically:

<div class="highlight highlight-ruby"><pre>irb(main):<span class="pl-c1">006</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> b <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.first
irb(main):<span class="pl-c1">006</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> b.cache_key
=&gt; <span class="pl-s1"><span class="pl-pds">"</span>beers/1-20150106210715764510000<span class="pl-pds">"</span></span>
irb(main):<span class="pl-c1">014</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> b.update_attribute(<span class="pl-c1">:name</span>, <span class="pl-s1"><span class="pl-pds">'</span>ISO 4<span class="pl-pds">'</span></span>)
irb(main):<span class="pl-c1">015</span>:<span class="pl-c1">0</span><span class="pl-k">&gt;</span> b.cache_key
=&gt; <span class="pl-s1"><span class="pl-pds">"</span>beers/1-20150211200158000903000<span class="pl-pds">"</span></span></pre></div>

The solution is not perfect yet. If the beer is made a new rating, the object itself does not change and the fragment does not expire. The issue is easy to fix, though. Rating should be added the information that when it is created, modified, or distroyed, also its beer has to be 'influenced':

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">Rating<span class="pl-e"> &lt; ActiveRecord::Base</span></span>
  belongs_to <span class="pl-c1">:beer</span>, <span class="pl-c1">touch:</span> <span class="pl-c1">true</span>

  <span class="pl-c"># ...</span>
<span class="pl-k">end</span></pre></div>

In fact the <code>touch: true</code> connected to <code>belongs_to</code> causes that the object field <code>updated_at</code> at the other end of the connection will also be updated.

<blockquote>

<a id="user-content-tehtävä-11" class="anchor" href="#teht%C3%A4v%C3%A4-11" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 11

Implement fragment cache for the page of singular beers. Notice that the brewery page fragment expires automatically if the brewery beers are modified, like as it is in the example above.
</blockquote>

When you have to implement cache explicit expiring as it is for the page with all beers, it is always a bit sensible because there is the small risk that you forget to expire the fragment in all the required parts of the code.

When you use straight the object as fragment key (as you did in the page of singular beers), the cache expires automatically when the object is updated. It would also be possible to make the cache of the page with all beers expire automatically. To do this, you should generate the fragment key with a suitable method; see the guide <a href="http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching">Caching with Rails: An overview</a> at the point "If you want to avoid expiring the fragment manually..."


<a id="user-content-selainpuolen-cachays" class="anchor" href="#selainpuolen-cachays" aria-hidden="true"><span class="octicon octicon-link"></span></a>Browser  caching

Caching is possible at different levels, also at browser level. If you were interested about the topic, I suggest you should take a look at Codeschool video to know more about how to use _etag_ on Rails application, see <a href="http://rails4.codeschool.com/videos">http://rails4.codeschool.com/videos</a>

Other Rails 4 Codeschool videos are warmly recommended!


<a id="user-content-eventual-consistency" class="anchor" href="#eventual-consistency" aria-hidden="true"><span class="octicon octicon-link"></span></a>Eventual consistency

It is not essential that users should see the application situation which is really up to date. For instance, it is not critical that the ratings statistics page shows a situation which is few minute old. The important thing is that all the data coming to the system is not shown too late to users. The name <a href="http://en.wikipedia.org/wiki/Eventual_consistency"><strong>eventual consistency</strong> describes this not-too-strict demand for up-to-date information, which can improve the application performance sensibly.</a>.

Eventual consistency is easy to define in Rails by setting an expiring time to fragment cache:

<div class="highlight highlight-erb"><pre><span class="pl-pse">&lt;%</span><span class="pl-s2"> cache <span class="pl-s1"><span class="pl-pds">'</span>fragment_name<span class="pl-pds">'</span></span>, <span class="pl-c1">expires_in:</span><span class="pl-c1">10</span>.minutes <span class="pl-k">do </span></span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
  ...
<span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span></pre></div>

This simplifies the application also because expiring the cache becomes easy, now that you don't have to take into consideration all the small parts of the code that may cause time inconsistency.

Suppose that your system really had many users and ratings happened frequently every minute. If you wanted to show complitely consistent information on the ratings page, the performance would become weak, because every beer rating would change the page status, and the page should expire too often. This would make caching almost useless.

SQL optimisation and caching are not yet able to make the ratings page too fast, because controller operations like <code>User.top(3)</code> requre that all the database data should be parsed, in fact. If you wanted to optimise your page better than this, you should use even more sofisticated tools. For instance, the performance of <code>User.top</code> would improve dramatically if the ratings amount was saved straight in the user objects, so that finding out this information wouldn't require calculating the amount of Raiting objects of the user. This would also require that when a user makes a new rating, the user object should also be updated. So executing the rating operation would be slightly slower.

Another and maybe better way to accelerate the rating page would be caching the information needed by the Rails.cache controller. So the controller is this

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@top_beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.top(<span class="pl-c1">3</span>)
    <span class="pl-vo">@top_breweries</span> <span class="pl-k">=</span> <span class="pl-s3">Brewery</span>.top(<span class="pl-c1">3</span>)
    <span class="pl-vo">@top_styles</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.top(<span class="pl-c1">3</span>)
    <span class="pl-vo">@most_active_users</span> <span class="pl-k">=</span> <span class="pl-s3">User</span>.most_active(<span class="pl-c1">3</span>)
    <span class="pl-vo">@recent_ratings</span> <span class="pl-k">=</span> <span class="pl-s3">Rating</span>.recent
    <span class="pl-vo">@ratings</span> <span class="pl-k">=</span> <span class="pl-s3">Rating</span>.all
  <span class="pl-k">end</span></pre></div>

You could do the same as you did in week 5 when you stored the brewery information into Rails cache, so the controller would look more or less like this:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-s3">Rails</span>.cache.write(<span class="pl-s1"><span class="pl-pds">"</span>beer top 3<span class="pl-pds">"</span></span>, <span class="pl-s3">Beer</span>.top(<span class="pl-c1">3</span>)) <span class="pl-k">if</span> cache_does_not_contain_data_or_it_is_too_old
    <span class="pl-vo">@top_beers</span> <span class="pl-k">=</span> <span class="pl-s3">Rails</span>.cache.read <span class="pl-s1"><span class="pl-pds">"</span>beer top 3<span class="pl-pds">"</span></span>

    <span class="pl-c"># ...</span>
  <span class="pl-k">end</span></pre></div>

The course page to mark the exercises <a href="http://wadrorstats2015.herokuapp.com/">http://wadrorstats2015.herokuapp.com/</a> works quite fast now even though the page shows the statistics of all the exercises submitted to the system. The submissions are more than 600, so far. In fact, the main page performance would not change even though there were a thousand times more submissions.

The page stores each submission in a <code>Submission</code> object. When the main page is generated, if the submission statistics were computed going through all the <code>Submission</code> objects, this would slow down according to how many submissions there are in the system.

The application works anyway that the submission statistics of each week are computed beforehand in <code>WeekStatistic</code> objects. The application updates the week statistics after each new or updated submission. This slows down the actual submission marginally (the operation bottleneck is the email delivery, which takes 95% of the time consumption), but improves significantly how fast the main page is loaded. So the main page information of week 7 can be generated thanks to seven objects, the objects are fetched from the database with one SQL request, and the actual page generation performance is completely independent on the number of submissions.

The application code can be found at <a href="https://github.com/mluukkai/wadrorstats">https://github.com/mluukkai/wadrorstats</a>

Optimising applications performance is not necessarily the easiest thing to do, because it requires solutions at different levels and you will often have to tailor them according to the situation. Optimisation will most probably make your code less pleasant to read.


<a id="user-content-asynkronisuus-viestijonot-ja-taustatyöt" class="anchor" href="#asynkronisuus-viestijonot-ja-taustaty%C3%B6t" aria-hidden="true"><span class="octicon octicon-link"></span></a>Asyncrony, message queues and background work

A negative part of the fact cache expires from time to time is that if you used that strategy for the ratings page, this would take time from some users when the operation is executed and the data has to be generated again for the cache memory.

A better solution would be if users were always provided with data which is as updated as possible, so the controller would be like:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@top_beers</span> <span class="pl-k">=</span> <span class="pl-s3">Rails</span>.cache.read <span class="pl-s1"><span class="pl-pds">"</span>beer top 3<span class="pl-pds">"</span></span>

    <span class="pl-c"># ...</span>
  <span class="pl-k">end</span></pre></div>

The cache update could be executed as a process/thread on the background, which wakes up from time to time:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">background_worker</span>
    <span class="pl-k">while</span> <span class="pl-c1">true</span> <span class="pl-k">do</span>
       sleep <span class="pl-c1">10</span>.minutes
       <span class="pl-s3">Rails</span>.cache.write(<span class="pl-s1"><span class="pl-pds">"</span>beer top 3<span class="pl-pds">"</span></span>, <span class="pl-s3">Beer</span>.top(<span class="pl-c1">3</span>))
       <span class="pl-s3">Rails</span>.cache.write(<span class="pl-s1"><span class="pl-pds">"</span>brewery top 3<span class="pl-pds">"</span></span>, <span class="pl-s3">Brewery</span>.top(<span class="pl-c1">3</span>))
       <span class="pl-c"># ...</span>
    <span class="pl-k">end</span>
  <span class="pl-k">end</span></pre></div>

The background processing strategy shown above is simple because the application and the thread/process executing the background processing don't need to syncronise their activity. Then again, sometimes the background processing is required by a request coming to the application. In such case, the syncronisation between the application and the background processing can be handled through message queues.

There are various options in Rails to implement background processing whether it is managed with message queues or with singular processes or threads. The best option for the moment is <a href="http://railscasts.com/episodes/366-sidekiq">Sidekiq</a>.

Before Rails 4, Rails application worked with only one thread by default. This caused anyway that the application handled HTTP requests one after the other (unless the server system hadn't defined that various instances of the application were running simultaneously, see for instance <a href="https://devcenter.heroku.com/articles/rails-unicorn">https://devcenter.heroku.com/articles/rails-unicorn</a>). Starting from Rails 4, handling each request in their own thread is permetted by default. It is good to consider though, that Rails default sever system WEBrick <em>does not</em> support the functionality to handle threaded requests. If you need threads, you should use <a href="http://puma.io/">Puma</a> as sever system. It is <a href="https://discussion.heroku.com/t/using-puma-on-heroku/150">easy</a> to use Puma.

More about what is needed to make threaded Rails applications from<a href="https://www.cs.helsinki.fi/i/mluukkai/365-thread-safety.mp4">Rails cast</a>. Notice that after allowing threads you will have to take care of the thread safety!

<blockquote>

<a id="user-content-tehtävä-12" class="anchor" href="#teht%C3%A4v%C3%A4-12" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 12

Accelerate ratings page performance. You can assume that users are satisfied with eventual consistency. If you want you can use background processing libraries – but you don't have to, or at least you don't have to deploy it to Heroku. In any case, <a href="http://blog.mattheworiordan.com/post/44862390383/running-sidekiq-concurrently-on-a-single-worker">this would not be difficult either</a>. Write a small description of your strategy to speed up the page in the <code>index</code> method of the ratings controller, in case it is not so clear based on the code.
</blockquote>


<a id="user-content-sovelluksen-koostaminen-palveluista" class="anchor" href="#sovelluksen-koostaminen-palveluista" aria-hidden="true"><span class="octicon octicon-link"></span></a>An application made of various services

You can manage to scale your application performance only till a certain point if your application is a monolitic entity which has to be executed with one sever and that relies completely on one database. The application can be optimised so that it is scaled <strong>horizontally</strong> – that is, increasing the physical resources of its server.

You will have better scaling results if you act <strong>vertically</strong>, meaning that instead of improving the physical resources of one server, you start to use various servers, all executing your application actions at the same time. Vertical scaling is not necessarily trivial, you'll have to change the application architecture. If your application works with only one database, you may run into trubles despite vertical scaling, expecially if that is a relation database, that is not easy to hash and to scale vertically.

Scaling an application (and sometimes also updating and extending it) is easier if the application is made of various different <strong>services</strong> that work indipendently and communicate with each other for instance with an HTTP protocol. In fact, your application is already making use of another service, that is BeermappingAPI. In the same way, your application functionality could be expanded if new services were integrated.

For instance, suppose you wanted that your application created food receipt suggestions for users based on their favourite beer style and location (that you can find looking at the IP location of the user's HTTP request). You should implement the recommendation as an independent service. Your application would communicate with the service using the HTTP protocol.

Suppose instead that you wanted your application recommend beers to users based on their own favourite style. Separating this functionality into its own service that works on an independent server would be a bit more challenging. In fact, recommendations would depend most probably on other people's ratings. It would mean that accessing this information would require that the recommender accessed your application database. So if you wanted to implement the beers recommender as a separate service, you may need to rethink the whole structure of your application. You would need to make sure that the information about the ratings can be shared between the ratabeer application and the beer recommendation service.


<a id="user-content-single-sign-on" class="anchor" href="#single-sign-on" aria-hidden="true"><span class="octicon octicon-link"></span></a>Single sign on

Many current Web sites make it possible to sign on with your Google, Facebook or GitHub IDs, just to name a few. The Web sites have outsourced the user management and authentication to separate services.

Authentication is managed using the OAuth2 standard (see <a href="https://tools.ietf.org/html/draft-ietf-oauth-v2-31">https://tools.ietf.org/html/draft-ietf-oauth-v2-31</a>). More about OAuth authentication for instance at <a href="http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified">http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified</a>

Authentication that is based on OAuth can be managed easily on Rails with the Omniauth gem, see <a href="http://www.omniauth.org/">http://www.omniauth.org/</a> Nowadays, every service provider has got its own gem, for instance <a href="https://github.com/intridea/omniauth-github">omniauth github</a>

<blockquote>

<a id="user-content-tehtävä-13" class="anchor" href="#teht%C3%A4v%C3%A4-13" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 13

Make it possible to sign on your application with GitHub IDs. Proceed in the following way:

<ul class="task-list">
<li>Sign in GitHub and go to the <a href="https://github.com/settings/profile">settings page</a>. Choose <em>Applications</em> on the left, and then click on <em>Register new Application</em>. Define http://localhost:3000 as <em>homepage url</em>, and http://localhost:3000/auth/github/callback as <em>ahthorization callback url</em></li>
<li>Set up <a href="https://github.com/intridea/omniauth-github">omniauth github</a></li>
<li>Create the file <em>omniauth.rb</em> in the folder <em>config/initializers</em>, containing the following code:</li>
</ul>

<div class="highlight highlight-ruby"><pre><span class="pl-s3">Rails</span>.application.config.middleware.use <span class="pl-s3">OmniAuth</span>::<span class="pl-vo">Builder</span> <span class="pl-k">do</span>
 provider <span class="pl-c1">:github</span>, <span class="pl-vo">ENV</span>[<span class="pl-s1"><span class="pl-pds">'</span>GITHUB_KEY<span class="pl-pds">'</span></span>], <span class="pl-vo">ENV</span>[<span class="pl-s1"><span class="pl-pds">'</span>GITHUB_SECRET<span class="pl-pds">'</span></span>]
<span class="pl-k">end</span></pre></div>

<ul class="task-list">
<li>Set up in GitHub the <em>client id</em> and the <em>client secret</em> on the page of your application as <a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko5.md#sovelluskohtaisen-datan-tallentaminen">environment variable ympäristömuuttujien</a> values</li>
<li>Add the route below to the file <em>routes.rb</em></li>
</ul>

<div class="highlight highlight-ruby"><pre>  get <span class="pl-s1"><span class="pl-pds">'</span>auth/:provider/callback<span class="pl-pds">'</span></span>, <span class="pl-c1">to:</span> <span class="pl-s1"><span class="pl-pds">'</span>sessions#create_oauth<span class="pl-pds">'</span></span></pre></div>

<ul class="task-list">
<li>Create the controller method defined by the route in the <em>session controller</em>
</li>
<li>Make a button for your application so that users can click on it to sign on your application with their GitHub IDs. The button path is <em>auth/github</em>
</li>
<li>When you write on your application with GitHub IDs, the browser is redirected to the address <em>auth/github/callback</em>. This means that thanks to the definition in routes.rb, the execution is moved to the <em>create_oauth</em> method of the session controller; there you'll have access to the information you need thanks to the method <code>env["omniauth.auth"]</code>:</li>
</ul>

<div class="highlight highlight-ruby"><pre>(byebug) env[<span class="pl-s1"><span class="pl-pds">"</span>omniauth.auth<span class="pl-pds">"</span></span>].info
<span class="pl-c">#&lt;OmniAuth::AuthHash::InfoHash email="mluukkai@iki.fi" image="https://avatars.githubusercontent.com/u/523235?v=3" name="Matti Luukkainen" nickname="mluukkai" urls=#&lt;OmniAuth::AuthHash Blog=nil GitHub="https://github.com/mluukkai"&gt;&gt;</span>
(byebug)</pre></div>

<ul class="task-list">
<li>Make the require changes to your application</li>
<li>When your application works locally, change the GitHub application <em>homepage url</em> and the <em>authorization callback url</em> so that they correspond to the Heroku URLs of your application</li>
</ul>

The changes are not so straighfoward:

<ul class="task-list">
<li>The new method of the session controller should be written a code that checks the user identity and occasionally creates a <code>User</code> object corresponding to the GitHub user</li>
<li>You will have to modify the <code>User</code> model so that it can manage the users who sign-up in the system both with their own password and through GitHub</li>
<li>So far, the <code>User</code> objects validation requires that they have an at least four-character long password. You will have to make the validation optional so that it is not required for users who sign-on with GitHub IDs (google this for help); another option is generating a random password for users who sign on through GitHub</li>
<li>
The <code>User</code> objects validation requires that the username length is less than 15 characters. This may be a problem if you want to keep the GitHub usernames in your application, too. Change the username maximum length if required.</li>
</ul>
</blockquote>


<a id="user-content-nosql-tietokannat" class="anchor" href="#nosql-tietokannat" aria-hidden="true"><span class="octicon octicon-link"></span></a>NoSQL databases

Ralation databases have been dominating information storage for decades. Recently, we have started to see a new database fronteer, and the "non-relation databases" that are known under the term NoSQL have started to gain favour.

A motivation behind NoSQL databases has been that relation database are difficult to scale to the perfomance required by massive Internet applications. Also, the fact some NoSQL databases are schema-less makes applications more elastic than the carefully defined database schemes of SQL databases.

There are various NoSQL databases, and their operating mechanisms are completely different from each other, for instance

<ul class="task-list">
<li>key-value databases</li>
<li>document databases</li>
<li>columnar databases</li>
<li>graph databases</li>
</ul>

You already became familiar with <code>Rails.cache</code> which in fact is a simple key-value database, which enables storing an infinite amount of objects based on their key. The database search works only against objects key and the database does not support connections between objects at all.

Despite the growth of different database types, relation databases will survive still, and it is lukily that bigger application make use of various databases together, and they try to select the most suitable database type according to the storing reason, see
<a href="http://www.martinfowler.com/bliki/PolyglotPersistence.html">http://www.martinfowler.com/bliki/PolyglotPersistence.html</a>

<blockquote>

<a id="user-content-tehtävä-14" class="anchor" href="#teht%C3%A4v%C3%A4-14" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 14

The course is coming to an end, and it is time to give feedback at <a href="https://ilmo.cs.helsinki.fi/kurssit/servlet/Valinta">https://ilmo.cs.helsinki.fi/kurssit/servlet/Valinta</a>
</blockquote>


<a id="user-content-tehtävien-palautus" class="anchor" href="#teht%C3%A4vien-palautus" aria-hidden="true"><span class="octicon octicon-link"></span></a>Tehtävien palautus

Commit all the changes that you have done and push the code in Github. Deploy also the newest version into Heroku.

You should mark that you have returned the exercises at <a href="http://wadrorstats2015.herokuapp.com/">http://wadrorstats2015.herokuapp.com/</a>

<a id="user-content-mitä-seuraavaksi" class="anchor" href="#mit%C3%A4-seuraavaksi" aria-hidden="true"><span class="octicon octicon-link"></span></a>What next?

If Rails sounds interesting you could dig more into it in the following ways

<ul class="task-list">
<li>
<a href="https://leanpub.com/tr4w">https://leanpub.com/tr4w</a> The Book. You should <strong>definitely</strong> fetch it and read its every single line</li>
<li>
<a href="http://guides.rubyonrails.org/">http://guides.rubyonrails.org/</a> A lot of good stuff...</li>
<li>
<a href="http://railscasts.com/">http://railscasts.com/</a> excellent videos about singular topics. There haven't been new videos for more than one year, unfortunately, and we hope the Web site will become active again. Apparently, the newer not-for-free pro-episodes can be found from Youtube...</li>
<li>
<a href="https://www.ruby-toolbox.com/">https://www.ruby-toolbox.com/</a> some help to look for gems</li>
<li>
<a href="http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104">Eloquent Ruby</a> a great book about Ruby.</li>
</ul>
</article>
  </div>

</div>

<a href="#jump-to-line" rel="facebox[.linejump]" data-hotkey="l" style="display:none">Jump to Line</a>
<div id="jump-to-line" style="display:none">
  <form accept-charset="UTF-8" class="js-jump-to-line-form">
    <input class="linejump-input js-jump-to-line-field" type="text" placeholder="Jump to line&hellip;" autofocus>
    <button type="submit" class="button">Go</button>
  </form>
</div>

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
      <li><a href="http://training.github.com" data-ga-click="Footer, go to training, text:training">Training</a></li>
      <li><a href="http://shop.github.com" data-ga-click="Footer, go to shop, text:shop">Shop</a></li>
        <li><a href="/blog" data-ga-click="Footer, go to blog, text:blog">Blog</a></li>
        <li><a href="/about" data-ga-click="Footer, go to about, text:about">About</a></li>

    </ul>

    <a href="/" aria-label="Homepage">
      <span class="mega-octicon octicon-mark-github" title="GitHub"></span>
    </a>

    <ul class="site-footer-links">
      <li>&copy; 2015 <span title="0.05074s from github-fe121-cp1-prd.iad.github.net">GitHub</span>, Inc.</li>
        <li><a href="/site/terms" data-ga-click="Footer, go to terms, text:terms">Terms</a></li>
        <li><a href="/site/privacy" data-ga-click="Footer, go to privacy, text:privacy">Privacy</a></li>
        <li><a href="/security" data-ga-click="Footer, go to security, text:security">Security</a></li>
        <li><a href="/contact" data-ga-click="Footer, go to contact, text:contact">Contact</a></li>
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


      <script crossorigin="anonymous" src="https://assets-cdn.github.com/assets/frameworks-9643b0378c6bcb216adcdaaaa703eed77aa239aaf1c2ae44cadb2fb5099ec172.js"></script>
      <script async="async" crossorigin="anonymous" src="https://assets-cdn.github.com/assets/github-d967f968a967d73050b6f00df5ceb05917ff8f3c7f3803e832bee5eda8037365.js"></script>
      
      

  </body>
</html>

