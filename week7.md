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





 and defines a return call function, which is called by the browser when the GET request receives an answer from the server. The <code>beers</code> parameter of the return call function contains the data from the server, that is, the beers list in json form. The contents of the variable is placed in the global variable <code>oluet</code> and the beer list length is shown is shown on the Web page in the element with the ID beers.

Because you placed a reference to a global variable of the list, you can check it manually from the browser console (remember that the console can be opened in Chrome from the tools tab, or pressing c ctrl, shift, j (in Linux) or alt, cmd, i (in Mac) ja jos olit jo sulkenut konsolin teit pahan virheen. <strong>Konsoli tulee pitää aina auki javascriptillä ohjelmoitaessa!</strong>):

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-2.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-2.png" alt="kuva" style="max-width:100%;"></a>

Once it has fetched the beers from the browser, the return call function should form the HTML code that lists them and add it to the page.

Change the javascript code now to list only the beer names at first:

<div class="highlight highlight-javascript"><pre>    $.getJSON(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">beers</span>) {
        <span class="pl-s">var</span> beer_list <span class="pl-k">=</span> [];

        $.each(beers, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
            beer_list.<span class="pl-s3">push</span>(<span class="pl-s1"><span class="pl-pds">'</span>&lt;li&gt;<span class="pl-pds">'</span></span> <span class="pl-k">+</span> beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>] <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/li&gt;<span class="pl-pds">'</span></span>)
        });

        $(<span class="pl-s1"><span class="pl-pds">"</span>#beers<span class="pl-pds">"</span></span>).html(<span class="pl-s1"><span class="pl-pds">'</span>&lt;ul&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span> beer_list.<span class="pl-s3">join</span>(<span class="pl-s1"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>) <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/ul&gt;<span class="pl-pds">'</span></span>);
     });</pre></div>

The code defines the local table variable <code>beer_list</code> and goes through the list <code>beers</code> that it received as parameter (and it does this using the jquery <code>each</code> method which is quite particular, see<a href="http://api.jquery.com/jQuery.each/">http://api.jquery.com/jQuery.each/</a>). Then it adds an HTML element to <code>beer_list</code> for each beer. The HTML element looks like

<div class="highlight highlight-erb"><pre>    &lt;<span class="pl-ent">li</span>&gt;Karhu tuplahumala&lt;/<span class="pl-ent">li</span>&gt;</pre></div>

At the end, ul tags are added to the beginning and the end of the list, and the list elements are joined with the join method. The HTML code which originates is joined to the element with the <code>beers</code> ID.

In this way you have got a simple list of the names of the beers on the page.

What if you wanted to sort the beers? To make this happen, refactor first the code as follows:

<div class="highlight highlight-javascript"><pre><span class="pl-s">var</span> BEERS <span class="pl-k">=</span> {};

<span class="pl-s3">BEERS</span>.<span class="pl-en">show</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    <span class="pl-s">var</span> beer_list <span class="pl-k">=</span> [];

    $.each(BEERS.list, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
        beer_list.<span class="pl-s3">push</span>(<span class="pl-s1"><span class="pl-pds">'</span>&lt;li&gt;<span class="pl-pds">'</span></span> <span class="pl-k">+</span> beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>] <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/li&gt;<span class="pl-pds">'</span></span>)
    });

    $(<span class="pl-s1"><span class="pl-pds">"</span>#beers<span class="pl-pds">"</span></span>).html(<span class="pl-s1"><span class="pl-pds">'</span>&lt;ul&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span> beer_list.<span class="pl-s3">join</span>(<span class="pl-s1"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>) <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/ul&gt;<span class="pl-pds">'</span></span>);
};

$(<span class="pl-s3">document</span>).ready(<span class="pl-st">function</span> () {

    $.getJSON(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">beers</span>) {
        BEERS.list <span class="pl-k">=</span> beers;
        BEERS.show();
     });
});</pre></div>

You defined now the object <code>BEERS</code>, that receives the beer list from the server in the attribute <code>BEERS.list</code>. The method <code>BEERS.show</code> creates an HTML table of <code>BEERS.list</code> objects and places them in the view.

In this way, the beer list from the server remains "in memory" in the browser variable <code>BEERS.list</code> and the list can be resorted when needed, and it can be shown to users in new orders without that the Web page needed to communicate with the server.

Add a button (or a link) to the page, allowing to make the beers on the page in discending order:

<div class="highlight highlight-erb"><pre>&lt;<span class="pl-ent">a</span> <span class="pl-e">href</span>=<span class="pl-s1"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>reverse<span class="pl-pds">"</span></span>&gt;reverse!&lt;/<span class="pl-ent">a</span>&gt;
&lt;<span class="pl-ent">div</span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>beers<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;</pre></div>

Then add a click handler to the link in javascript, to sort the beers in descending order when the link is clicked and to show them in the beers elements in the page:

<div class="highlight highlight-javascript"><pre><span class="pl-s">var</span> BEERS <span class="pl-k">=</span> {};

<span class="pl-s3">BEERS</span>.<span class="pl-en">show</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    <span class="pl-s">var</span> beer_list <span class="pl-k">=</span> [];

    $.each(BEERS.list, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
        beer_list.<span class="pl-s3">push</span>(<span class="pl-s1"><span class="pl-pds">'</span>&lt;li&gt;<span class="pl-pds">'</span></span> <span class="pl-k">+</span> beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>] <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/li&gt;<span class="pl-pds">'</span></span>)
    });

    $(<span class="pl-s1"><span class="pl-pds">"</span>#beers<span class="pl-pds">"</span></span>).html(<span class="pl-s1"><span class="pl-pds">'</span>&lt;ul&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span> beer_list.<span class="pl-s3">join</span>(<span class="pl-s1"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>) <span class="pl-k">+</span> <span class="pl-s1"><span class="pl-pds">'</span>&lt;/ul&gt;<span class="pl-pds">'</span></span>);
};

<span class="pl-s3">BEERS</span>.<span class="pl-en">reverse</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    BEERS.list.<span class="pl-s3">reverse</span>();
};

$(<span class="pl-s3">document</span>).ready(<span class="pl-st">function</span> () {
    $(<span class="pl-s1"><span class="pl-pds">"</span>#reverse<span class="pl-pds">"</span></span>).<span class="pl-s3">click</span>(<span class="pl-st">function</span> (<span class="pl-vpf">e</span>) {
        BEERS.<span class="pl-s3">reverse</span>();
        BEERS.show();
        e.preventDefault();
    });

    $.getJSON(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">beers</span>) {
        BEERS.list <span class="pl-k">=</span> beers;
        BEERS.show();
    });
});</pre></div>

The link click handler is defined within the <code>document ready</code> event, so when the document has been loaded, the click handler function is registered for the link element which has "reverse" as ID. The browser calls the function defined when the link is clicked. At last, the command <code>preventDefault</code> prevents the "normal" functionality – that is, accessing a (now inexistent) link.

You'll now understand the basics well enough to implement the real functionality.

Change the view as follows:

<div class="highlight highlight-erb"><pre>&lt;<span class="pl-ent">h2</span>&gt;Beers&lt;/<span class="pl-ent">h2</span>&gt;

&lt;<span class="pl-ent">table</span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>beertable<span class="pl-pds">"</span></span> <span class="pl-e">class</span>=<span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">tr</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span> <span class="pl-e">href</span>=<span class="pl-s1"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span>&gt;Name&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span> <span class="pl-e">href</span>=<span class="pl-s1"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>style<span class="pl-pds">"</span></span>&gt;Style&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span> <span class="pl-e">href</span>=<span class="pl-s1"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-e">id</span>=<span class="pl-s1"><span class="pl-pds">"</span>brewery<span class="pl-pds">"</span></span>&gt;Brewery&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
  &lt;/<span class="pl-ent">tr</span>&gt;

&lt;/<span class="pl-ent">table</span>&gt;</pre></div>

So the three column names have become a link to register the click listeners. The table was given the ID <code>beertable</code>.

Change the <code>show</code> method defined in the javascript so that it adds the beer names to the table:

<div class="highlight highlight-javascript"><pre><span class="pl-s3">BEERS</span>.<span class="pl-en">show</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    <span class="pl-s">var</span> table <span class="pl-k">=</span> $(<span class="pl-s1"><span class="pl-pds">"</span>#beertable<span class="pl-pds">"</span></span>);

    $.each(BEERS.list, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
        table.append(<span class="pl-s1"><span class="pl-pds">'</span>&lt;tr&gt;&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;&lt;/tr&gt;<span class="pl-pds">'</span></span>);
    });
};</pre></div>

