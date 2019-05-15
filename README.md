# Creating a Rails API from Scratch

## Learning Goals

- Create an API-Only Rails Build

## Introduction

In the last few lessons, we saw how we can easily adapt the MVC structure of
Rails to render JSON. Rails is flexible enough to be able to respond to
different formats, and is ready to do so out of the box. For the purposes of
building applications in JavaScript and frameworks like React, though, we
specifically need it to act as an API that responds with JSON.

When first building a Rails application, it is possible to flag that the
application should be API-only. In this lesson, we will take a look at what this
means and how it provides us with some useful automatic configurations. We will
also briefly look at the `--database` flag, which will be necessary in our
pursuit of building our own APIs.

## Using the `--api` Flag

To create an API-only Rails build from scratch, include the `--api` after the
name of the Rails application on creation:

```sh
rails new bird-watcher-api --api
```

By using the `--api` flag, Rails will remove a lot of default features and
middleware, mostly related to the browser, since it won't be needed. Controllers
will inherit from `ActionController::API` rather than `ActionController::Base`
and generators will skip generating views.

One noticeable change - browser errors will disappear. Normally, when a Rails
server is running, it produces an error message in browser when something goes
wrong while attempting to render. Since there is no way to render views in this
API-only build, if the Rails API fails and we visit it in browser, it will just
show a blank screen.

No changes are required when setting up resources for an API-only Rails build.

## From Beginning to End

We've gone over the pieces of building an API, so to briefly summarize what
we've done in the last few lessons combined with this new knowledge, first we
create the API-only Rails build:

```sh
rails new bird-watcher-api --api
```

Then, navigate into the new Rails application once created. Rather than create
_everything_ by hand this time, we can use a generator to help us out with
resources.

```sh
rails g resource bird name species
rails g resource location latitude longitude
rails g resource sighting bird:references location:references
```

This will create three migrations, three models, and three empty controllers. With
minimal seed data we could then test that everything was working as expected:

```ruby
bird_a = Bird.create(name: "Black-Capped Chickadee", species: "Poecile Atricapillus")
bird_b = Bird.create(name: "Grackle", species: "Quiscalus Quiscula")
bird_c = Bird.create(name: "Common Starling", species: "Sturnus Vulgaris")
bird_d = Bird.create(name: "Mourning Dove", species: "Zenaida Macroura")

location_a = Location.create(latitude: "40.730610", longitude: "-73.935242")
location_b = Location.create(latitude: "30.26715", longitude: "-97.74306")
location_c = Location.create(latitude: "45.512794", longitude: "-122.679565")

sighting_a = Sighting.create(bird: bird_a, location: location_a)
sighting_b = Sighting.create(bird: bird_b, location: location_b)
sighting_c = Sighting.create(bird: bird_c, location: location_c)
```

Then the controller actions we want in the API will need to be added:

```ruby
def index
@sightings = Sighting.all
render json: @sightings, include: [:bird, :location]
end
```

Since the `resource` generator was used, it would be good to be diligent and
clean up `config/routes.rb` once we've decided what endpoints the API should
have.

## A Note While Developing APIs - Dealing with CORS

While working on your own APIs, you'll typically want to have your Rails server
running while also trying out various endpoints using `fetch()`. In order to do
this, though, you will need deal with [Cross-Origin Resource Sharing][CORS], or CORS.

CORS is designed to prevent scripts like `fetch()` from one origin accessing a
resource from a different origin unless that resource specifically states that
it expects to share. So, for instance, if you have run the command `rails s` and
have your Rails server running at `http://localhost:3000`, then go to
'www.google.com,' open the browser console and attempt to send a fetch to your
server. The browser considers these two different origins, and will _refuse_ your
request.

A solution is already provided though. By using the `--api` flag, the `Gemfile`
was altered to include the [`rack-cors`][rack-cors] gem. The gem will be commented out initially:

```ruby
# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
# gem 'rack-cors'
```

To get `rack-cors` working, uncomment the gem and run `bundle install`. Then, add the following to
`config/application.rb` **within** `class Application < Rails::Application`:

```ruby
class Application < Rails::Application
  ...
  config.middleware.insert_before 0, Rack::Cors do
    allow do
        origins '*'
        resource '*', headers: :any, methods: [:get, :post]
    end
  end
  ...
end
```

This will allow you to test your APIs while developing them locally.
**WARNING:** This allows any requests to be made to your API and is meant for
development. Disabling CORS altogether in the long term will leave your server
unsecure, but with `rack-cors`, it is possible to specify what endpoints and
types of requests your API will allow. Check out the documentation on [CORS] and
[`rack-cors`][rack-cors] for additional information.

**Note:** Secretly, `rack-cors` has been bundled with the last set of lessons to
ensure they were all working smoothly in case you decided to code along and spin
up a rudimentary API.

## Conclusion

With that, you have all that you need to get an API-only Rails build into
development. If you can think of something that can be turned into an API, you
now the power to one up in short order.

Now that we know how to create APIs, we will take a closer look at shaping them.

## Resources

- [Using Rails for API-Only Applications][api]
- [CORS][]

[CORS]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[api]: https://guides.rubyonrails.org/api_app.html
[manual setup]: https://help.learn.co/technical-support/local-environment/mac-osx-manual-environment-set-up
[sqlite]: https://www.sqlite.org/index.html
[rack-cors]: https://github.com/cyu/rack-cors
