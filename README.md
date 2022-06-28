# Cool stuff I found in ruby

## Data type regular expression

Regex syntax is fairly same and possible delimiters are `/pattern/` and `%r{pattern}`

You can test strings agaisnt regex using the `=~` match operator

i.e. 
```ruby
if line =~ /P(erl|ython)/
  puts "Hello"
end
```

## Logic

`puts "Danger" if radiation > 3000`

`distance = distance * 1.2 while distance < 100`

## Blocks and iterators

`do ... end` and curly brackets `{}` both can be blocks

You can pass block to a method. i.e. `greet { puts 'Hi' }`

and you can call this block method with yield

If you want you can even pass parameters.

## Ruby Idioms

Ruby method names can end with an exclamation mark (a bang method) or a question mark (a predicate method). Bang methods normally do somethiugn destructive to the receiver. Predicate methods return true or false depending on some condition.

```ruby
count += 1
price *= discount
count ||= 0  # same as count = count  || 0
```

### lambda

The `lambda` operator converts a block into an object of type Proc. An alternate syntax, introduced in Ruby 1.9, is ->.

# Rails

## Generate

You can generate model, views, controller, and migration with `scaffold`. Scaffold also creates test units, helpers, jbuilders, and assets(js, css, scss)

## Test

You can unit test models only.

i.e.

<details>
<summary>test/models/product_test.rb</summary>

```ruby
test "product price muts be positive" do 
  product = Product.new(title: "test")
  product.price = -1 
  assert product.invalid?
  assert_equal ["must be greater than or equal to 0.01"],
    product.errors[:price]

  product.price = 0
  assert product.invalid?
  assert_equal ["must be greater than or equal to 0.01"],
    product.errors[:price]
  
  product.price = 1
  assert producct.valid?
end
```
</details>

### Text Fixtures

It's a test model. Same as robodog.

i.e.

<details>
<summary>test/fixtures/products.yml</summary>

```yml
one:
  title: MyString
  description: test
  price: 1
two:
  title: MYSTRING
  description: TSET
  price: 0.1
```
</details>

You can load data of fixtures using directive fixtures(). i.e. `fixtures :products`

## Handling Errors

<details>
<summary>about flash</summary>

Raisl has a convenient way of dealing with errors and error reporting. It defines a structure called a `flash`. A flash is a bucket (actually close to a Hash) in which you can store stuff as you process a request. The contents of the flash are available to the next request in this session before beign deleted automatically. Typically the flash is used to collect error messages. i.e. when our show() method detects that it was passed an invalid ID, it can store that error message in the flash area and redirect to the index() action to redisplay it at the top of the catalog page. The flash information is accessible within the views by using the flash accessor method.
</details>


<details>
<summary>Test cart controller</summary>

```rb
class CartsController < ApplicationController
  rescue_from ActiveRecord::RecordNotFound, with: :invalid_cart
  #
  private
  def invalid_cart
    logger.error "Attempt to access invalid cart #{params[:id]}"
    redirect_to store_url, notice: 'Invalid cart'
  end
end
```
</details>

## Atom Feeds


<details>
<summary>About atom</summary>

There are a number of different feed formats, most notably RSS 1.0, RSS 2.0, and Atom, standardized in 2000, 2002, and 2005, respectively. These three are all widely supported. To aid with the transition, a number of sites provide multiple feeds for the same site, but this is no longer necessary, increases user confusion, and generally is not recommended.

 The Ruby language provides a low-leve library, which can produce any of these formats, as well as a number o ohter less common versions of RSS. For best results, stick with one of the three main versions.

 The Rails framework is all about picking reasonable defaults and has chosen Atom as the default for feed formats. It is specified as an Internet standards-track protocol for the Internet community by the IETF, and Rails provides a higher-level helper named atom_feed that takes care of a number of details based on knowledge of Rails naming conventions for things like IDs and dates.
</details>

## Integration Testing of Applications


<details>
<summary>Before coverage</summary>

`Unit testing` of models Model classes contai5n business logic. For example, when we add a product to a cart, the cart model class checks to see whether that product is already in the cart's list of items. If so, it increments the quantity of that item; if not, it adds a new item for that product.