Eli ensin koodi tallettaa viitteen taulukkoon muuttujana <code>table</code> ja lisää sinne <code>append</code>-komennolla uuden rivin kutakin olutta varten.

Laajennetaan sitten metodia näyttämään kaikki tiedot oluista. Huomaamme kuitenkin, että oluiden json-muotoisessa listassa ei ole panimosta muuta tietoa kuin olioiden id:t, haluaisimme kuitenkin näyttää panimon nimen. Oluttyylin tiedot löytyvät kokonaisuudessaan jsonista jo nyt.

Ongelma on onneksi helppo ratkaista muokkaamalla oluiden listan tuottavaa json-jbuildertemplatea. Template näyttää nyt seuraavalta:

<div class="highlight highlight-ruby"><pre>json.array!(<span class="pl-vo">@beers</span>) <span class="pl-k">do </span>|<span class="pl-vo">beer</span>|
  json.extract! beer, <span class="pl-c1">:id</span>, <span class="pl-c1">:name</span>, <span class="pl-c1">:style</span>, <span class="pl-c1">:brewery_id</span>
  json.url beer_url(beer, <span class="pl-c1">format:</span> <span class="pl-c1">:json</span>)
<span class="pl-k">end</span></pre></div>

The template defines that the fields <em>id</em>, <em>name</em> and <em>brewery_id</em> as well as <em>style</em> should be contained in the json form for each beer; also, style refers to the <code>Style</code> object of the beer. The style object has to be rendered completely in the json form of the beer. You will also get the brewery json form in the same of the beer json form if you replace <em>brewery_id</em> with <em>brewery</em>, in the template. So change the template form:

<div class="highlight highlight-ruby"><pre>json.array!(<span class="pl-vo">@beers</span>) <span class="pl-k">do </span>|<span class="pl-vo">beer</span>|
  json.extract! beer, <span class="pl-c1">:id</span>, <span class="pl-c1">:name</span>, <span class="pl-c1">:style</span>, <span class="pl-c1">:brewery</span>
<span class="pl-k">end</span></pre></div>

the last line was deleted, which would have added an URL to the beer own json form – in the same way as for each beer json form.

Now the table can be generated in the following way in Javascript:

<div class="highlight highlight-javascript"><pre><span class="pl-s3">BEERS</span>.<span class="pl-en">show</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    <span class="pl-s">var</span> table <span class="pl-k">=</span> $(<span class="pl-s1"><span class="pl-pds">"</span>#beertable<span class="pl-pds">"</span></span>);

    $.each(BEERS.list, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
        table.append(<span class="pl-s1"><span class="pl-pds">'</span>&lt;tr&gt;<span class="pl-pds">'</span></span>
                        <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
                        <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>style<span class="pl-pds">'</span></span>][<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
                        <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>brewery<span class="pl-pds">'</span></span>][<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
                    <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/tr&gt;<span class="pl-pds">'</span></span>);
    });
};</pre></div>

The beers list in json form will contain a lot useless information, because at the same time, the brewery json forms of each beer brewery and style are rendered completely. You could optimize the template so that the beer brewery and style would follow the json form only as far as their name is concerned:

<div class="highlight highlight-ruby"><pre>json.array!(<span class="pl-vo">@beers</span>) <span class="pl-k">do </span>|<span class="pl-vo">beer</span>|
  json.extract! beer, <span class="pl-c1">:id</span>, <span class="pl-c1">:name</span>
  json.style <span class="pl-k">do</span>
    json.name beer.style.name
  <span class="pl-k">end</span>
  json.brewery <span class="pl-k">do</span>
    json.name beer.brewery.name
  <span class="pl-k">end</span>
<span class="pl-k">end</span></pre></div>

Now the json-form list sent by the server is much more compact:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-5.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w7-5.png" alt="kuva" style="max-width:100%;"></a>

The links have to be set up now with the event listeners that execute the sorting (you find the final javascript code below):

<div class="highlight highlight-javascript"><pre><span class="pl-s">var</span> BEERS <span class="pl-k">=</span> {};

<span class="pl-s3">BEERS</span>.<span class="pl-en">show</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    $(<span class="pl-s1"><span class="pl-pds">"</span>#beertable tr:gt(0)<span class="pl-pds">"</span></span>).<span class="pl-s3">remove</span>();

    <span class="pl-s">var</span> table <span class="pl-k">=</span> $(<span class="pl-s1"><span class="pl-pds">"</span>#beertable<span class="pl-pds">"</span></span>);

    $.each(BEERS.list, <span class="pl-st">function</span> (<span class="pl-vpf">index</span>, <span class="pl-vpf">beer</span>) {
        table.append(<span class="pl-s1"><span class="pl-pds">'</span>&lt;tr&gt;<span class="pl-pds">'</span></span>
            <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
            <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>style<span class="pl-pds">'</span></span>][<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
            <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;td&gt;<span class="pl-pds">'</span></span><span class="pl-k">+</span>beer[<span class="pl-s1"><span class="pl-pds">'</span>brewery<span class="pl-pds">'</span></span>][<span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>]<span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/td&gt;<span class="pl-pds">'</span></span>
            <span class="pl-k">+</span><span class="pl-s1"><span class="pl-pds">'</span>&lt;/tr&gt;<span class="pl-pds">'</span></span>);
    });
};

<span class="pl-s3">BEERS</span>.<span class="pl-en">sort_by_name</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    BEERS.list.<span class="pl-s3">sort</span>( <span class="pl-st">function</span>(<span class="pl-vpf">a</span>,<span class="pl-vpf">b</span>){
        <span class="pl-k">return</span> a.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>().localeCompare(b.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>());
    });
};

<span class="pl-s3">BEERS</span>.<span class="pl-en">sort_by_style</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    BEERS.list.<span class="pl-s3">sort</span>( <span class="pl-st">function</span>(<span class="pl-vpf">a</span>,<span class="pl-vpf">b</span>){
        <span class="pl-k">return</span> a.<span class="pl-sc">style</span>.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>().localeCompare(b.<span class="pl-sc">style</span>.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>());
    });
};

<span class="pl-s3">BEERS</span>.<span class="pl-en">sort_by_brewery</span> <span class="pl-k">=</span> <span class="pl-st">function</span>(){
    BEERS.list.<span class="pl-s3">sort</span>( <span class="pl-st">function</span>(<span class="pl-vpf">a</span>,<span class="pl-vpf">b</span>){
        <span class="pl-k">return</span> a.brewery.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>().localeCompare(b.brewery.<span class="pl-sc">name</span>.<span class="pl-s3">toUpperCase</span>());
    });
};

$(<span class="pl-s3">document</span>).ready(<span class="pl-st">function</span> () {
    $(<span class="pl-s1"><span class="pl-pds">"</span>#name<span class="pl-pds">"</span></span>).<span class="pl-s3">click</span>(<span class="pl-st">function</span> (<span class="pl-vpf">e</span>) {
        BEERS.sort_by_name();
        BEERS.show();
        e.preventDefault();
    });

    $(<span class="pl-s1"><span class="pl-pds">"</span>#style<span class="pl-pds">"</span></span>).<span class="pl-s3">click</span>(<span class="pl-st">function</span> (<span class="pl-vpf">e</span>) {
        BEERS.sort_by_style();
        BEERS.show();
        e.preventDefault();
    });

    $(<span class="pl-s1"><span class="pl-pds">"</span>#brewery<span class="pl-pds">"</span></span>).<span class="pl-s3">click</span>(<span class="pl-st">function</span> (<span class="pl-vpf">e</span>) {
        BEERS.sort_by_brewery();
        BEERS.show();
        e.preventDefault();
    });

    $.getJSON(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">beers</span>) {
        BEERS.list <span class="pl-k">=</span> beers;
        BEERS.sort_by_name();
        BEERS.show();
    });

});</pre></div>

Your Javascript code is linked to each application page. The unfortunate result is that wherever you may be on your site, the Javascript will load the beers list due to the command <code>getJSON('beers.json', ...) </code>. Refine your Javascript code so that the beers list is loaded only if you are on a page with the table <code>beertable</code>:

<div class="highlight highlight-javascript"><pre>    <span class="pl-k">if</span> ( $(<span class="pl-s1"><span class="pl-pds">"</span>#beertable<span class="pl-pds">"</span></span>).<span class="pl-sc">length</span><span class="pl-k">&gt;</span><span class="pl-c1">0</span> ) {
      $.getJSON(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">beers</span>) {
        BEERS.list <span class="pl-k">=</span> beers;
        BEERS.sort_by_name;
        BEERS.show();
      });
    }</pre></div>

