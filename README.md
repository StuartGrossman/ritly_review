# Ritly App Review

This readme will cover the setup and coding of the Ritly app as if I were starting from scratch. In other words, without another repo to fork from.

## Objetives
* Deeper understanding of routes
* Depper understanding of models and migrations
* Observing best practices for creating an application:
	* Commit frequently
	* Test the changes you make before you commit
	* Do not commit files that you don't need
	* Setup sublime to use the proper spacing
	
## Sublime Setup

Before we get started on the coding, make sure sublime text is setup correctly to only use 2 spaces for indentation.  In sublime, open up any ruby file, then in the menu go to __Sublime Text > Preferences > Settings - More > Syntax Specific - User__  Add the following configuration to the file that appears and save it:

```
{
	"tab_size": 2,
	"translate_tabs_to_spaces": true
}
```

Now sublime will use the proper settings for ruby files.

## Creating a Github repo

First go to github and create a new repo for this project.  For the current app, I named it `ritly_review`.  The repo should not have anything in it yet.  Let's create a new rails project to put in the repo.  In your terminal type:

```
rails new Ritly -T
```
The `-T` option tells rails not to generate the test directory, which we do not want. Instead, we'll setup rspec after our intial commit. __Do not check in files that you do not plan to use!__

Now that the app is created change into the directory for your app, and setup git:

```
cd Ritly
git init
git add .
git commit -m "First commit on the Ritly app"
```

So far all we have done is created a git repository on our machine.  Next we need to push that code up to github:

```
git remote add origin https://github.com/__username__/ritly_review.git
```

The remote url must correspond to the git repo you have created.  Now, push your change up to the remote:

```
git push -u origin master
```

Now go to github and verify that your changes have been saved.  __You should be commiting to github often!!__

## Initial Database Setup

Next, we'll switch the app to use postgres.  First add the Gem to your `Gemfile` and remove the sqlite3 gem.

```
gem 'pg'
# Delete gem 'sqlite3'
```
Do a `bundle install` so that the updates we just made to the Gemfile are installed.

Next, change the `config/database.yml` file so that postgres is specified.  Also change the names of the databases for each of the 3 modes.  Here is an example file:

```

development:
  adapter: postgresql
  database: ritly_review_development
  host: localhost
  pool: 5
  timeout: 5000

test:
  adapter: postgresql
  database: ritly_review_test
  host: localhost
  pool: 5
  timeout: 5000

production:
  adapter: postgresql
  database: ritly_review_prod
  pool: 5
  timeout: 5000

```

#### Test & Commit

Now that the database is setup, we should be able to create it using rake. Do:

```
rake db:create
```

Verify that the database got created:

```
rails dbconsole
```

You should see a psql prompt with your database name.  Type `\q` to quit.  And commit the changes so for:

```
git add Gemfile Gemfile.lock config/database.yml
git commit -m "Configuring the database to use postgres"
git push origin master
```

#### Questions

* What does `rake db:create` do?
* What file is `rake db:create` referencing to setup the database?

## Initial Setup: Hiding Secrets

There are a few steps that must be followed in order for your application to be secure.

__Step 1__: Ensure that the ```config/initializers/secret_token.rb``` does not have any secret strings in it.  This is used to generate a signed cookie.  If the secret is checked into github, your application could easily be compromised.  Instead, your secret_token.rb file should contain something like the following:

```
CookBook::Application.config.secret_token = ENV['SECRET_KEY_BASE']
```

__NOTE__: In the latest version of rails, the file to edit might be called ```config/initializers/secrets.yml```.  This file is typically already using a ENV hash.

__Step 2__: Add the ```dotenv-rails``` gem to the ```Gemfile```.  The change should look something like this:

```
group :development, :test do
  gem 'dotenv-rails'
end
```

Then, do `bundle install` in the terminal.

