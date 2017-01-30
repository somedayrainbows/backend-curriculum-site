---
title: Consuming Apis
length: 180
tags: API, json, HTTP client, faraday, hurley, HTTParty, Postman
---

### Goals

By the end of this lesson, you will know/be able to:

* Explain what an API is and what an external API provides to us
* Know why we need APIs
* Set up an HTTP client for an API
* Create an application that consumes an API
* Access JSON data returned from an API
* Hide api keys within your app

### Structure

* 10 mins - API Discussion and Clarification
* 15 mins - Exploring the external API
* 5  mins - Break
* 25 mins - Create API consuming app - Service
* 5 mins - Break
* 25 mins - Continue with our app - Service/Models
* 5 mins - Break
* 25 mins - Continue with our app - Models/Controllers
* 15 mins - Break
* 25 mins - Continue with our app - Controllers/views
* 5 mins - Break
* 20 mins - Finish up and review

## Discussion

### What is an API?

* What does API stand for?
* What are some common use cases and examples of APIs
* Any experience with APIs?
* Commonalities between APIs? Differences?
* Are there _better_ APIs out there?

### APIs: Variety is the Spice of Life

One of the problems with working with APIs is that there are very few consistent
generalizations or assumptions that can be made.

As an exercise, let's consider some of the things that make it (relatively)
easy for us, as Rubyists, to read code written by other Rubyists:

* Common idioms for expressing ourselves
* Common language features for structuring our code (classes, OOP, method practices)
* Common set of tools for designing projects
* Common style practices (capitalization, underscores, spelling, predicate methods, etc)
* Shared community best practices -- `SOLID`, Sandi Metz' rules, conference talks, ruby books, etc etc

In short, when working within the ruby community, we have a lot of shared practices
and ideas that help us write "unsurprising" code that will hopefully be easily intelligble
to a future reader, especially one versed in the same community idioms as we are.

Why bring this up? Because we are about to make our first forays into the realm of
public APIs, which is, comparatively the Wild Wild West.

Let's consider some of the reasons why we might find relative non-conformity within
this corner of the tech world:

* Huge cross-section of language and community backgrounds (power of HTTP is its accessibility
from any platform)
* Very little shared design principles. Even around the ideas of REST there are differing interpretations
and implementations
* Lack of standardization around request/response formats, status codes, etc. etc.
* Tremendous variation in quality of documentation (often very little...)

In short, there's a good chance that no 2 APIs you will encounter are alike.