The current trend tells us we should move and more Web pages functionality to the browser. The advantage is that you'll make your Web applications remind more and more of desktop applications.


<a id="user-content-angularjs" class="anchor" href="#angularjs" aria-hidden="true"><span class="octicon octicon-link"></span></a>AngularJS

The code of the page you implemented with Javascript to list the beers was not wide. However, if compared to Rails fluency and effortless coding style, what you wrote was quite heavy and full of annoying and rutine-like details, at times.

Have a look next at how you could implement the same functionality making use of <a href="http://angularjs.org/">AngularJS</a> an MVC application (or MVW, that is, Model View Whatever... as Angular's developer has stated himself) that has gained quite much support lately. Angular is fluent and magical just like Rails, and in fact, many members of Rails communities have been moving to write applications more and more on browser side with Angular.

Make a new route to the file routes.rb for your functionality:

<pre><code>get 'ngbeerlist', to:'beers#nglist'
</code></pre>

and an empty controller method:

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">BeersController<span class="pl-e"> &lt; ApplicationController</span></span>
  <span class="pl-c"># muut before_actionit säilyvät ennallaan</span>
  before_action <span class="pl-c1">:ensure_that_signed_in</span>, <span class="pl-c1">except:</span> [<span class="pl-c1">:index</span>, <span class="pl-c1">:show</span>, <span class="pl-c1">:list</span>, <span class="pl-c1">:nglist</span>]

  <span class="pl-k">def</span> <span class="pl-en">nglist</span>
  <span class="pl-k">end</span></pre></div>

Make a Javascript code this time, and instead of placing it in a view template, put it in a script tag (this is in no way a best practice!) The code can be moved to its own file if you want.

The first version of the view app/views/beers/nglist.html.erb is below:

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>script src<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.13/angular.min.js<span class="pl-pds">"</span></span><span class="pl-k">&gt;&lt;</span>/script<span class="pl-k">&gt;</span>
<span class="pl-k">&lt;</span>script<span class="pl-k">&gt;</span>
  <span class="pl-s">var</span> myApp <span class="pl-k">=</span> angular.module(<span class="pl-s1"><span class="pl-pds">'</span>myApp<span class="pl-pds">'</span></span>, []);

  myApp.controller(<span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">$scope</span>, <span class="pl-vpf">$http</span>) {
    $scope.teksti <span class="pl-k">=</span> <span class="pl-s1"><span class="pl-pds">"</span>Hello Angular!<span class="pl-pds">"</span></span>
  });
<span class="pl-k">&lt;</span>/script<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>h2<span class="pl-k">&gt;</span>Beers<span class="pl-k">&lt;</span>/h2<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>div ng<span class="pl-k">-</span>app<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>myApp<span class="pl-pds">"</span></span> ng<span class="pl-k">-</span>controller<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>

  {{teksti}}

<span class="pl-k">&lt;</span>/div<span class="pl-k">&gt;</span></pre></div>

The Javascript tag in the script code creates first the Angular module <em>myApp</em> and then defines the module controller function <em>BeersController</em>.

The div tag on the page defines that it is an <code>ng-app</code>, that is, an Angular application, which is defined by the Angular module <em>"myApp</em>. The HTML code inside div tag is defined to manage the controller method <code>BeersController</code>.

You will see that the controller method sets the string "Hellow Angular!" as value of the field <code>teksti</code> of the variable <code>$scope</code>. All fields of the variable <code>$scope</code> can be used from the view which is managed by the controller. The view contains the peculiar descriptor <code>{{teksti}}</code>. When the browser renders the page for users, the Angular code between the double curly brakets is executed, and the result is rendered on the screen. So the screen renders the string contained in the controller variable.

Change your code as below:

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>script src<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.13/angular.min.js<span class="pl-pds">"</span></span><span class="pl-k">&gt;&lt;</span>/script<span class="pl-k">&gt;</span>
<span class="pl-k">&lt;</span>script<span class="pl-k">&gt;</span>
  <span class="pl-s">var</span> myApp <span class="pl-k">=</span> angular.module(<span class="pl-s1"><span class="pl-pds">'</span>myApp<span class="pl-pds">'</span></span>, []);

  myApp.controller(<span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">$scope</span>, <span class="pl-vpf">$http</span>) {
    $scope.beers <span class="pl-k">=</span> [{<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-c1">6</span>,<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Hefeweizen<span class="pl-pds">"</span></span>,<span class="pl-s1"><span class="pl-pds">"</span>style<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Weizen<span class="pl-pds">"</span></span>},<span class="pl-s1"><span class="pl-pds">"</span>brewery<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Weihenstephaner<span class="pl-pds">"</span></span>}},{<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-c1">7</span>,<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Helles<span class="pl-pds">"</span></span>,<span class="pl-s1"><span class="pl-pds">"</span>style<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>European pale lager<span class="pl-pds">"</span></span>},<span class="pl-s1"><span class="pl-pds">"</span>brewery<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Weihenstephaner<span class="pl-pds">"</span></span>}},{<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-c1">4</span>,<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Huvila Pale Ale<span class="pl-pds">"</span></span>,<span class="pl-s1"><span class="pl-pds">"</span>style<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>American pale ale<span class="pl-pds">"</span></span>},<span class="pl-s1"><span class="pl-pds">"</span>brewery<span class="pl-pds">"</span></span><span class="pl-k">:</span>{<span class="pl-s1"><span class="pl-pds">"</span>name<span class="pl-pds">"</span></span><span class="pl-k">:</span><span class="pl-s1"><span class="pl-pds">"</span>Malmgard<span class="pl-pds">"</span></span>}}];
  });
<span class="pl-k">&lt;</span>/script<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>h2<span class="pl-k">&gt;</span>Beers<span class="pl-k">&lt;</span>/h2<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>div ng<span class="pl-k">-</span>app<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>myApp<span class="pl-pds">"</span></span> ng<span class="pl-k">-</span>controller<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>

  <span class="pl-k">&lt;</span>ul<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>li ng<span class="pl-k">-</span>repeat<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>beer in beers<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
      {{beer.<span class="pl-sc">name</span>}} brewed by {{beer.brewery.<span class="pl-sc">name</span>}}
    <span class="pl-k">&lt;</span>/li<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span>/ul<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>/div<span class="pl-k">&gt;</span></pre></div>

Now the list of beers and their breweries will render in the browser. The core of the code is like below

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>li ng<span class="pl-k">-</span>repeat<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>beer in beers<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
   {{beer.<span class="pl-sc">name</span>}} brewed by {{beer.brewery.<span class="pl-sc">name</span>}}
<span class="pl-k">&lt;</span>/li<span class="pl-k">&gt;</span></pre></div>

Angular's <em>ng-repeat</em> command (or directive) creates their own li tag for each item of the <code>beers</code> table which was set up by the controller to scope. As you'll see the difference from Rails view templates is not much. The biggest difference is of course that everything happens in the browser.

With the help of Angular, is very easy to retrive json data from the server. You only have to change the controller function in this way:

<div class="highlight highlight-javascript"><pre>  myApp.controller(<span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">$scope</span>, <span class="pl-vpf">$http</span>) {
    $http.get(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>).success( <span class="pl-st">function</span>(<span class="pl-vpf">data</span>, <span class="pl-vpf">status</span>, <span class="pl-vpf">headers</span>, <span class="pl-vpf">config</span>) {
      $scope.beers <span class="pl-k">=</span> data;
    });
  });</pre></div>

<code>$http.get</code> obviously makes a GET call to the address in parameter. The rendered return call function sets up the server data to the scope variable <code>beers</code>.

Change now the view template so that beers are not shown in a list but in a table:

<div class="highlight highlight-html"><pre>&lt;<span class="pl-ent">div</span> <span class="pl-e">ng-app</span>=<span class="pl-s1"><span class="pl-pds">"</span>myApp<span class="pl-pds">"</span></span> <span class="pl-e">ng-controller</span>=<span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span>&gt;

  &lt;<span class="pl-ent">table</span> <span class="pl-e">class</span>=<span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">thead</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span>&gt;name&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span>&gt;style&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
    &lt;<span class="pl-ent">th</span>&gt; &lt;<span class="pl-ent">a</span>&gt;brewery&lt;/<span class="pl-ent">a</span>&gt; &lt;/<span class="pl-ent">th</span>&gt;
    &lt;/<span class="pl-ent">thead</span>&gt;
    &lt;<span class="pl-ent">tr</span> <span class="pl-e">ng-repeat</span>=<span class="pl-s1"><span class="pl-pds">"</span>beer in beers<span class="pl-pds">"</span></span>&gt;
      &lt;<span class="pl-ent">td</span>&gt;{{beer.name}}&lt;/<span class="pl-ent">td</span>&gt;
      &lt;<span class="pl-ent">td</span>&gt;{{beer.style.name}}&lt;/<span class="pl-ent">td</span>&gt;
      &lt;<span class="pl-ent">td</span>&gt;{{beer.brewery.name}}&lt;/<span class="pl-ent">td</span>&gt;
    &lt;/<span class="pl-ent">tr</span>&gt;
  &lt;/<span class="pl-ent">table</span>&gt;

