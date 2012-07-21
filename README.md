This article was forked from [Marshall Huss's Bamboo stack article](https://devcenter.heroku.com/articles/static-sites-on-heroku) and updated by [Lee Reilly](http://www.leereilly.net). Lee is a toolsmith and master pintsman hacking on [GitHub Enterprise](https://enterprise.github.com).

# Static Sites with Ruby on Heroku/Cedar

Sometimes you just have a static website with one or two pages. Here is a simple way to host your static site and cache it on Heroku using a [Rack](http://rack.rubyforge.org/) app.

Your folder should be organized like this:

```
- MySite
  |- config.ru
  |- Gemfile
  |- public
    |- index.html
    |- images
    |- js
    |- css
```

In `Gemfile` file add the following:

```ruby
source :rubygems

gem 'rack'
```

You should use [bundler](https://devcenter.heroku.com/articles/bundler) to generate the `Gemfile.lock` file:

```
GEM
  remote: http://rubygems.org/
  specs:
    rack (1.4.1)

PLATFORMS
  ruby

DEPENDENCIES
  rack
```

In `config.ru` file add the following:

```ruby
use Rack::Static,
  :urls => ["/images", "/js", "css"],
  :root => "public"

run lambda { |env|
  [
    200,
    {
      'Content-Type'  => 'text/html',
      'Cache-Control' => 'public, max-age=86400'
    },
    File.open('public/index.html', File::RDONLY)
  ]
}
```

This assumes that your template uses relative references to the images and stylesheets. Go ahead and deploy the app. If you are not sure how to deploy to Heroku check out the [quickstart guide](https://devcenter.heroku.com/articles/quickstart).

All Heroku apps on the Cedar stack are fronted by [Varnish](http://www.varnish-cache.org/), an [HTTP Accelerator](https://devcenter.heroku.com/articles/http-caching), and we should take advantage of it. You may have noticed in the config.ru file we set the `Cache-Control` header to `86400` seconds (24 hours). This tells the accelerator to go ahead and cache the page. The accelerator already caches any `Rack::File` for 12 hours as well meaning all your stylesheets and images will also be cached.

> A more detailed discussion of HTTP caching can be found in the [Increasing Application Performance with HTTP Caching](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers) article.

To confirm it’s working, let’s take a look at the HTTP headers returned from your site. The `Age` header indicates that it’s being served from the HTTP cache.

```
$ curl -I http://nezumiapp.com/
HTTP/1.1 200 OK
Server: nginx/0.7.67
Date: Sat, 30 Apr 2011 02:26:12 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
X-Runtime: 29
ETag: "a90d1cf7a727aee09abfc7df23bc0b71"
Cache-Control: public, max-age=86400
Content-Length: 3400
X-Varnish: 695131796 695100190
Age: 98
Via: 1.1 varnish
```

And there you go, a static site being served on Heroku completely cached and easily served using a single [dyno](https://devcenter.heroku.com/articles/dynos).

If this article is incorrect or outdated, or omits critical information, please [let us know](https://devcenter.heroku.com/articles/static-sites-on-heroku#). For all other issues, please see our [support channels](https://devcenter.heroku.com/articles/support-channels).