`Functional testing of controllers` Controllers direct the show. They receive incoming web requests, interact with models to gather application state, and then respond by causing the appropriate view to display something to the user. So when we're testing controllers. we're making sure that a given request is answered with an appropriate response.

</details>

To generate integration test you need to use `rails generate integration_test user_stories`.

In the test file you can use keys below to test the integration.

get, assert_response, assert_template, xml_http_request, assert_equal, post_via_redirect


<details>
<summary>test/integration/user_stories_test.rb</summary>

```rb
require 'test_helper'

class UserStoriesTest < ActionDispatch::IntegrationTest
  fixtures :products

  test "buying a product" do
    Order.delete_all
    ruby_book = products(:ruby)

    get "/"
    assert_response :success
    assert_template "index"

    xml_http_request :post, '/line_items', product_id: ruby_book.id
    assert_response :success

    cart = Cart.find(session[:cart_id])
    assert_equal 1, cart.line_items.size
    assert_equal ruby_book, cart.line_items[0].product

    get "/orders/new"
    assert_response :success
    assert_template "new"

    post_via_redirect "/orders",
      order: { name: "test"}
    assert_response :success
    assert_template "index"
  end
end
```

</details>

## Internationalization - Locale


<details>
<summary>config/initializers/i18n.rb</summary>

```rb
#encoding: uf-8
I18n.default_locale = :en

LANGUAGES = [
  ['English', 'en'],
  ['Mongolian', 'mn']
]
```

</details>


<details>
<summary>app/controllers/application_controller.rb</summary>

```rb
class ApplicationController < ActionController::Base
  before_action :set_i18n_locale_from_params

  #
  protected
    def set_i18n_locale_from_params
      if params[:locale]
        if I18n.available_locales.map(&:to_s).include?(params[:locale])
          I18n.locale = params[:locale]
        else
          flash.now[:notice] = "#{params[:locale]} translation not available"
          logger.error flash.now[:notice]
        end
      end
    end

    def default_url_options
      { locale: I18n.locale }
    end
end
```

You put your locales in `config/locales` naming them `en.yml` and `mn.yml`

</details>

## Deployment and Production

1. Configure machine

```sh
gem install buidler
bundle install
rake about
rake test
rake server
```

2. Install apache

1. isntall Passenger

### Deploying application remotely 

To add necessary fiels to the project for Capistrano to do its magic, execute the following command:

`capify .`

Capfile
```Capfile
load 'deploy'
# load 'deploy/assets'
load 'config/deploy'
```

<details>
<summary>Cap commands</summary>

The first tiemm we deploy our application, we have to perform an additional step to set up the basic directory structure to deploy on the server.

`cap deploy:setup`

When we execute this command, Capistrano will prompt us for our server's password. If it faisl to do so and faisl to log in, we might need to uncomment out the `default_run_options` lien in our `deploy.rb` file and try agai5n. Once it can connect successfully,it will make the necessary directories. After this command is done, we can check out the configuration for any other problems.

`cap deploy:check`

As before, we might need to uncomment out and adjust the `default_environment` lines in our `deploy.rb`. We can repeat this command until it completes successfully, addressing any issues it may identify.

One last task: we load the "seed" data containing our products.

`cap deploy:seed`

At this point, we should be off to the races.

`cap deploy:rollback`

For some reason we need to step back in tiem and go back to a previous version of our application, we can use this.
  
</details>

## Rails documenting

Rails makes it easy to run Ruby's [RDoc](http://rdoc.sourceforge.net) utility on all the source files in an application to create good-looking programmer documentation. But before we generate that documentation, we should probably create a nice introductory page so that future generatiosn of developers will know what our application does.

To do this, edit the fiel README.rdoc, and enter anything you think might be useful. This file will be processed using RDoc, so you have a fair amount of formatting flexibility.

You can generate the documentation in HTML format using the rake command.

`rake doc:app`

This generates documentation into the directory `doc/app`.

