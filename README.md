# Rails
https://guides.rubyonrails.org/getting_started.html


## Create the Docker files
### file name: Dockerfile
```
FROM ruby:3.1.0
WORKDIR /usr/src/app

#COPY . .
#RUN bundle install
```
### file name: docker-compose.yml
```
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
    command: rails s -b 0.0.0.0
```

## Generate a Rails application
```docker-compose run --service-ports web bash```

### Installing Rails
```gem install rails```

### Creating the Blog Application
```rails new .```

### Start the application
```rails s -b 0.0.0.0```
--> http://localhost:3000/

### Install project dependencies when building the Docker image
Press ctrl+c to stop the server
Type exit to exit the container
Type docker-compose run --service-ports web bash to start a new terminal session
Type rails s -b 0.0.0.0 to start the Rails application
--> The terminal will show an error message that it can’t find the command rails.

Dockerfile:
```
COPY . .
RUN bundle install
```
we can now run ```docker-compose build```
```docker-compose up```

# create a controller articles
```rails generate controller Articles index --skip-routes```

app/controllers/articles_controller.rb
```
class ArticlesController < ApplicationController
  def index
  end
end
```

app/views/articles/index.html.erb
```
<h1>Hello, Rails!</h1>
```

config/routes.rb
```
get "/articles", to: "articles#index"
```
Test:
```http://localhost:3000/articles```