&lt;/<span class="pl-ent">div</span>&gt;</pre></div>

Also add titles for sorting the table which should be within a tags, not as links. Add finally the magic which will take care of the sorting operation:

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>script src<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.13/angular.min.js<span class="pl-pds">"</span></span><span class="pl-k">&gt;&lt;</span>/script<span class="pl-k">&gt;</span>
<span class="pl-k">&lt;</span>script<span class="pl-k">&gt;</span>
    <span class="pl-st">function</span> <span class="pl-en">BeersController</span>(<span class="pl-vpf">$scope</span>, <span class="pl-vpf">$http</span>) {
        $http.get(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>).success( <span class="pl-st">function</span>(<span class="pl-vpf">data</span>, <span class="pl-vpf">status</span>, <span class="pl-vpf">headers</span>, <span class="pl-vpf">config</span>) {
            $scope.beers <span class="pl-k">=</span> data;
        });

        $scope.order <span class="pl-k">=</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>;

        <span class="pl-s3">$scope</span>.<span class="pl-en">sort_by</span> <span class="pl-k">=</span> <span class="pl-st">function</span> (<span class="pl-vpf">order</span>){
            $scope.order <span class="pl-k">=</span> order;
            <span class="pl-en">console</span><span class="pl-s3">.log</span>(order);
        }
    }
<span class="pl-k">&lt;</span>/script<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>h2<span class="pl-k">&gt;</span>Beers<span class="pl-k">&lt;</span>/h2<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>div ng<span class="pl-k">-</span>app<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>myApp<span class="pl-pds">"</span></span> ng<span class="pl-k">-</span>controller<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>

  <span class="pl-k">&lt;</span>table <span class="pl-st">class</span><span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>thead<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>name<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('style.name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>style<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('brewery.name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>brewery<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>/thead<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>tr ng<span class="pl-k">-</span>repeat<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>beer in beers| orderBy:order<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.<span class="pl-sc">style</span>.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.brewery.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>/tr<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span>/table<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>/div<span class="pl-k">&gt;</span></pre></div>

This added a couple of things to your code. You defined the method <code>sort_by</code> to scope; clicking it helps defining the value of the variable <code>order</code> which was added to scope. The default value is 'name,' but if you click on the brewery title, the value becomes 'brewery.name,' and so if you click on the title of the style column, 'style.name' will be picked as value. The following line was added to ng repeat:

<div class="highlight highlight-javascript"><pre>ng<span class="pl-k">-</span>repeat<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>beer in beers| orderBy:order<span class="pl-pds">"</span></span></pre></div>

The collection <code>beers</code> is contained in the variable, and it is sorted through the <code>orderBy</code> filter against the <code>order</code> value of the variable.

In addition, implement now the functionality to filters the beers shown in the list:

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>script<span class="pl-k">&gt;</span>
  <span class="pl-s">var</span> myApp <span class="pl-k">=</span> angular.module(<span class="pl-s1"><span class="pl-pds">'</span>myApp<span class="pl-pds">'</span></span>, []);

  myApp.controller(<span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span>, <span class="pl-st">function</span> (<span class="pl-vpf">$scope</span>, <span class="pl-vpf">$http</span>) {
    $http.get(<span class="pl-s1"><span class="pl-pds">'</span>beers.json<span class="pl-pds">'</span></span>).success( <span class="pl-st">function</span>(<span class="pl-vpf">data</span>, <span class="pl-vpf">status</span>, <span class="pl-vpf">headers</span>, <span class="pl-vpf">config</span>) {
      $scope.beers <span class="pl-k">=</span> data;
    });

    $scope.order <span class="pl-k">=</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>;

    <span class="pl-s3">$scope</span>.<span class="pl-en">sort_by</span> <span class="pl-k">=</span> <span class="pl-st">function</span> (<span class="pl-vpf">order</span>){
      $scope.order <span class="pl-k">=</span> order;
      <span class="pl-en">console</span><span class="pl-s3">.log</span>(order);
    }

    $scope.searchText <span class="pl-k">=</span> <span class="pl-s1"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>;
  });
<span class="pl-k">&lt;</span>/script<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>h2<span class="pl-k">&gt;</span>Beers<span class="pl-k">&lt;</span>/h2<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>div ng<span class="pl-k">-</span>app<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>myApp<span class="pl-pds">"</span></span> ng<span class="pl-k">-</span>controller<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>BeersController<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>

  search<span class="pl-k">:</span>  <span class="pl-k">&lt;</span>input ng<span class="pl-k">-</span>model<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>searchText<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>

  <span class="pl-k">&lt;</span>table <span class="pl-st">class</span><span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>thead<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>name<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('style.name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>style<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>a ng<span class="pl-k">-</span>click<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>sort_by('brewery.name')<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>brewery<span class="pl-k">&lt;</span>/a<span class="pl-k">&gt;</span> <span class="pl-k">&lt;</span>/th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>/thead<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>tr ng<span class="pl-k">-</span>repeat<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>beer in beers| orderBy:order | filter:searchText<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.<span class="pl-sc">style</span>.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>{{beer.brewery.<span class="pl-sc">name</span>}}<span class="pl-k">&lt;</span>/td<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>/tr<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span>/table<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>/div<span class="pl-k">&gt;</span></pre></div>

A new filter was added to ng repeat, the so-called <code>filter</code>, which filters the collection beers and showing only the ones that comply with the string in the scope variable <code>searchText</code> in the filter parameter.

The string restricting the search is input with an input tag:

<div class="highlight highlight-javascript"><pre><span class="pl-k">&lt;</span>input ng<span class="pl-k">-</span>model<span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>searchText<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span></pre></div>

As you'll see, Angular magic updates the value of the <code>searchText</code> variable in scope while you write the text in the search box.

If you happened to be interested in the topic, you may read more here:
<a href="http://docs.angularjs.org/tutorial">http://docs.angularjs.org/tutorial</a>

<a href="https://github.com/tuhoojabotti/AngularJS-ohjelmointiprojekti-k2014/blob/master/material/aloitusluento.md">A tutorial has been written for Angular hash</a>, which you may find useful too. The tutorial makes you use Angular to build a page that uses Rails in its backend.

It looks like Angular is taking over the scene extremely quickly. Angular is like ice hockey: easy to learn, but there is wide scope for development and masterng it requires hard work. Luckily, using Angular is not like an either-everything-or-nothing solution. There is nothing that prevents you from  using Angular carefully, for instance, refining a Web site with Angular partially, and using Rails templates to produce the rest of the contents on the server side. This is what has been done on the course page with the homework statistics <a href="http://wadrorstats2015.herokuapp.com/">http://wadrorstats2015.herokuapp.com/</a>, where the charts are designed with Angular and GoogleCharts.

There is no standardized way that defines how a Rails-AngularJS project should be organised. Good insights on the issue are provided at
<a href="http://rockyj.in/2013/10/24/angular_rails.html">http://rockyj.in/2013/10/24/angular_rails.html</a>

<blockquote>

<a id="user-content-tehtävä-3" class="anchor" href="#teht%C3%A4v%C3%A4-3" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 3

Follow the examples above, and use Javascript (JQuery or AngularJS) to implement the page listing all breweries http:localhost:3000/brewerylist so that they can be sorted either in alphabetic order or against their founding year, as well as against the number of beers they produced. The page <strong>does not</strong> need to separate the expired breweries in their own table.

Remember to keep the Javascript console open the whole time while you proceed with the exercise! You can the debug by printing the Javascript in the console with the command <code>console.log()</code>

<strong>ATTENTION:</strong> due to the changes you did last week, the breweries json list http://localhost:3000/breweries.json does not work, because the breweries#index controller is not given the list of all breweries in the variable <code>@breweries</code> any more. Fix the situation.
</blockquote>


<a id="user-content-selainpuolella-toteutetun-toiminnallisuuden-testaaminen" class="anchor" href="#selainpuolella-toteutetun-toiminnallisuuden-testaaminen" aria-hidden="true"><span class="octicon octicon-link"></span></a>Selainpuolella toteutetun toiminnallisuuden testaaminen

Make some tests with rspec/capybara for the Javascript beers list. Your starting point is the following file, spec/features/beerlist_spec.rb:

<div class="highlight highlight-ruby"><pre><span class="pl-k">require</span> <span class="pl-s1"><span class="pl-pds">'</span>rails_helper<span class="pl-pds">'</span></span>

describe <span class="pl-s1"><span class="pl-pds">"</span>Beerlist page<span class="pl-pds">"</span></span> <span class="pl-k">do</span>
  before <span class="pl-c1">:each</span> <span class="pl-k">do</span>
    <span class="pl-vo">@brewery1</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Koff<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@brewery2</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Schlenkerla<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@brewery3</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Ayinger<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@style1</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Lager<span class="pl-pds">"</span></span>
    <span class="pl-vo">@style2</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Rauchbier<span class="pl-pds">"</span></span>
    <span class="pl-vo">@style3</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Weizen<span class="pl-pds">"</span></span>
    <span class="pl-vo">@beer1</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span> <span class="pl-vo">@brewery1</span>, <span class="pl-c1">style:</span><span class="pl-vo">@style1</span>)
    <span class="pl-vo">@beer2</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Fastenbier<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span><span class="pl-vo">@brewery2</span>, <span class="pl-c1">style:</span><span class="pl-vo">@style2</span>)
    <span class="pl-vo">@beer3</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span><span class="pl-s1"><span class="pl-pds">"</span>Lechte Weisse<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span><span class="pl-vo">@brewery3</span>, <span class="pl-c1">style:</span><span class="pl-vo">@style3</span>)
  <span class="pl-k">end</span>

  it <span class="pl-s1"><span class="pl-pds">"</span>shows one known beer<span class="pl-pds">"</span></span> <span class="pl-k">do</span>
    visit beerlist_path
    expect(page).to have_content <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>
  <span class="pl-k">end</span>