Finally, we might be interested to see how much code we've written. There's a Rake task for that, too.

`rake stats`

<details>
<summary>in addition</summary>

`doc:rails` will provide documentation for the version of Rails you are running.

`doc:guides` will provide usages guides. Before you build the guides, you will need to add the gem `redcarpet` to your Gemfile and run `bundle install`.

Rails also provides other document-related tasks. To see them all, enter the command `rake -T doc`.

</details>

## Depth

### Scripts

`bin` directory is the place to put wrappers that call those scripts. You can use `bundle binstubs` to populate this directory.

This directory also holds the Raisl script. This is the script that is run when you run the `rails` command line. The first argument yuou pass to that script determines the function Raisl will perform.

`console` - Allows you to interact with your Raisl application methods.

`dbconsole` - Allows you to directly interact with your databsae via the command line.

`destroy` - Removes autogenerated field created by generate.

`generate` - A code generator. Out of the box, it will create controllers, mailers, models, scaffolds, and web services.

`new` - Generates Rails application code.

`runner` - Executes a method in your application outside the context of the Web. This is the noninterative equivalent of `rails console`. You could use this to invoke cache expiry methods from a `cron` job or handle incoming email.

`server` - Runs your Rails appliation in a self-contai5ned web server, usiugn Mongrel or WEBrick. We've been using this in our Depot application durign development.

### Naming Conventions

We often name variables and classes usign short phrases. In Ruby, the convention is to have variable names where the letters are all lowercase and words are separated by underscores. Classes and modules are named differently: there are no underscores, and each word in the phrase is capitalized. These conventions lead to variable names such as `order_status` and class names such as `LineItem`.

Rails takes this convention and extends it in two ways. First, it assumes that database table names, such as variable names, have lowercase letters and underscores between the words. Rails also assumes that table names are always plural. This lead to table names such as `orders` and `third_parties`.

On another axis, Rails assumes that files are named using lowercase with underscores.

<details>
<summary>Why Plurals for Tables?</summary>

Because it sounds good in conversatiosn. Really. "Select a Product from products." And "Order has_many :line_items."

The intent is to  bridge programming and conversation by creating a domain language that can be shared by both. Having such a language means cutting down on the mental translation that otherwise confuses the discussion of a product description with the client when it's really implemented as merchandise body. These communications gaps are bound to lead to errors.

Rails sweetens the deal by giving you most of the configuration for free if you follow the standard conventions. Developers are thus rewarded for doing the right thing, so it's less about giving up "your ways" and more about gettign productivity for free.

</details>

### Active Record

Active Record is the object-relational mapping (ORM) layer supplied with Rails. 

#### Organizing Using Tables and Columns

Each subclass of `ActiveRecord::Base`, such as our `Order` class, wraps a separate database table. By default, Active Record assumes that the name of the table associated with a given class is the plural form of the name of that class. If the class name contains multiple capitalized words, the table name is assumed to have underscores between these words.

These rules reflect Rails' philosophy that class names should be singular whiel the names of tables should be plural.

Although Raisl handles most irregular plurals correctly, occasionally you may stumble across one that is not handled correctly. If you encounter such a case, you can add to Rails' understaning of the idiosyncrasies and inconsistencies of the Englilsh language by modifying the inflection file provided which can be located in `config/initializers/inflections.rb`.

If you have legacy tables you have to deal with or don't like this behavior, you can control the table name associated with a given model by setting the `table_name` for a given class.

```rb
class Sheep < ActiveRecord::Base
  self.table_name = "sheep"
end
```

Moreover you can get information about tables by using `rails console`

`Order.column_names` - get `order` table's column names.

Order.column_hash["pay_type"] - gets the detail about `pay_type` column.

We can see the mapping between SQL types and their Ruby representation in the following table. Decimal and Boolean are slightly tricky.

| SQL Type  | Ruby Class  |
|---|---|
| int, integer | Fixnum |
| float, double  | Float |
| decimal, numeric | BigDecimal |
| char, varchar, string | String |
| interval, date | Date |
| datetime, time | Time |
| clob, blob, text | String |
| boolean | See text |
