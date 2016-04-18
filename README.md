# API design guide and conventions
Designing good APIs is key to our growth as we push more things into front-end applications. To that end, we should standardize conventions and follow general good practices while building the APIs.

## TL;DR version:
* [Make your APIs RESTful](#restful)
* [Return useful and meaningful responses](#meaningful-responses)
* [Use HTTP codes effectively](#http-codes)
* [Use SSL everywhere](#ssl)
* [Take time to make responses pretty and understandable](#pretty-responses)
* [Namespace and version the api](#namespacing)
* [Dates](#dates)

## Make your APIs RESTful [#restful] ##

* As much as possible, make your APIs restful.
* Rails makes this super easy by giving you the routes helpers.

Example:
```ruby
# config/routes.rb

resources :member, :only => [:index], :defaults => { :format => 'json' }
resource :brand,   :only => [:show],  :defaults => { :format => 'json' }
```


## Return useful and meaningful responses

* It is important to return responses that make sense.
* The response should be in JSON format and varies depending on the type of request:

Request Type |  Response
--- | ---
GET | the entity corresponding to the requested resource
POST |  the entity that is created
PUT | the entity that is updated

* It is also important to return error messages should something go wrong.

Example:
```ruby
def create
  if member = Member.create(params[:member])
    render :json => member, :status => :created
  else
    render :json => member.errors.messages, :status => :unprocessable_entity
  end
end
```


## Use HTTP codes effectively

* HTTP codes are a great way to indicate the status of the response and should be used properly.
* Use the following with the appropriate requests:

HTTP Code | Rails status symbol | Usage
--- | --- | ---
200 | :ok |Response to a successful GET, PUT, PATCH or DELETE. Can also be used for a POST that doesn't result in a creation
201 | :created | Response to a successful POST request that results in a creation
204 | :no_content | Response to a success DELETE request
400 | :bad_request | When the request is malformed(eg. body cannot be parsed)
401 | :unauthorized | When the request is not authenticated with the proper credentials(eg. client accessing an admin url)
403 | :forbidden | When the requestor is authenticated, but not authorized to request the resource(eg. client accessing another client's brand)
404 | :not_found | When the requested resource is not found
422 | :unprocessable_entity | When there are validation errors


## Use SSL everywhere

* Always use SSL. **No exceptions**.
* Today, your web APIs can get accessed from anywhere there is internet (like libraries, coffee shops, airports among others). Not all of these are secure. Many don't encrypt communications at all, allowing for easy eavesdropping or impersonation if authentication credentials are hijacked.


## Take time to make the responses pretty and understandable

* Make sure that the response keys are all readable and meaningful.
* Denormalize data in the response and nest data as needed so that the response makes sense.
* By convention, use the decorator pattern and JBuilder to reender the json response.
* The benefits of using views here instead of just `render :json => @user.to_builder.attributes!` is that it allows us to do view caching if needed.

Example:
```ruby
# controller
# app/controllers/api/v1/users_controller.rb

module Api::V1
  class UsersController < BaseController
    def show
      @user = UserDecorator.decorate(User.find(params[:id]))
    end
  end
end

# decorator
# app/decorators/user_decorator.rb

class UserDecorator
  decorates :user

  def to_builder
    JBuilder.new do |json|
      json.first_name first_name
      json.last_name last_name
    end
  end
end

# view
# app/views/api/v1/users/show.json.jbuilder

json.user @user.to_builder.attributes!
```


## Namespace and version the API

* Versioning your API is important if you need to make drastic changes to your API.
* One way to version your API is in the url by namespacing your api endpoints

Example:
```ruby
# config/routes.rb

namespace :api, :defaults => { :format => 'json' } do
  namespace :v1 do
    resources :member
  end
end
```

* NOTE: Another way to version your API is in the headers, but this is not the preferred way of versioning at Crowdtap.


## Dates

* Always return dates in ISO 8601 format.

Example:
```ruby
class UserDecorator
  decorates :user

  def to_builder
    JBuilder.new do |json|
      json.first_name first_name
      json.last_name last_name
      json.date_of_birth date_of_birth.strftime("%Y-%m-%d")
    end
  end
end
```
