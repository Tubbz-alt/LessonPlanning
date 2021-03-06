# Advanced Web Application Topics

## Consuming Web API's
API's (Application Programming Interface) are the languages that software uses. When we use Classes and Methods in Ruby, we are using Ruby's API. We can write an API for web applications, which would allow for people and other software to use the resources within our application. There are many uses for web api's, a common one would be iPhone apps, the native iphone app will use a web API to get, create, modify, and delete resources in a web application. The Twitter app would be a very common instance of this.

### Structure
A good Web API will be structured in a RESTful way. Meaning that it's focused around resources, which fits perfectly with Rails and object oriented programming. Web API's act in the same way as a standard request from our browser, but typically for ease of consumption the response is in a standard data format other than HTML, such as XML or JSON. A common set of API endpoints for a given resource (users in this case) might look something like

- GET /users (gets a collection of users)
- GET /users/1 (gets a single user)
- POST /users (creates a user)
- PUT /users/1 (modifies an existing user)
- DELETE /users/1 (deletes an existing user)


### Data format
The response of a web API can be in many formats but we will focus on JSON. JSON looks very much like a Ruby Hash, so it's quite easy and familiar. The data returned from an API can be in any format and many application structure the data in different ways. In the example above here is what the responses might look like

#### GET /users

    {
      users : [
        {id : 1},
        {id : 2}
      ]
    }
The data returned is pretty simple. As described in our documentation, we can see that the root of the data is a JSON Hash, that contains a key `users`, and a value which is an `Array`. The `Array` contains two resources. Given these resources we could now make a GET request for a specific resource.
#### GET /users/1

    {
      user : {id : 1, name: "Ada Lovelace", age: 184, created_at: "Jan 1 2013"}
    }
    
Some API responses may just be HTTP status codes, for example if we are using the `POST /users` method, since it's a user we are creating we will already know of the attributes, we most likely only care if the request was successful (200) or not (500).

### Documentation

An API should have public documentation outlining the available resources, required and optional parameters, and how to handle the response. Learning how to properly read the documentation for any API is the first step to successfully using it. Here is an example document for our fake API we used above:

#### Resource GET /users
- Returns a list of user resource objects with a root key of "users". A maximum of 20 results will be returned ordered by the most recently created. 
- ***Required Parameters***: -
- ***Optional Parameters***: *page* (must be an integer greater than 0). 
- ***Resource Attributes***: Each resource will contain the attributes: [`id`]


#### Resource GET /users/:id
- Returns a single user resource with a root key of "user".
- ***Required Parameters***: `id` must be included in the URL, must be an attribute matching a existing resource.
- ***Optional Parameters***: -
- ***Resource Attributes***: The resource will contain the attributes: [`id, created_at, name, age`]