A handful of community projects such as [Swagger](http://swagger.io/) and [json:api](http://jsonapi.org/)
have attempted to address this, but even these are fragmented and not well adopted.

([See Relevant XKCD](https://xkcd.com/927/))

So why should we venture into this cesspool of non-conforming inter-process communication?

Because APIs let us do cool stuff. And fortunately HTTP with JSON (or even XML) is a relatively
easy to understand format.

With a little careful experimentation and probing, we can generally figure out what we need to do.
But it's good to have a sense of what to expect so that when we run into issues we won't be surprised and
will know what to do.

### API Spectrum of Quality

You can never really know what to expect from an API, but here are a few general predictors

* The larger the platform, the better. The top names in the social media space (twitter, foursquare,
facebook, instagram, etc) will generally have better public APIs since more people use them.
* The newer the better. Unsurprisingly an API that was implemented in the early 2000's will
often feel dated or clunky
* The more "RESTful" the better. This is hard to predict until you start digging in,
but the more Resource-oriented an API is, the more natural it will feel to use. You can often
"guess" a resource or endpoint and be relatively close

## Let Us Explore!

* Postman
* Bearer token authentication
* api url structure
* .json
* If you want to see how an api is built - `git clone git@github.com:neight-allen/my_chaims.git`

![How Models Work](http://i.imgur.com/FVg9ODy.png?1)

### Postman Exercise
The first step to consuming an api is always to read the documentation for the api you are planning on consuming. Let’s take a look at the iTunes search API [here](https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/#searching). After reading through the documentation, can you find the keys we need to search for songs by Gloria Estefan, limiting the results to 10 and excluding any explicit songs. After running the search url with params through Postman, can you see the keys needed to return the song name, album name, date it was released, and price for the song on iTunes?

__Finish the exercise above before scrolling down for the answer__

The url with params should look something like this:

https://itunes.apple.com/search?term=Gloria+Estefan&country=US&media=music&entity=musicTrack&limit=10&explicit=No

Keys for:
Song Name: “trackName”
Album: “collectionName”
Date Released: “releaseDate”
Song Price: “trackPrice"

## Let Us Build!

### Prep Work
Signup for an api key to access NREL data [here](https://developer.nrel.gov/signup/).

We are going to be using the NREL API to build an app that retrieves the nearest fuel stations to a provided zip code.  We also want to set some limits to the results that are returned. Here is what we want to limit the results to:
* 5 miles of the zip code
* The fuel type to only include Electric & Propane (sometimes called Liquefied Petroleum Gas)
* Only return 10 stations

As mentioned above, our very first step is always going to be to read documentation. Let’s start by going out to the NREL API docs [here](http://developer.nrel.gov/docs/) and seeing if we can find the endpoint we need to get the nearest alternative fuel stations.

Like many API docs, you sometimes have to dig through to get to the endpoint you need. To get to the nearest fuel stations endpoint, navigation will look like this: `Transportation > Alternative Fuel Stations > Nearest Stations`.

Now we can see the type of request `GET` and request url we need to get back the data `/api/alt-fuel-stations/v1/nearest.format?parameters` Notice that the endpoint here does not include the base url. In some apis, they explicitly give you the base url. In this case, if you scroll down on the page to examples and click on one of the request urls, you can see in the url box in the browser, that the base url is `developer.nrel.gov`.

Now that we know what are endpoint is, let’s figure out what the parameters are for the limits we want to set. To reiterate, these are the limits we want to set:

* zip code
* 5 miles within the zip code
* fuel types of Electric & Propane
* limit the results to 10

Based off of this, what are the parameters we want to set in the url?

__Look at the documentation before scrolling down for the answer__

Here are the params we are going to use:

`location` to provide the zip code
`radius` to set the results to be within 5 miles of the zip code we provide
`fuel_type` to set results to only include Electric  & Propane stations
`limit` to limit results to 10

Lastly, we need to make sure we are also using `api_key` and use the api key we were given.

Now that we have all we need to hit the end point, let’s enter all of this information into Postman to see what kind of results we get. Once you send the request, you should see a key `total_results` that should have 66 as the count of how many stations were returned. If you change the radius to 10, you’ll see that the number changes to 142.

We know what the data is going to look like now, so let’s build this thing!

### Build


We will start by building out our Service using TDD. Once we get our tests passing, we can start to refactor out to a Model, then Controllers & Views.

Create the rails app.

```sh
$ rails new nrel_fuel_stations -T —database=postgresql
$ cd nrel_fuel_stations
$ bundle install
$ rake db:create
```

The first file we are goign to be creating is `app/services/nrel_service.rb`. Wait, but what is a service? [Check out Ben Lewis' Post on Services](https://blog.engineyard.com/2014/keeping-your-rails-controllers-dry-with-services)
In order to hit the api within our app, we are going to be using the [Faraday](https://github.com/lostisland/faraday) gem. We are also going to be TDD’ing so we want to make sure we setup our tests. To debug, we are going to use pry. Let’s setup all of these gems.

In your Gemfile add rspec

```rb
gem 'faraday'

group :development, :test do
  gem 'rspec-rails'
  gem 'pry'
end
```


```sh
$ bundle install
$ rails generate rspec:install
```

Setup your spec. Remember, we are going to be creating a service file which will be nested in the `services` directory of our app. Let’s do the same with our spec.

```sh
$ mkdir app/spec/services
$ touch app/spec/services/nrel_service_spec.rb
```

Create the service spec. We want to test that we can retrieve a list of fuel stations when we provide a zip code, and using the parameters above.

```sh
$ mkdir spec/services
$ touch spec/services/nrel_service_spec.rb
```

# spec/services/nrel_service_spec.rb

```rb
require 'rails_helper'

RSpec.describe 'NREL Service' do
  it 'can return a list of fuel stations with a specific zip code' do
      service = NrelService.new
      stations = service.get_stations('80203')

      expect(stations.count).to eq(10)
      expect(stations.class).to eq(Array)
      
      station = stations.first

      expect(stations.class).to be(Hash)
      expect(station).to have_key(:station_name)
      expect(station).to have_key(:access_days_time)
      expect(station).to have_key(:fuel_type_code)
      expect(station).to have_key(:city)
      expect(station).to have_key(:state)
      expect(station).to have_key(:street_address)
      expect(station).to have_key(:zip)
      expect(station).to have_key(:distance)
    end
end
```

Run your tests and you should get an error `uninitialized constant NrelService`

Setup the service

```sh
$ mkdir app/services
$ touch app/services/nrel_service.rb
```

# app/services/nrel_service.rb

```rb
class NrelService
end
```

Run your tests and you should get an error `undefined method 'get_stations'`. Let set up our method.

# app/services/nrel_service.rb

```rb
class NrelService

  def get_stations(zip)
  end
end
```

Run your tests and you now should get an error `undefined method ‘count’ for nil:NilClass`. Let’s go ahead and hit the api! To get our tests passing, let’s put all of our code in the `get_stations` method for now. Since we used Postman to test our data, we can take the url from postman and copy it into our code.

```rb
class NrelService

  def get_stations(zip)
    response = Faraday.get("https://developer.nrel.gov/api/alt-fuel-stations/v1/nearest.json?location=#{zip}&api_key=[your_api_key_here]&fuel_types=ELEC,LPG&radius=5&limit=10")
  end

end
```

Run your tests and you should get an error `undefined method ‘count’ for #<Faraday::Response:0x007fc80d1300d8>`. Let’s use pry to debug and see what the response actually looks like.

```rb
class NrelService

  def get_stations(zip)
      response = Faraday.get(‘https://developer.nrel.gov/api/alt-fuel-stations/v1/nearest.json?location=#{zip}&api_key=[your_api_key_here]&fuel_types=ELEC,LPG&radius=5&limit=10')
      binding.pry
  end
end
```

Now when you run your tests, it should hit your pry. Let’s look at the response.

```sh
pry(#<NrelService>) > response
```

We can see that the entire response is sent back to us, which is great, but what is it that we actually want? We just need what is in the body of the response. We can access this by calling `body` on the response.

```sh
pry(#<NrelService>) > response.body
```

Great! Now we have the info we need, BUT the format doesn’t look quite right for us to be able to use this in our code. What format is this in? Hint: We specified this in our url. JSON! Let’s parse the JSON in order to get back the hash we want. We can do this using the `JSON.parse()` method.

```sh
pry(#<NrelService>) > JSON.parse(response.body)
```

Now it looks like we almost have what we need. `JSON.parse(response.body)` give us back all of the results, but it also includes information about all of the results. How can we only get back the fuel_stations? We can access the data within the key `fuel_stations`. Notice that all of the keys are strings. Let’s say we want these to be symbols instead to reduce the typing of quotes. We can add `symbolize_names: true` to our parse method to convert all of the keys to symbols. 

```sh
pry(#<NrelService>) > JSON.parse(response.body, symbolize_names: true)[:fuel_stations]
```

That looks like it’s returning what we want. Let’s update our code and run our tests again.

```rb
class NrelService

  def get_stations(zip)
      response = Faraday.get(‘https://developer.nrel.gov/api/alt-fuel-stations/v1/nearest.json?location=#{zip}&api_key=[your_api_key_here]&fuel_types=ELEC,LPG&radius=5&limit=10')
      JSON.parse(response.body, symbolize_names: true)[:fuel_stations]
  end
end
```

Run your tests. Heyoooo! Our tests pass! Now you can do a happy dance! ![](http://i.giphy.com/l3V0lsGtTMSB5YNgc.gif)

We can’t stop here though. Our code is quite ugly, but more importantly, if we returned this in the view, it would just be an array of hashes and we don’t want that. Ultimately, what we want to be doing is to be returning station objects, so we can call methods on the object to get the data we want.  Before pulling out code into a model though, let’s refactor our service. 

Right now, we have everything in the `get_stations` method, and our api call is all on one line. Let’s refactor a bit. First, we should take a look at the Faraday [docs](https://github.com/lostisland/faraday) to see if there is a better way to write our code. If you look under the Basic Usage section, you can see that we can make a base connection using the base url.  This allows us to create multiple methods with different endpoints in the future. Once you have this connection, you can see that there are two ways to pass parameters. We can do it by giving the connection object a hash, or by using block notation.  Let’s start by creating a base connection in our intialize method and then refactoring our `get_stations` method to use the connection. 

Here are the steps:

1. Set the connection to a variable using the Faraday block syntax
2. Make `@connection` an attr_reader under `private` so it is secure
3. Refactor `get_stations` to use `connection`

This is what the code should look like:

```rb
class NrelService

  def initialize
    @connection = Faraday.new(:url => 'https://developer.nrel.gov') do |faraday|
      faraday.request :url_encoded
      faraday.response :logger
      faraday.adapter Faraday.default_adapter
    end
  end
 
  def get_stations(zip)
    response = connection.get("/api/alt-fuel-stations/v1/nearest.json?location=#{zip}&api_key=[your_api_key_here]&fuel_types=ELEC,LPG&radius=5&limit=10")
    JSON.parse(response.body, symbolize_names: true)[:fuel_stations]
  end

  private
    attr_reader :connection
end
```

Run your tests and they should still pass. Next, let’s refactor `get_stations` so that we can get the request off of one line. Let’s use the block.

Block:

```rb
class NrelService

  def initialize
    @connection = Faraday.new(:url => 'https://developer.nrel.gov') do |faraday|
      faraday.request :url_encoded
      faraday.response :logger
      faraday.adapter Faraday.default_adapter
    end
  end
 
  def get_stations(zip)
    response = connection.get('/api/alt-fuel-stations/v1/nearest.json') do |request|
      request.params[‘location’]='80203'
      request.params['api_key’]= your_api_key_here
      fuel_types=ELEC,LPG
      radius=5
      limit=10
    end
    JSON.parse(response.body, symbolize_names: true)[:fuel_stations]
  end

  private
    attr_reader :connection
end
```

Your tests should still pass. If they aren’t, make sure you don’t have any typos. 

### Video

* [Oct 2015 - Consuming and API](https://vimeo.com/143957483)

### Repository

* [My-Chaims API repo](https://github.com/neight-allen/my_chaims)
* [Example of Final Curating Chaims Api Consuming App](https://github.com/Carmer/chaims_consumption_practice) - Uses an old version of the API that doesn't authenticate
* [1511 curating chaims](https://github.com/neight-allen/chaimz_curator) - Uses HTTParty
* [1602 curating chaims](https://github.com/neight-allen/chaimz-curator) - Uses Faraday

### Resources

* [JSON API](http://jsonapi.org/)
* [Government Data API](https://api.data.gov/)
* [World Bank API](http://data.worldbank.org/developers?display=)
* [YouTube data API](https://developers.google.com/youtube/v3/?hl=en)
* [Programmable Web](http://www.programmableweb.com/)
* [MashApe](https://www.mashape.com/)
* What is a service? [Check out Ben Lewis' Post on Services]