<span class="pl-k">end</span></pre></div>

Execute the test with the command <code>rspec spec/features/beerlist_spec.rb</code>. You will receive an error message anyway:

<div class="highlight highlight-ruby"><pre>  <span class="pl-c1">1</span>) beerlist page <span class="pl-vo">Beerlist</span> page shows one known beer
     <span class="pl-vo">Failure</span><span class="pl-k">/</span><span class="pl-c1">Error:</span> expect(page).to have_content <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>
       expected to find text <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span> <span class="pl-k">in</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries beers styles ratings users clubs places | signin signup Beers Name Style Brewery<span class="pl-pds">"</span></span></pre></div>

It looks like that the page does not contain any beers list at all. Check this out with the command <code>save_and_open_page</code> that you should put right before the test, which will open the browser page where capybara has navigated to
(see <a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#capybarav4#capybara">https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko4.md#capybarav4#capybara</a>).

And the beer table to show on the page is empty as expected:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-2.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-2.png" alt="kuva" style="max-width:100%;"></a>

You find the reason for this from Capybara documentation <a href="https://github.com/jnicklas/capybara#drivers">https://github.com/jnicklas/capybara#drivers</a>

<blockquote>
By default, Capybara uses the :rack_test driver, which is fast but limited: it does not support JavaScript, nor is it able to access HTTP resources outside of your Rack application, such as remote APIs and OAuth services. To get around these limitations, you can set up a different default driver for your features.
</blockquote>

Fixing this is simple, too. The Javascript tests only need to be added a parameter and they will be executed with the help of Selenium, a test machine which knows Javascript:

<div class="highlight highlight-ruby"><pre>    it <span class="pl-s1"><span class="pl-pds">"</span>shows the known beers<span class="pl-pds">"</span></span>, <span class="pl-c1">js:</span><span class="pl-c1">true</span> <span class="pl-k">do</span></pre></div>

In order to start using Selenium, the Gemfile test scope has to be added the following gem:

<pre><code>gem 'selenium-webdriver'
</code></pre>

Execute <code>bundle install</code>, and run the tests. You'll run in an error message again:

<div class="highlight highlight-ruby"><pre>     <span class="pl-vo">Failure</span><span class="pl-k">/</span><span class="pl-c1">Error:</span> visit beerlist_path
     <span class="pl-s3">WebMock</span>::<span class="pl-c1">NetConnectNotAllowedError:</span>
       <span class="pl-vo">Real</span> <span class="pl-vo">HTTP</span> connections are disabled. <span class="pl-vo">Unregistered</span> <span class="pl-c1">request:</span> <span class="pl-vo">GET</span> <span class="pl-c1">http:</span><span class="pl-sr"><span class="pl-pds">//</span></span><span class="pl-c1">127.0</span>.<span class="pl-c1">0.1</span>:<span class="pl-c1">60873</span><span class="pl-k">/</span>__identify__ with headers {<span class="pl-s1"><span class="pl-pds">'</span>Accept<span class="pl-pds">'</span></span>=&gt;<span class="pl-s1"><span class="pl-pds">'</span>*/*<span class="pl-pds">'</span></span>, <span class="pl-s1"><span class="pl-pds">'</span>Accept-Encoding<span class="pl-pds">'</span></span>=&gt;<span class="pl-s1"><span class="pl-pds">'</span>gzip;q=1.0,deflate;q=0.6,identity;q=0.3<span class="pl-pds">'</span></span>, <span class="pl-s1"><span class="pl-pds">'</span>User-Agent<span class="pl-pds">'</span></span>=&gt;<span class="pl-s1"><span class="pl-pds">'</span>Ruby<span class="pl-pds">'</span></span>}</pre></div>

The reason is that you started to use the WebMock gem in week 5, blocking the test code HTTP connections by default. The Javascript beer list tries to fetch the beers list in json form from the server, in fact. You get over this if you allow the connections to the local server, for instance adding the following command to the beginning of the <code>before</code> code chunk:

<pre><code>WebMock.disable_net_connect!(allow_localhost:true)
</code></pre>

The test works finally, but it does not go through. You'll see that the original issue hasn't been fixed: even though you added the beers to the database in the <code>before :each</code> chunk, the database looks empty.

The reason is that when you run tests with Selenium, the Respec's normal way to execute the test in one single database transection (which automatically executes a rollback operation at the end of the test, resetting all the database changes) is not supported (see <a href="https://github.com/jnicklas/capybara#transactions-and-database-setup">https://github.com/jnicklas/capybara#transactions-and-database-setup</a>). You will have to disable this property with the command <code>self.use_transactional_fixtures = false</code>, at the beginning of the tests to run through Selenium.</code>

The bad news is that the test data saved in the database will not be revomed after each test. The DatabaseCleaner gem <a href="https://github.com/bmabey/database_cleaner">https://github.com/bmabey/database_cleaner</a> will help you here anyway. Get started with it, by adding the following line to the Gemfile test scope

<pre><code>gem 'database_cleaner'
</code></pre>

And execute <code>bundle install</code>.

Now configure your tests so that <em>before all tests</em> (<code>before :all</code>) you set off the transactionality, allow the HTTP connections to the local server, and define the strategy to use for DatabaseCleaner (see <a href="http://stackoverflow.com/questions/10904996/difference-between-truncation-transaction-and-deletion-database-strategies">http://stackoverflow.com/questions/10904996/difference-between-truncation-transaction-and-deletion-database-strategies</a>). <em>Before each test</em> (<code>before :each</code>) DatabaseCleaner is started and after each test (<code>after :each</code>) DatabaseCleaner is requested to reset the database. When all tests are executed (<code>after :all</code>) the normal transactionality is returned:

<div class="highlight highlight-ruby"><pre><span class="pl-k">require</span> <span class="pl-s1"><span class="pl-pds">'</span>rails_helper<span class="pl-pds">'</span></span>

describe <span class="pl-s1"><span class="pl-pds">"</span>beerlist page<span class="pl-pds">"</span></span> <span class="pl-k">do</span>

  before <span class="pl-c1">:all</span> <span class="pl-k">do</span>
    <span class="pl-v">self</span>.use_transactional_fixtures <span class="pl-k">=</span> <span class="pl-c1">false</span>
    <span class="pl-s3">WebMock</span>.disable_net_connect!(<span class="pl-c1">allow_localhost:</span><span class="pl-c1">true</span>)
  <span class="pl-k">end</span>

  before <span class="pl-c1">:each</span> <span class="pl-k">do</span>
    <span class="pl-s3">DatabaseCleaner</span>.strategy <span class="pl-k">=</span> <span class="pl-c1">:truncation</span>
    <span class="pl-s3">DatabaseCleaner</span>.start

    <span class="pl-vo">@brewery1</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Koff<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@brewery2</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Schlenkerla<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@brewery3</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:brewery</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Ayinger<span class="pl-pds">"</span></span>)
    <span class="pl-vo">@style1</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Lager<span class="pl-pds">"</span></span>
    <span class="pl-vo">@style2</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Rauchbier<span class="pl-pds">"</span></span>
    <span class="pl-vo">@style3</span> <span class="pl-k">=</span> <span class="pl-s3">Style</span>.create <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Weizen<span class="pl-pds">"</span></span>
    <span class="pl-vo">@beer1</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span> <span class="pl-vo">@brewery1</span>, <span class="pl-c1">style:</span> <span class="pl-vo">@style1</span>)
    <span class="pl-vo">@beer2</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Fastenbier<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span> <span class="pl-vo">@brewery2</span>, <span class="pl-c1">style:</span> <span class="pl-vo">@style2</span>)
    <span class="pl-vo">@beer3</span> <span class="pl-k">=</span> <span class="pl-s3">FactoryGirl</span>.create(<span class="pl-c1">:beer</span>, <span class="pl-c1">name:</span> <span class="pl-s1"><span class="pl-pds">"</span>Lechte Weisse<span class="pl-pds">"</span></span>, <span class="pl-c1">brewery:</span> <span class="pl-vo">@brewery3</span>, <span class="pl-c1">style:</span> <span class="pl-vo">@style3</span>)
  <span class="pl-k">end</span>

  after <span class="pl-c1">:each</span> <span class="pl-k">do</span>
    <span class="pl-s3">DatabaseCleaner</span>.clean
  <span class="pl-k">end</span>

  after <span class="pl-c1">:all</span> <span class="pl-k">do</span>
    <span class="pl-v">self</span>.use_transactional_fixtures <span class="pl-k">=</span> <span class="pl-c1">true</span>
  <span class="pl-k">end</span>

  it <span class="pl-s1"><span class="pl-pds">"</span>shows one known beer<span class="pl-pds">"</span></span>, <span class="pl-c1">js:</span> <span class="pl-c1">true</span> <span class="pl-k">do</span>
    visit beerlist_path
    save_and_open_page
    expect(page).to have_content <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>
  <span class="pl-k">end</span>
