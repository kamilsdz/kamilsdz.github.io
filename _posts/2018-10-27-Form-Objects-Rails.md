---
layout: post
title: Form Objects on Rails
---

Form objects separated into a new classes are great for separating views from business logic. They are especially useful when we update one model, for example user details, on a few steps.

So, let's build a new app with this approach!

```
rails new test-app
cd test-app
rails db:create
rails generate scaffold Product name:string
rails db:migrate
rails s
```

By the way, I love Rails 'Convention Over Configuration' - great for building test applications.

![_config.yml]({{ site.baseurl }}/images/posts/form-objects/yay.png)
{: style="text-align: center;"}

First of all, set root path on 'routes.rb':
```ruby
Rails.application.routes.draw do
  root 'products#index'
  resources :products
end
```
and refresh the page:

![_config.yml]({{ site.baseurl }}/images/posts/form-objects/root.png)
{: style="text-align: center;"}

Everything should work - we can create new product and edit its 'name' attribute.

Let's add form object to this! 
I will use my gem to this, but feel free to write the form object class from scratch.

Add this line to your Gemfile:
```ruby 
gem 'active_objects'
```
and then:
```
bundle
```

We need a new directory for our form objects, I will use 'app/forms':
```
mkdir app/forms
```
You should also remember to inform rails about this directory. You can do this easily on 'config/application.rb' by adding this line:
```ruby
config.autoload_paths << Rails.root.join('app/forms/**/')
```
Reset your server to load new path. 

It's the right time to think about the strategy - our project will be still grow up and maybe it will be good if we start to properly arrange and name our classes from very beginning.

I see two ways:

#### 1) By models 
Where we will create a structure similar to the structure of models. For example:
```
.
+-- forms
|   +-- product
|   	+-- form_object.rb
|	+-- pricing_form_object.rb
|	+-- attributes_form_object.rb
```
#### 2) By controllers
```
.
+-- forms
|   +-- products
|       +-- update_form.rb
|       +-- create__object.rb
```

Which one is better? It depends from model-controller relation - look at these examples:

A)
```
.
+-- forms
|   +-- user
|       +-- form_object.rb
```
B)
```
.
+-- forms
|   +-- baskets
|       +-- step_one_form.rb
|       +-- step_two_object.rb
|	+-- step_three_object.rb
```

In the first example (A) we can use one form object dedicated for one model. We can validate there for example email and user name, and these validations will be the same in the entire application. 
In second example the controller's structure was used. Imagine that at the first step of basket we require from user to provide personal information, on second step - his address and on the third step - phone number. What's more, everything lands in one model. It's problematic to valid data because, in a traditional rails-way, we have to do something like this in Basket model:
```ruby
attr_accessor :step

with_options if: proc { |basket| basket.step == :one } do
  validates :first_name, :last_name, presence: true
end

with_options if: proc { |basket| basket.step == :two } do
  validates :city, :street, presence: true
end

with_options if: proc { |basket| basket.step == :three } do
  validates :phone, presence: true
end
```
It will work, but we have to be honest - it's not pretty and our models are getting fat.
For me, controllers structure is more clean and becouse of DRY - common parts can be moved to a common class, for example to 'forms/common/product/form_object.rb'

Let`s create an example form object 'app/forms/product/form_object.rb':

```ruby
class Product::FormObject < ActiveObjects
  validates :name, presence: true
  validates :name, length: { minimum: 3 }

  private

  def object_class
    Product
  end

  def permitted_attributes
    %i[name]
  end
end

```
Not so hard, right?

We have an inheritance from the ActiveObjects class, which is our base, we also create two private instance methods:
* object_class - which contains the original model class - Product
* permitted_attributes - which contains array of attributes of our form. In my example - we have only one attribute - 'name'.
Thanks to them ActiveObject will create an initializer and attr_accessor's for us.

In the form object you have access to all [ActiveModel validation](https://api.rubyonrails.org/classes/ActiveModel/Validations.html), so feel free to use them! 

Next, we have to edit the controllers and views:
app/controllers/products_controller.rb
```ruby
class ProductsController < ApplicationController
  before_action :set_product, only: [:show, :edit, :update, :destroy]

  def index
    @products = Product.all
  end

  def new
    @product_form = ::Product::FormObject.new
  end

  def edit
    @product_form = ::Product::FormObject.new(@product)
  end

  def create
    @product_form = ::Product::FormObject.new(product_params)
    if @product_form.save
      redirect_to products_path
    else
      render :new
    end
  end

  def update
    @product_form = ::Product::FormObject.new(@product)
    if @product_form.update(product_params)
      redirect_to products_path
    else
      render :edit
    end
  end

  private
    def set_product
      @product = Product.find(params[:id])
    end

    def product_params
      params.require(:product_form_object).permit(:name)
    end
end
```
And our views:
app/views/products/edit.html.erb:
```erb
<h1>Editing Product</h1>

<%= form_for(@product_form, url: product_path(@product), method: :patch) do |form| %>

  <div class="field">
    <%= form.label :name %>
    <%= form.text_field :name %>
    <small><%= form.object.errors[:name]&.first %></small>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>


<%= link_to 'Show', @product %> |
<%= link_to 'Back', products_path %>
```

app/views/products/new.html.erb:
```erb
<h1>New Product</h1>

<%= form_for(@product_form, url: products_path, method: :post) do |form| %>

  <div class="field">
    <%= form.label :name %>
    <%= form.text_field :name %>
    <small><%= form.object.errors[:name]&.first %></small>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>

<%= link_to 'Back', products_path %>
```

### voila!

![_config.yml]({{ site.baseurl }}/images/posts/form-objects/form.png)
{: style="text-align: center;"}

Our forms are powered by dedicated form object and our application has gained more isolation.
Last, but not least - quick comparision: 

### Benefits
* Validation works only when we need it
* It's easier to validate an object created in a few steps
* Our application has more separated logic from the views
* Slimmer models

### Disadvantages
* More complex code
* We have to be careful - let's try not to mix the validations in the model and in the form object
* In form_for, we need to specify the HTTP method
* When we create form object for the form - we should be consistent when validating other forms

We also should remember that if we use form object for the form - we should be consistent when validating other forms of this object. This means that if you use form object - you should not use validation on the model side.

