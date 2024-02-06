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

# MVC (Model-View-Controller) 

### Generating a Model
```rails generate model Article title:string body:text```

### Database Migrations
```rails db:migrate```

### Showing a List of Articles
app/controllers/articles_controller.rb
```
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end
end
```

app/views/articles/index.html.erb
```
<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <%= article.title %>
    </li>
  <% end %>
</ul>
```

# CRUD (Create, Read, Update, and Delete) 

### Showing a Single Article

config/routes.rb
```get "/articles/:id", to: "articles#show"```

app/controllers/articles_controller.rb
```
 def show
    @article = Article.find(params[:id])
  end
```

create app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

```

### Test
visit http://localhost:3000/articles/1!

### let's add a convenient way to get to an article's page
app/views/articles/index.html.erb
```
<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <a href="/articles/<%= article.id %>">
        <%= article.title %>
      </a>
    </li>
  <% end %>
</ul>
```

### Resourceful Routing
So far, we've covered the "R" (Read) of CRUD. We will eventually cover the "C" (Create), "U" (Update), and "D" (Delete)
==> resources :articles

config/routes.rb
```
root "articles#index"
  resources :articles
```

check: ```rails routes```


###  the article_path helper
returns "/articles/#{article.id}"
app/views/articles/index.html.erb
```
<li>
  <a href="/articles/<%= article.id %>">
    <%= article.title %>
  </a>
</li>
```

### link_to helper
```
<li>
  <%= link_to article.title, article %>
</li>
```