<span class="pl-k">end</span></pre></div>

The process was not the easiest, but now tests finally work!

When you create page contents with Javascript, these contents do not appear on the page together with the HTML base, but only later on, when the Javascript is loaded after executing the return call function. So if you look at the page contents right after navigating to the page, the Javascript won't have managed to form the final page outlook. For instance, the following <code>save_and_open_page</code> may open a page, that does not contain beer yet:

<div class="highlight highlight-ruby"><pre>  it <span class="pl-s1"><span class="pl-pds">"</span>shows a known beer<span class="pl-pds">"</span></span>, <span class="pl-c1">js:</span><span class="pl-c1">true</span> <span class="pl-k">do</span>
    visit beerlist_path
    save_and_open_page
    expect(page).to have_content <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>
  <span class="pl-k">end</span></pre></div>

As the page <a href="https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends">https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends</a> says, Capybara is able to wait for asycronic Javascript calls till the page elements required for the tests have loaded.

It's known that the Javascript should add lines to the page table. You will get the page to look correct by adding the command <code>find('table').find('tr:nth-child(2)')</code> at its beginning, which looks for a table in the page and for the secondline inside the table (the table first line is already the table title in the page background):

<div class="highlight highlight-ruby"><pre>  it <span class="pl-s1"><span class="pl-pds">"</span>shows a known beer<span class="pl-pds">"</span></span>, <span class="pl-c1">:js</span> =&gt; <span class="pl-c1">true</span> <span class="pl-k">do</span>
    visit beerlist_path
    find(<span class="pl-s1"><span class="pl-pds">'</span>table<span class="pl-pds">'</span></span>).find(<span class="pl-s1"><span class="pl-pds">'</span>tr:nth-child(2)<span class="pl-pds">'</span></span>)
    save_and_open_page
    expect(page).to have_content <span class="pl-s1"><span class="pl-pds">"</span>Nikolai<span class="pl-pds">"</span></span>
  <span class="pl-k">end</span></pre></div>

Capybara will now wait, moving to the command to open the page only when the table is loaded (in fact, only two lines of the table will be ready for sure).

<blockquote>

<a id="user-content-tehtävä-4" class="anchor" href="#teht%C3%A4v%C3%A4-4" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 4

Implement a test to check that the beers are sorted in ascending order against their name in the beerlist page by default.

The test can be implemented now so using the <code>find</code> selector and making sure that each line has the right contents. Because the table has one line for the header, the first actual line can be found like this:

find('table').find('tr:nth-child(2)')

The line contents can be tested as usually with the expect and have_content methods.


<a id="user-content-tehtävä-5" class="anchor" href="#teht%C3%A4v%C3%A4-5" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 5

Test the following pieces of functionality

<ul class="task-list">
<li>clicking on the column 'style,' the beers are sorted in ascending order against their style name</li>
<li>clicking on the column 'brewery,' the beers are sorted in ascending order against their brewery name</li>
</ul>
</blockquote>

<strong>Attention.</strong> Travis is not able to run Selenium tests directly. A solution to the probelm can be found here  <a href="http://about.travis-ci.org/docs/user/gui-and-headless-browsers/#Using-xvfb-to-Run-Tests-That-Require-GUI-(e.g.-a-Web-browser)">http://about.travis-ci.org/docs/user/gui-and-headless-browsers/#Using-xvfb-to-Run-Tests-That-Require-GUI-(e.g.-a-Web-browser)</a>
Making Travis work after the changes is optional.


<a id="user-content-asset-pipeline" class="anchor" href="#asset-pipeline" aria-hidden="true"><span class="octicon octicon-link"></span></a>Asset pipeline

The Javascript and style files (and pictures) of Rails applications are managed with the so-called Asset pipeline, see <a href="http://guides.rubyonrails.org/asset_pipeline.html">http://guides.rubyonrails.org/asset_pipeline.html</a>

The idea is that the application developer places the Javascript files in the folder <em>app/assets/javascripts</em> and the style files in <em>app/assets/stylesheets</em>. Both can be placed in various different files and subfolders, if needed.

When the application is in the so-called development mood, Rails links all the Javascript and style files (which defined in the Manifest file) together in the application. If you check the application with the view source property of your browser, you will notice that a large amount of Javascript and style files are linked together there.

The Javascript files to link in the application are defined in the file <em>app/assets/javascripts/application.js</em>, whose contents look like this now

<div class="highlight highlight-javascript"><pre><span class="pl-c">//= require jquery</span>
<span class="pl-c">//= require jquery_ujs</span>
<span class="pl-c">//= require turbolinks</span>
<span class="pl-c">//= require bootstrap</span>
<span class="pl-c">//= require_tree .</span></pre></div>

Even though the whole file contents look as if they were comments, they are actually the "real" commands of the <a href="https://github.com/sstephenson/sprockets">sprockets translator</a> that takes care of asset pipeline. They help define the Javascript files that have to be linke in the application. The first four lines tell to take jquery, jquery_ujs, turbolinks, and bootstrap. These are all set up in the application through gems.

The last line defines that all the Javascript files contained in <em>assets/javascripts/</em> as well as in its subfolders have to be included in the program.

Asset pipeline also allows you to make use of <a href="http://coffeescript.org/">coffeescript</a>, in which case the files ending will be <code>.js.coffee</code>. When the application is run, Scprockets will translate Coffeescript into Javascript automatically.

For execution performance reasons, it is usually better to avoid using too many Javascript and style files in development mood applications. When the application is started in production mood, Sprockets links all the application Javascript and style files into singular, optimised files. You'll notice this if you look at application HTML source code in Heroku: for instance <a href="http://wad-ratebeer.herokuapp.com/">http://wad-ratebeer.herokuapp.com/</a> contains now only one js and one css files, and expecially the js file readability is weak for an individual person.

More about asset pipiline and for instance Javascript linking in Rails applications, at:

<ul class="task-list">
<li><a href="http://railscasts.com/episodes/279-understanding-the-asset-pipeline">http://railscasts.com/episodes/279-understanding-the-asset-pipeline</a></li>
<li><a href="http://railsapps.github.io/rails-javascript-include-external.html">http://railsapps.github.io/rails-javascript-include-external.html</a></li>
</ul>

<blockquote>

<a id="user-content-tehtävä-6-8-kolmen-tehtävän-arvoinen" class="anchor" href="#teht%C3%A4v%C3%A4-6-8-kolmen-teht%C3%A4v%C3%A4n-arvoinen" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercises 6-8 (it gives three points)