More information on the gem is available on the [dot env github page](https://github.com/bkeepers/dotenv).

__Step 3__: __IMPORTANT__ your 	```.gitignore``` file must ignore the file  that will be created on the next step.  __DO NOT CHECK IN A .ENV FILE TO GITHUB!!__  The following line should be added to the ```.gitignore``` file and committed to github:

```
.env
```

__Step 4__: Add a ```.env``` file in the root of the project.  The file contains keys and values.  The keys will be accessible in the ENV hash of rails.  We want the SECRET_KEY_BASE to have a long secret key associated with it.  To generate the secret key, run ```rake secret```.  The ```.env``` file should look something like the follwing.  __NOTE: The following is a sample.  Do not put this exact string into your .env file__:

```
SECRET_KEY_BASE=91599e3db059958b939385e5eb1dd0199e3f94b47ae8b4c1ced283472f0e0f31a5ba1f99bbdc79fd8c9a1f2c1495f181b025f7d3eb39e65c72044500e4f047aa
```

__GENERATE YOUR SECRET WITH THE FOLLOWING:__ To generate the file and the secret string all in one shot, use the follwing command while your current directory is the project's root directory:

```
echo "SECRET_KEY_BASE=`rake secret`" > .env
```

__Step 5__: Force the app to use SSL in production.  In ```config/environments/production.rb```, make sure the following is present:

```
config.force_ssl = true
```

#### Test & Commit

As usual, we've made a change that we should test out then check it in if it looks good.  Run your rails server and make sure the default rails page comes up:

```
rails s
```

If your rails server still runs, we're ready to commit:


```
git add Gemfile Gemfile.lock .gitignore config/initializers/secret_token.rb
git commit -m "Hiding the secrets using a ENV hash. Forcing ssl in production"
git push origin master
```

####Questions

* What does the __.gitignore__ file do?
* Why do you `force_ssl` in production?
* In the Gemfile, we added a gem to `group :development, :test do`.  What are the groups for?  Why do we only want the dotenv-rails gem to development and test?


## Ritly Data Modeling

Since the [ritly specs](https://github.com/wdi-sf-march-2014/ritly_mini_project/blob/master/README.md) say I need a table with `id`, `link`, `random_string`, `created_at` and `updated_at`, I'll focus on the model now.  To create a new model I can use the `rails g model` command:

```
rails g model Url link:text random_string:string --no-test-framework
```
There are a few things to notice here  

* __Datatypes:__ I used `text` for the link column and `string` for the random_string column.  If you are curious about why I chose one versus the other, look at this [stack overflow post](http://stackoverflow.com/questions/3354330/difference-between-string-and-text-in-rails)
* __No test framework__:  I created the model with the `--no-test-framework` option.  This way, files are not created in the test directory.  I'd rather use rspec, which will be the next step.  
* __Files Generated__:  Look at what files were generated: 
	* `app/models/url.rb`
	* `db/migrate/20140502130622_create_urls.rb`
	
We should always check the migration generated to make sure that the changes to the database are correct.  The migration, located at `db/migrate/20140502130622_create_urls.rb` should look like the following. __NOTE: Your file name with be different because the timestamp will be different__:

```
class CreateUrls < ActiveRecord::Migration
  def change
    create_table :urls do |t|
      t.text :link
      t.string :random_string

      t.timestamps
    end
  end
end
```

It looks good.  Now do a migration so that the postgres database gets changed:

```
rake db:migrate
```

Now I want to verify that I can create a Url.  Run `rails c` and make a url:

```
Url.create(link: 'https://www.google.com', random_string: 'goog1')
Url.create(link: 'http://www.yahoo.com', random_string: 'yahoooooo')
Url.create(link: 'http://arstechnica.com/', random_string: 'ars')
```

Now I can do `Url.all` and see that my data has been persisted.  This test data looks good, so I'm going to add it to my `db/seeds.rb` file:

```
Url.create(link: 'https://www.google.com', random_string: 'goog1')
Url.create(link: 'http://www.yahoo.com', random_string: 'yahoooooo')
Url.create(link: 'http://arstechnica.com/', random_string: 'ars')
```

Now that the seed data is created, let's test it out.  The first thing we'd do is get rid of the test data we created on the console:

```
rake db:drop
```
Next, we have to do `rake db:create` to create the database again, then `rake db:migrate` to make the tables in the database.

```
rake db:create
rake db:migrate
```

Now we can run the seed file to populate our new data:

```
rake db:seed
```


__Test And Commit__

Assuming that running `rake db:seed` didn't give any errors, we can go back into `rails c` and verify that the data we seeded is there.  In rails c, type:

```
Url.all
```
Notice the query that gets run and the data that we get back.  If the data looks like what we expect, it's time to commit the changes to git:

```
git add app/models/url.rb
git add db/migrate/20140502130622_create_urls.rb
git add db/schema.rb
git add db/seeds.rb
git commit -m "Adding the Url model, migration and seed data"
git push origin master
```

#### Questions

* What does the rails g model create?
* What is a model?  How is a model different from a table in a database?
* What is active record?
* What does a migration do?
* When does a migration change the database?

## Setting up Rspec

We want to make sure our app is tested so that we have confidence that other peices of the app do not break when we make a change.  Let's add rspec to the project.  First add the rspec gem to the development and test group in the `Gemfile`  While we are here, let's add pry too:

```
group :development, :test do
  gem 'dotenv-rails'
  gem 'pry'
  gem 'rspec-rails'
end
```
__Note__: The group and the dotenv-rails already existed.  No need to add it twice.  Do a bundle install then use the rspec generator

```
bundle install
rails generate rspec:install
```
Now generate a spec file for the model that we created earlier:

```
rails generate rspec:model url
```

Next, setup our test database and run rspec:

```
rake db:test:prepare
rspec
```

You will see a pending spec because we created a model spec for Url but didn't put any specs in it.  This walk through won't focuse on rspec tests, but you should continue to add tests as you go.  Let's add a test to ensure that we can create a Url and that the url has the fields we are expecting.  Modify `spec/models/url_spec.rb` to look like the following:

```
require 'spec_helper'

describe Url do
  it 'should respond to link and random_string' do
    url = Url.create(link: "https://www.google.com", random_string: 'abcdefg')
    url.should respond_to(:link)
    url.should respond_to(:random_string)
  end
end
```

#### Test & Commit

Now run `rspec` and verify that the test passes.  You should continue to add tests as you go and always run rspec before you commit something to make sure your change hasn't caused any issues.

```
rspec
```

Assuming everything looks good, add the files we created or modified to git and push them to github:

```
git add Gemfile Gemfile.lock
git add .rspec spec/.
git commit -m "Adding rspec and a url model test"
git push origin master
```

#### Questions
* What does `rake db:test:prepare` do?
* Why are tests useful?
* When should you run the tests?
* 





