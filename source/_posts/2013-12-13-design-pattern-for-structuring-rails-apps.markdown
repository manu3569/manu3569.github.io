---
layout: post
title: "Design Pattern for Structuring Rails Apps"
date: 2013-12-13 03:03
comments: true
categories: [Rails, Routes, API, Inheritance, Design Pattern]
---

I recently worked on the web application [Wishgram](http://wishgr.am) and its development quickly got more complex when the need for an API arose to support an iOS app.

I'm not a proponent for building controllers that handle both web browser requests and API calls, because it yields a less skinny controller with more complex code when web application and API specific code are mixed.

For example, responses to success and failure actions in web application requests are generally handled with `render` or `redirect_to`, while API requests encapsulate the action outcome in a `JSON` or `XML` response. In a RESTful controller with 6 actions, we are quickly losing the beach body of a skinny controller.

Usually, the login mechanisms for web application and API users differ, where the former typically uses a username or email with a password and the latter tends to use a certificate or an OAuth implementation for granting authentication. This was the case in my project, and handling two authentication methods in the controller added unnecessary complexity. 

Furthermore, the flat structure of controllers &mdash; as produced by the Rails generators &mdash; became too cumbersome to organize with two or more separate areas of application functionality. 

My solution and proposed Rails design pattern is to always group common controllers into modules and to have them inherit from a dedicated group parent controller. Thus, group specific code such as callbacks that would generally reside in the `ApplicationController` can now be applied in isolation to a group in its dedicated parent controller. Even if, at the time of developing a Rails app, there are only plans for a basic web application this pattern should still be used to account for future growth.

<br/>
<br/>
## Implementation of Controller Groups

In my example, we are creating a `Customers` controller for the customer-facing web application, the company-internal back end system and the API.

<br/>
<br/>
### Generate Controllers

```bash
$ rails generate controller FrontEnd
$ rails generate controller FrontEnd/Customers

$ rails generate controller BackEnd
$ rails generate controller BackEnd/Customers

$ rails generate controller API
$ rails generate controller API/V1/Customers
$ rails generate controller API/V2/Customers
```

<br/>
<br/>
### Update Controllers

After generating the controllers in the groups, we change the superclass from `ApplicationController` to `<Group>Controller`.

<br/>
```ruby
# app/controllers/front_end/customers_controller.rb
class FrontEnd::CustomersController < FrontEndController

  def index
    render text: "Welcome to our Website"
  end

end

# app/controllers/back_end/customers_controller.rb
class BackEnd::CustomersController < BackEndController

  def index
    render text: "Admin Panel"
  end

end

# app/controllers/api/v1/customers_controller.rb
class Api::V1::CustomersController < ApiController

  def index
    render json: { customers: [] }
  end

end

# app/controllers/api/v2/customers_controller.rb
class Api::V2::CustomersController < ApiController

  def index
    render json: { customers: [], total_count: 0, request_time: Time.now }
  end
end
```
<br/>

Since all API controllers inherit from `ApiController`, we can easily disable CSRF for API calls to make non-GET valid requests without affecting the behavior in the front and back end controllers. This is a great example to demonstrate the advantage of separate controller groups.

<br/>
```ruby
# app/controllers/api_controller.api
class ApiController < ApplicationController
  protect_from_forgery with: :null_session
end
```

<br/>
<br/>
### Building Routes

Now that we have the controllers separated, we want to separate the 3 different controller groups in individual subdomains thus reflecting a logical separation in the domain name. One bonus with this approach is that cookies won't be shared between our front and back end.

The `constraints` option is used to tie controller groups to subdomains, and the `scope` specifies the `Module` that denotes a controller group without exposing it in the route path.

<br/>
```ruby
#config/routes.rb
require 'api_constraints'

MyApp::Application.routes.draw do

  constraints subdomain: "www" do
    scope module: :front_end do
      resources :customers
    end
  end

  constraints subdomain: "admin" do
    scope module: :back_end do
      resources :customers
    end
  end

  constraints subdomain: "api" do
    scope module: :api do
      scope module: :v1, constraints: ApiConstraints.new(version: 1) do
        resources :customers, controller: "customers"
      end
      scope module: :v2, constraints: ApiConstraints.new(version: 2, default: :true) do
        resources :customers, controller: "customers"
      end
    end
  end
```

<br/>
<br/>
### API Versioning

At this point, we also want to future-proof the API and allow for supplemental versions. [Ryan Bates' REST API Versioning Rail Cast #350](http://railscasts.com/episodes/350-rest-api-versioning) provides a perfect solution. We will integrate it by creating a customized constraint for the API version route `scope`.

<br/>
```ruby
#lib/api_constraints
class ApiConstraints
  def initialize(options)
    @version = options[:version]
    @default = options[:default]
  end
    
  def matches?(req)
    @default || req.headers['Accept'].include?("application/vnd.myapp.v#{@version}")
  end
end
```

<br/>
<br/>
## Final Words

At this time, there is no functionality in the Rails generator to set the inheritance for a controller. I'm currently looking into building a gem to add such a feature to facilitate the use of the proposed Rails design pattern.

[The example application code used in this blog post is on Github](https://github.com/manu3569/mega-rails), so feel free to fork it, and please share your thoughts on the proposed Rails pattern.