### HTTP Libraries
Ruby has a few libraries to help us with external HTTP requests. The standard library included with Ruby is `Net/HTTP`, but we will look at a more simple library [HTTParty](https://github.com/jnunemaker/httparty "HTTParty"). HTTParty would be a good option for consuming a simple weather API [openweathermap.org](http://openweathermap.org/wiki/API/2.1/JSON_API#Find_City_via_name).

    require 'httparty'
    require 'json'
    
    response = HTTParty.get("http://api.openweathermap.org/data/2.1/find/name?q=seattle")
    response.success # => true
    response.code    # => 200
    weather_data = JSON.parse(response.body)
    weather_data["list"].first["weather"].first["description"] # => "Sky is Clear"
    
The raw JSON returned from the request is:

    {
      "message":"",
      "cod":"200",
      "type":"base",
      "calctime":"",
      "units":"internal",
      "count":1,
      "list":[
        {"id":5809844,
        "coord":{"lat":47.606209,"lon":-122.332069},
        "name":"Seattle",
        "main":{
          "temp":296.43,"pressure":1017,"humidity":49,"temp_min":294.15,"temp_max":298.71
        },
        "dt":1371936039,
        "date":"2013-06-22 21:20:39",
        "wind": {"speed":2.1,"deg":40},
        "clouds":{"all":1},
        "weather":[{"id":800,"main":"Clear","description":"Sky is Clear","icon":"01d"}],
        "sys":
          {"country":"US","population":608660},
          "url":"http:\/\/openweathermap.org\/city\/5809844"}
        ]
      }
      
### Gems
Many Gems exist to utilize specific public and private API's, often called "wrappers". For example the [Twitter](https://github.com/sferik/twitter) gem to make consuming Twitter's API as easy as `Twitter.home_timeline`. Most likely if there is a service that you'd like to consume, a gem already exists for it.

## Advanced Forms

HTML forms can seem very simple at times, a single text field that submits to a resource is quite easy to grasp, but some of the nuances of forms can be quite complicated. We will look at advanced forms and specifically how they relate to Rails `ActiveResource`. We will be talking about:

* Input Types
* - text
* - textarea
* - submit
* - file
* - checkbox
* - radio
* - label
* - select
* Multiple fields with the same identifier.
* Nested Resources
* Serialized Data

### Forms role with Rails

First we'll have a quick recap of why HTML forms are so important and how they relate to our resources within Rails. When form fields are submitted to rails, rails will turn the fields into a Ruby `Hash` for use in our controllers assigned to the params attribute. a HTML input that looks like 

`<input name="email">` 

when submitted to the rails application will result in 

`{params: {email: <input value>}}`

We can nest HTML field attributes in `[]` to create `Hash` objects. The form to update a typical rails resource would usually look something like:

    <input name="user[email]">
    <input name="user[username]">

Which would translate to:

    {params {user: {email: <email value>, username: <username value>}}}
    
Not by coincidence, this is the exact syntax that would be used to create or update a resource:

    User.create(params[:user])
    
Now we can see how useful knowing the nuances of forms can be. We can manipulate our resources in any way that the `ActiveRecord` create/update will allow.

### Multiple fields with the same identifier
There are a couple scenarios when we would need multiple form inputs to have the same identifier. One of the most common is in the use of checkboxes. When the user does not input a value, the field is not submitted to the server at all, for instance an unchecked checkbox. Sometimes we may want to know if a checkbox has been unchecked. For this we will use one input with `type=hidden`   and one with `type=checkbox` with the same `name` attribute

    <form action="/" method='POST'>
      <input type="hidden"   value=0   name="user[subscribe_to_email_list]">
      <input type="checkbox" value=1   name="user[subscribe_to_email_list]" id="subscribe_to_email_list">
      <label for='subscribe_to_email_list'>Subscribe to our email list</label>
      <input type="submit">
    </form>
    
In this example the subscribe_to_email_list value will be 0 if the box is not checked and 1 if the box is checked. If the hidden input field were not there, subscribe_to_email_list would not be submitted with the form at all, and if we expected that unchecking this box removed the user from the mailing list if they we're already subscribed, that information would not be passed along to the server.

Although I feel this concept is important to grasp to fully understand rails relationship with HTML forms, this problem is often inconsequential because rails deals with it automatically. The [`check_box`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-check_box) rails helper allows us to specify unchecked value, and the method we described is exactly what the helper will implement in our generated HTML. 

    <%= form_for @user do |f| %>
      <%= f.check_box :subscribe_to_email_list %>
      <%= f.label :subscribe_to_email_list %>
      <%= f.submit %>
    <%= end %>
    
This `erb` will generate the equivilant HTML, hidden field and all.

### Nested Resources

Rails resources can be nested, for instance when a `Cart` has many `Item` objects. Basic associations are quite easy to deal with, but what about when we want to update a set of resources that belong to a single resource. Like changing the quantity of each `Item` that belongs to a `Cart`. In our parent class we can specify that we want the ability to do this:

    class Cart < ActiveRecord::Base
      has_many :items
      accepts_nested_attributes_for :items
    end

Suppose we have 2 existing resources

      Cart
          - id: 1
      Item
          - cart_id: 1
          - id: 2
          - quantity: 1
        
          
This give the ability to do something like:

    Cart.find(1).update_attributes(items_attributes: {"0" => {id: 2, quantity: 2}, "1" => {quantity: 3}})
    
To update the `Cart` children. The `items_attributes` value is a collection of `Item` attributes. `{id: 1, quantity: 2}` will update the `Item` with an id of `2`, to have a `quantity` of `2`. If the `items_attributes` does not contain an `id` key (just `{quantity: 3}`), a new `Item` record will be created with an `quantity` of `3` and a `cart_id` of `1`.

So how do we accomplish this with HTML? It's quite easy. To create the same Hash for `params[:cart]` our HTML would be:

    <form action="/cart" method="POST">
      <input type="hidden" name="cart[items_attributes][0][id]" value=2>
      <input type="text"   name="cart[items_attributes][0][quantity]">

      <input type="text"   name="cart[items_attributes][1][quantity]">
      <input type="submit">
    </form>
  
Using Rails helpers we would generate this HTML using the `fields_for` helper

    <%= form_for @cart do |f| %>
      <%= f.fields_for :items do |ff| %>
        <%= ff.text_field :quantity %>
      <% end %>
      
      <%= f.fields_for :items, Item.new do |ff| %>
        <%= ff.text_field :quantity %>
      <% end %>
      
    <% end %>

Notice that the second `fields_for` block has two arguments, `:items` and `Item.new`, `:items` tells the form what the nested fields resource is called. If a second argument is not added Rails will assume the the fields are for the objects associated with that name, `Item`. If there is a second attribute, rails will assume that you are creating fields for the object.

### Serialized Data
Rails can serialize more complex data types in the database in in text column. For instance if we want to store an `Array` of tags to blog `Post` objects, we can add a tags column to our table in the database, and serialize an array to as `YAML` text blob.

    class Post < ActiveRecord::Base
      serialize :tags, Array
    end
    
Then we can do things like

    post = Post.last
    post.tags = ["running", "hiking", "footwear"]
    post.save
    post.tags # => ["running", "hiking", "footwear"]
    
The text value actually stored in the database would actually be

    "---\n- running\n- hiking\n- footwear\n"

But the magic `serialize :tags, Array` tells the model to convert the object assigned to the `tags` attribute to `YAML`, and when the attribute is accessed next the `YAML` text is converted to a Ruby `Array`. This assumes that the stored `YAML` is in a valid format to be converted to Ruby, if it is not an exception will be raised.

So how do we make an array in HTML forms? So far we've only seen the equivalent of hashes. We use an empty `[]`

    <input type='text' name="post[tags][]">
    <input type='text' name="post[tags][]">
    <input type='text' name="post[tags][]">
    
This would create 3 text fields for tags, the submitted value would be translated to
    
    {post: {tags: [<tag>, <tag>, <tag>]}}