<h3>
<a id="user-content-tehtävä-on-hieman-työläs-joten-tee-ensin-helpommat-pois-alta-muut-viikon-tehtävät-eivät-riipu-tästä-tehtävästä" class="anchor" href="#teht%C3%A4v%C3%A4-on-hieman-ty%C3%B6l%C3%A4s-joten-tee-ensin-helpommat-pois-alta-muut-viikon-teht%C3%A4v%C3%A4t-eiv%C3%A4t-riipu-t%C3%A4st%C3%A4-teht%C3%A4v%C3%A4st%C3%A4" aria-hidden="true"><span class="octicon octicon-link"></span></a>The exercise is hard, so first make the easier ones. The other exercises don't depend on this.</h3>

Anyone can join as beer club member in your application, so far. Change your application now, so that the membership has to be confirmed by old members before new ones can join.

Some notes

<ul class="task-list">
<li>the best way to implement membership is that the Membership model is added the boolean field <em>confirmed</em>
</li>
<li>When a club is created, the user who created it should automatically become that club member</li>
<li>Show a list of the membership applications which haven't been confirmed on the club page</li>
<li>Membership status change can be managed for instance with its own <a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/blob/master/web/viikko6.md#reitti-panimon-statuksen-muuttamiselle">custom route</a>.</li>
</ul>

The exercise may be a bit challenging. The section <a href="http://guides.rubyonrails.org/association_basics.html#controlling-association-scope">4.3.3 Scopes for has_many</a> of the Active Record Associations guide provides a good tool to make the exercise. Of course, the exercise can also be solved in different ways.
</blockquote>

At the end of the exercise, you application will look something like this. The beer club page shows a list of the membership applications, if the signed-in user is already that beer club user:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-6.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-6.png" alt="kuva" style="max-width:100%;"></a>

Users personal pages show the applications which haven't been confirmed yet:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-5.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-5.png" alt="kuva" style="max-width:100%;"></a>


<a id="user-content-indeksi-tietokantaan" class="anchor" href="#indeksi-tietokantaan" aria-hidden="true"><span class="octicon octicon-link"></span></a>Index to the database

When the user signs in the system, the session controller executes an operation to retrive the user object from the database against the user name:

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">SessionsController<span class="pl-e"> &lt; ApplicationController</span></span>
  <span class="pl-k">def</span> <span class="pl-en">create</span>
    user <span class="pl-k">=</span> <span class="pl-s3">User</span>.find_by <span class="pl-c1">username:</span> params[<span class="pl-c1">:username</span>]

     <span class="pl-c"># ...</span>
  <span class="pl-k">end</span>

<span class="pl-k">end</span></pre></div>

In order to execute the operation, the database has to go through the whole <code>users</code> table. Searches for the object ID are faster, because each table has been indexed against their ID. The index works as with hash tables, providing access to the required database line in "O(1)" time.

Database tables can be added other indexes if needed. Add an index to the <code>users</code> table, making the search against user ID faster.

Create a migration for the index

<pre><code>rails g migration AddUserIndexBasedOnUsername
</code></pre>

The migration is the following:

<div class="highlight highlight-ruby"><pre><span class="pl-k">class</span> <span class="pl-en">AddUserIndexBasedOnUsername<span class="pl-e"> &lt; ActiveRecord::Migration</span></span>
  <span class="pl-k">def</span> <span class="pl-en">change</span>
    add_index <span class="pl-c1">:users</span>, <span class="pl-c1">:username</span>
  <span class="pl-k">end</span>
<span class="pl-k">end</span></pre></div>

Execute the migration with the command <code>rake db:migrate</code> and the index is ready!

The bad thing about this is that when the system is added a new user or an existing user is deleted, the index has to be edited and this requires time obviuosly. Adding an index is a tradeoff of what operation you want to optimize, then.


<a id="user-content-laiska-lataaminen-n1-ongelma-ja-tietokantakyselyjen-optimointi" class="anchor" href="#laiska-lataaminen-n1-ongelma-ja-tietokantakyselyjen-optimointi" aria-hidden="true"><span class="octicon octicon-link"></span></a>Lazy loading, the n+1 issue, and database request optimisation

