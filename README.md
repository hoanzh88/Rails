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

### Creating a New Article
handled by a controller's new and create
* The new action  --> view
* he create action --> save 

app/controllers/articles_controller.rb
```
def new
  @article = Article.new
end

def create
  @article = Article.new(title: "...", body: "...")

  if @article.save
    redirect_to @article
  else
    render :new, status: :unprocessable_entity
  end
end
```

Let's create app/views/articles/new.html.erb
```
<form action="/articles" accept-charset="UTF-8" method="post">
  <input type="hidden" name="authenticity_token" value="...">

  <div>
    <label for="article_title">Title</label><br>
    <input type="text" name="article[title]" id="article_title">
  </div>

  <div>
    <label for="article_body">Body</label><br>
    <textarea name="article[body]" id="article_body"></textarea>
  </div>

  <div>
    <input type="submit" name="commit" value="Create Article" data-disable-with="Create Article">
  </div>
</form>
```

### Using Strong Parameters
```
  private
    def article_params
      params.require(:article).permit(:title, :body)
    end
```

### Validations and Displaying Error Messages
app/models/article.rb
```
class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```
display any error messages for title and body
app/views/articles/new.html.erb
```
<% @article.errors.full_messages_for(:title).each do |message| %>
  <div><%= message %></div>
<% end %>

<% @article.errors.full_messages_for(:body).each do |message| %>
  <div><%= message %></div>
<% end %>
```

Test: ```http://localhost:3000/articles/new```

### Updating an Article
handled by a controller's edit and update actions.
app/controllers/articles_controller.rb
```
 def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render :edit, status: :unprocessable_entity
    end
  end
```

let's create a very similar app/views/articles/edit.html.erb
```
<h1>Edit Article</h1>

<%= render "form", article: @article %>

```

Let's update app/views/articles/new.html.erb
```
<h1>New Article</h1>

<%= render "form", article: @article %>

```

called a partial.
Let's create app/views/articles/_form.html.erb
```
<%= form_with model: article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
    <% article.errors.full_messages_for(:title).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %><br>
    <% article.errors.full_messages_for(:body).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>

```

app/views/articles/show.html.erb
```
<li><%= link_to "Edit", edit_article_path(@article) %></li>
```

### Deleting an Article
app/controllers/articles_controller.rb
```
  def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to root_path, status: :see_other
  end
```

app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>
```

# Adding a Second Model

### Generating a Model
generate model Comment references model article
```
rails generate model Comment commenter:string body:text article:references
```
```rails db:migrate```

### Associating Models
edit app/models/article.rb
```
class Article < ApplicationRecord
  has_many :comments

  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```

### Adding a Route for Comments
config/routes.rb
```
  resources :articles do
    resources :comments
  end
```
### Generating a Controller
```rails generate controller Comments```

app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>

<h2>Add a comment:</h2>
<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>

```

app/controllers/comments_controller.rb
```
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
```

app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>

<h2>Comments</h2>
<% @article.comments.each do |comment| %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>

  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>

<h2>Add a comment:</h2>
<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>

```

### Refactoring

app/views/comments/_comment.html.erb

```
<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>

<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>

```
app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>

<h2>Comments</h2>
<%= render @article.comments %>

<h2>Add a comment:</h2>
<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>

```

app/views/articles/show.html.erb
```
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>

<h2>Comments</h2>
<%= render @article.comments %>

<h2>Add a comment:</h2>
<%= render 'comments/form' %>
```

# Security
Basic Authentication
app/controllers/articles_controller.rb
```
http_basic_authenticate_with name: "dhh", password: "secret", except: [:index, :show]
```