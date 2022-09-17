# README

### Set UP

1. Clone this template repository
2. Create `.env` file and write your enviromental variables
3. Create `Gemfile.lock`. You don't have to anything, keep empty.
4. Run `docker-compose run api rails new . --api --database=postgresql`
5. After making rails files, edit `database.yml` like below: See `Database Creation`
6. `docker-compose up -d`

### Explanation about Setup Command

- `docker-compose run api rails new . --api --database=postgresql`  
  At this time, we will use `Gemfile` set up manually by me so I don't use the `--force` option on set up command
  If you use other gems, you have to edit `Gemfile`.
  NOTE: **DO NOT FORGET type NO When you are asked overwrite `Gemfile` or not**

- Why using `Dockerfile_DB`?  
  `docker-compose run` means commands are run inside a container and so you might face some permission problems.
  In this case, `docker-compose run api rails new . --api --database=postgresql` cannot make a container run `00_create.sh`. You have to change permissions of the file when the container builds and `Dockerfile_DB` is necessary.
  ```
  ATTENTION!!
  I am facing this issue still.
  Although we cannot create tables and an user when building but you can do that by entering DB container because you already have permitted 00_create.sh file.  
  ```

### Attention while developing

- `bundle install`  
  Since we manage `gem` files in volumes to continuously keep them, we have to use `docker-compose run` when we install gem with bundler.
  The reason why is `docker-compose build` doesn't connect gem files to volumes.

- Annotating routes  
  When you want to see the list of URLs, you can run `bundle exec rake annotate_routes`

- Editing codes  
  If you are already familiar with Rails Project, you will be surprisided at the behavior that any changes aren't reloaded.
  For coding faster, you can add this code in `config/environments/development.rb`
  ```rb
  config.file_watcher = ActiveSupport::FileUpdateChecker
  ```
  When you develop rails with docker, you have to restart container every time you change file such as controllers, but the above code senses change and they will be reloaded

- `Ruby`'s environmental variables  
  - `GEM_HOME`
    When you run `gem install`, the gem is gonna be installd in `GEM_HOME` directory.
  - `BUNDLE_PATH`
    When you run `bundle install`, the gem is gonna be installd in `BUNDLE_PATH` directory.
  - `BUNDLE_BIN`
    If you use `rspec`, this is a kind of *executable file*. the bundler set executable files in `BUNDLE_BIN`.

### Configuration
- how to set up`redis`  
  1. install `redis` and `rails-redis`, which are already written in `Gemfile` but you can specify versions of them if you like.
  2. Since cache is disable in development env by default, you must run `docker-compose run --rm api rails dev:cache`
  3. Create `config/initializers/redis.rb` and add below codes.
    ```rb
    Redis.current = Redis.new
    ```
  4. Edit `config/environments/development.rb`, add some codes and comment out existing codes for cache.
    ```rb
    # Enable/disable caching. By default caching is disabled.
    # Run rails dev:cache to toggle caching.
    # redis 利用のためコメントアウト
    # if Rails.root.join('tmp', 'caching-dev.txt').exist?
    #   config.cache_store = :memory_store
    #   config.public_file_server.headers = {
    #     'Cache-Control' => "public, max-age=#{2.days.to_i}"
    #   }
    # else
    #   config.action_controller.perform_caching = false
    #
    #   config.cache_store = :null_store
    # end

    config.cache_store = :redis_store, "redis://redis:6379/0/cache"
    config.active_record.cache_versioning = false

    ```

### Database creation  
**DO NOT FORGET EDIT `database.yml` after initializing your rails app**
Here is the codes for `database.yml`

```yml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: <%= ENV.fetch("DB_USER") %>
  password: <%= ENV.fetch("DB_PASSWORD") %>

development:
  <<: *default
  database: <%= ENV.fetch("DB_dev") %>


~~~  SKIP  ~~~

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: <%= ENV.fetch("DB_test") %>


~~~  SKIP  ~~~


#   production:
#     url: <%= ENV['MY_APP_DATABASE_URL'] %>
#
# Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full overview on how database connection configuration can be specified.
#
production:
  <<: *default
  database: <%= ENV.fetch("DB_production") %>

```

* Ruby version

* System dependencies


* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions 