The controller to show all beers is simple. The beers are fetched from the database, sorted according to what the parameter of the HTTP call defines, and are assigned to a variable for the template:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.all

    order <span class="pl-k">=</span> params[<span class="pl-c1">:order</span>] <span class="pl-k">||</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span>

    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-k">case</span> order
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>name<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by{ |<span class="pl-vo">b</span>| b.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>brewery<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by{ |<span class="pl-vo">b</span>| b.brewery.name }
      <span class="pl-k">when</span> <span class="pl-s1"><span class="pl-pds">'</span>style<span class="pl-pds">'</span></span> <span class="pl-k">then</span> <span class="pl-vo">@beers</span>.sort_by{ |<span class="pl-vo">b</span>| b.style.name }
    <span class="pl-k">end</span>
  <span class="pl-k">end</span></pre></div>

The template shows a table where the beers are listed:

<div class="highlight highlight-erb"><pre><span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-vo">@beers</span>.each <span class="pl-k">do </span>|<span class="pl-vo">beer</span>| </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
  &lt;<span class="pl-ent">tr</span>&gt;
    &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.name, beer </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
    &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.style, beer.style </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
    &lt;<span class="pl-ent">td</span>&gt;<span class="pl-pse">&lt;%=</span><span class="pl-s2"> link_to beer.brewery.name, beer.brewery </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>&lt;/<span class="pl-ent">td</span>&gt;
  &lt;/<span class="pl-ent">tr</span>&gt;
<span class="pl-pse">&lt;%</span><span class="pl-s2"> <span class="pl-k">end</span> </span><span class="pl-pse"><span class="pl-s2">%</span>&gt;</span>
&lt;/<span class="pl-ent">table</span>&gt;</pre></div>

Simple and stylish... but not too efficient.

You could take a look at your log file log/development.log to see what happens when users go to the beers page. You will have access to the same piece of information in a fairly better form  through the <em>miniprofiler</em> gem (see <a href="https://github.com/MiniProfiler/rack-mini-profiler">https://github.com/MiniProfiler/rack-mini-profiler</a> and <a href="http://samsaffron.com/archive/2012/07/12/miniprofiler-ruby-edition">http://samsaffron.com/archive/2012/07/12/miniprofiler-ruby-edition</a>)

Getting started with Miniprofiler is easy, you only need to add the following line to your Gemfile

<pre><code>gem 'rack-mini-profiler'
</code></pre>

Execute <code>bundle install</code> and restart your Rails server. When you go to the address http:localhost:300/beers next time, you'll see a timer will have appeared on the upper side of the page. This measures the time used to execute the HTTP request. If you click the number, you'll find a better definition of the time frame:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-7.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-7.png" alt="kuva" style="max-width:100%;"></a>

The report shows that <code>Executing action: index</code> – which is the controller method execution – causes one SQL request <code>SELECT "beers".* FROM "beers"</code>. Instead, <code>Rendering: beers/index</code> – which is the view template execution – causes a succession of nine SQL requests!

Clicking on the requests you will be able to check their reason:

<a href="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-8.png" target="_blank"><img src="https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w6-8.png" alt="kuva" style="max-width:100%;"></a>

So, rendering the view template causes the following requests execution various times:

<div class="highlight highlight-ruby"><pre><span class="pl-vo">SELECT</span>  <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>  <span class="pl-vo">WHERE</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-k">=</span> <span class="pl-k">?</span>  <span class="pl-vo">ORDER</span> <span class="pl-vo">BY</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-vo">ASC</span> <span class="pl-vo">LIMIT</span> <span class="pl-c1">1</span>

<span class="pl-vo">SELECT</span>  <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>  <span class="pl-vo">WHERE</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-k">=</span> <span class="pl-k">?</span>  <span class="pl-vo">ORDER</span> <span class="pl-vo">BY</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-vo">ASC</span> <span class="pl-vo">LIMIT</span> <span class="pl-c1">1</span></pre></div>

In fact, a request to the tables of both <code>styles</code> and <code>breweries</code> is made for each singular beer.

The reason is that Activerecord is set up on <em>lazy loading</em> by default, and an object fields are fetched from the database only if they are referred to. This is reasonable sometimes, if an object is related to a huge amount of objects and not all of them are needed straight from the beginning. When you access the page of all beers, lazy loading is not the best idea, because you know for sure that you have to show the brewery and style names for each beer, and these pieces of information can be found only from the brewery and style database tables.

You can guide the SQL generated from the requests by acting on the method parameters. For instance, the following tells that breweries and not only the beers have to be fetched from the database:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.includes(<span class="pl-c1">:brewery</span>).all
    <span class="pl-c"># ...</span>
  <span class="pl-k">end</span></pre></div>

With the help of Miniprofiler, you'll see that the controller execution causes two requests:

<div class="highlight highlight-ruby"><pre><span class="pl-vo">SELECT</span> <span class="pl-s1"><span class="pl-pds">"</span>beers<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>beers<span class="pl-pds">"</span></span>
<span class="pl-vo">SELECT</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>  <span class="pl-vo">WHERE</span> <span class="pl-s1"><span class="pl-pds">"</span>breweries<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-vo">IN</span> (<span class="pl-c1">1</span>, <span class="pl-c1">2</span>, <span class="pl-c1">3</span>, <span class="pl-c1">6</span>)</pre></div>

The view template execution causes only five requests now, and they are all like:

<div class="highlight highlight-ruby"><pre>
<span class="pl-vo">SELECT</span>  <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>  <span class="pl-vo">WHERE</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-k">=</span> <span class="pl-k">?</span>  <span class="pl-vo">ORDER</span> <span class="pl-vo">BY</span> <span class="pl-s1"><span class="pl-pds">"</span>styles<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-vo">ASC</span> <span class="pl-vo">LIMIT</span> <span class="pl-c1">1</span></pre></div>

When the view is rendered, the styles have to be fetched from the database now, each with their own SQL request.

Optimize the controller in a way that all the styles needed are also read from the database at once:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">index</span>
    <span class="pl-vo">@beers</span> <span class="pl-k">=</span> <span class="pl-s3">Beer</span>.includes(<span class="pl-c1">:brewery</span>, <span class="pl-c1">:style</span>).all

    <span class="pl-c"># ...</span>
  <span class="pl-k">end</span></pre></div>

The controller execution causes now three requests, and rendering the view requires only one request. Miniprofiler shows that the request is

<div class="highlight highlight-ruby"><pre><span class="pl-vo">SELECT</span>  <span class="pl-s1"><span class="pl-pds">"</span>users<span class="pl-pds">"</span></span>.<span class="pl-k">*</span> <span class="pl-vo">FROM</span> <span class="pl-s1"><span class="pl-pds">"</span>users<span class="pl-pds">"</span></span>  <span class="pl-vo">WHERE</span> <span class="pl-s1"><span class="pl-pds">"</span>users<span class="pl-pds">"</span></span>.<span class="pl-s1"><span class="pl-pds">"</span>id<span class="pl-pds">"</span></span> <span class="pl-k">=</span> <span class="pl-k">?</span> <span class="pl-vo">LIMIT</span> <span class="pl-c1">1</span></pre></div>

and the reason is

<div class="highlight highlight-ruby"><pre>app<span class="pl-k">/</span>controllers<span class="pl-k">/</span>application_controller.<span class="pl-c1">rb:</span><span class="pl-c1">10</span><span class="pl-c1">:in</span> <span class="pl-s1"><span class="pl-pds">'</span>current_user<span class="pl-pds">'</span></span></pre></div>

eli näytön muuttujan <code>current_user</code> avulla tekemä viittaus kirjautuneena olevaan käyttäjään. Tämä ei kuitenkaan ole hirveän vakavaa.

From 1+2n (where n stands for the number of beers in the database), you managed to optimise the number of SQL requests to three (plus one)!

This is called the n+1 problem (see <a href="http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations">http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations</a>): when fetching a list of beers with one database request, each object of the list causes a new database search, and instead of only one search, around n+1 searches actually happen.

Before the next exercise, change the template to list all the users to look like

<div class="highlight highlight-ruby"><pre><span class="pl-k">&lt;</span>h1<span class="pl-k">&gt;</span><span class="pl-vo">Users</span><span class="pl-k">&lt;</span><span class="pl-k">/</span>h1<span class="pl-k">&gt;</span>

<span class="pl-k">&lt;</span>table <span class="pl-k">class</span><span class="pl-k">=</span><span class="pl-s1"><span class="pl-pds">"</span>table table-hover<span class="pl-pds">"</span></span><span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span>thead<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span>tr<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span><span class="pl-vo">Username</span><span class="pl-k">&lt;</span><span class="pl-k">/</span>th<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> rated beers <span class="pl-k">&lt;</span><span class="pl-k">/</span>th<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>th<span class="pl-k">&gt;</span> total ratings <span class="pl-k">&lt;</span><span class="pl-k">/</span>th<span class="pl-k">&gt;</span>
      <span class="pl-k">&lt;</span>th&gt;<span class="pl-k">&lt;</span><span class="pl-k">/</span>th<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span><span class="pl-k">/</span>tr<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span><span class="pl-k">/</span>thead<span class="pl-k">&gt;</span>
  <span class="pl-k">&lt;</span>tbody<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-vo">@users</span>.each <span class="pl-k">do </span>|<span class="pl-vo">user</span>| <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">      &lt;tr<span class="pl-pds">&gt;</span></span>
        <span class="pl-k">&lt;</span>td&gt;<span class="pl-k">&lt;</span><span class="pl-k">%=</span> link_to user.username, user <span class="pl-s1"><span class="pl-pds">%&gt;</span>&lt;/td<span class="pl-pds">&gt;</span></span>
        <span class="pl-k">&lt;</span>td&gt;<span class="pl-k">&lt;</span><span class="pl-k">%=</span> user.beers.size <span class="pl-s1"><span class="pl-pds">%&gt;</span>&lt;/td<span class="pl-pds">&gt;</span></span>
        <span class="pl-k">&lt;</span>td&gt;<span class="pl-k">&lt;</span><span class="pl-k">%=</span> user.ratings.size <span class="pl-s1"><span class="pl-pds">%&gt;</span>&lt;/td<span class="pl-pds">&gt;</span></span>
        <span class="pl-k">&lt;</span>td<span class="pl-k">&gt;</span>
          <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">if</span> is_admin? <span class="pl-k">and</span> user.is_frozen? <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">            &lt;span class="label label-info"<span class="pl-pds">&gt;</span></span>account frozen<span class="pl-k">&lt;</span><span class="pl-k">/</span>span<span class="pl-k">&gt;</span>
          <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">end</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">        &lt;/td<span class="pl-pds">&gt;</span></span>
      <span class="pl-k">&lt;</span><span class="pl-k">/</span>tr<span class="pl-k">&gt;</span>
    <span class="pl-k">&lt;</span><span class="pl-k">%</span> <span class="pl-k">end</span> <span class="pl-s1"><span class="pl-pds">%&gt;</span></span>
<span class="pl-s1">  &lt;/tbody<span class="pl-pds">&gt;</span></span>
<span class="pl-k">&lt;</span><span class="pl-k">/</span>table<span class="pl-k">&gt;</span></pre></div>

<blockquote>

<a id="user-content-tehtävä-9" class="anchor" href="#teht%C3%A4v%C3%A4-9" aria-hidden="true"><span class="octicon octicon-link"></span></a>Exercise 9

The change causes the n+1 problem to the users page. Fix the problem eager loading the required objects when the users are fetched, like in the exercise above. Make sure that the optimisation works with miniprofiler.
</blockquote>

<strong>Attention:</strong> if the table was added also the favourite beer column

<pre><code> &lt;td&gt;&lt;%= user.favorite_beer %&gt;&lt;/td&gt;
</code></pre>

the situation would be a bit harder in terms of SQL optimisation. The last version of your method was this:

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite_beer</span>
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?
    ratings.order(<span class="pl-c1">score:</span> <span class="pl-c1">:desc</span>).limit(<span class="pl-c1">1</span>).first.beer
  <span class="pl-k">end</span></pre></div>

Now, not even eager loading will help, because the method call causes an SQL request in any case. Instead, if you implemented the beer ratings in the method central memory (as you did at the beginning of week 4):

<div class="highlight highlight-ruby"><pre>  <span class="pl-k">def</span> <span class="pl-en">favorite_beer</span>
    <span class="pl-k">return</span> <span class="pl-c1">nil</span> <span class="pl-k">if</span> ratings.empty?
    ratings.sort_by{ |<span class="pl-vo">r</span>| r.score }.last.beer
  <span class="pl-k">end</span></pre></div>

the method call <em>would not</em> cause any database operations <em>if</em> the ratings are eager loaded when the method is called.

It may also make sense to keep two versions of the method in some situations to optimise its performance, one that executes the operation at database level and the other which does it in central memory.


<a id="user-content-cachays-eli-palvelinpuolen-välimuistitoiminnallisuudet" class="anchor" href="#cachays-eli-palvelinpuolen-v%C3%A4limuistitoiminnallisuudet" aria-hidden="true"><span class="octicon octicon-link"></span></a>Server chaching functionality

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

