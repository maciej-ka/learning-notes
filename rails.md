Hotwire Screencast
==================
https://d1d6azhz7lc2s3.cloudfront.net/hotwire-screencast.mp4

### Hotwire stands for "HTML Over The Wire"
Turbo: The core of Hotwire, which speeds up page navigation and updates by sending HTML instead of JSON.  
Turbo Drive: Enhances navigation by intercepting links and form submissions to prevent full page reloads.  
Turbo Frames: Allows partial page updates by replacing parts of the DOM.  
Turbo Streams: Enables real-time updates over WebSockets.  
Stimulus: A modest JavaScript framework that complements Turbo by adding interactivity.

### Stimulus is a lightweight JavaScript framework
It's designed to enhance static or server-rendered HTML by adding small bits of interactivity. =  
Easy Integration: Works seamlessly with server-rendered HTML without restructuring your HTML or writing complex JavaScript.  
Convention Over Configuration: Uses simple naming conventions to connect JavaScript classes to HTML elements.  
Lifecycle Callbacks: Provides hooks like initialize, connect, and disconnect to manage component behavior.

### Choose
#### Choose React if:
You need a highly interactive and dynamic user interface.  
You're comfortable with JavaScript and interested in client-side development.  
You want access to a large ecosystem of tools and libraries.

#### Choose Hotwire/Stimulus if:
You prefer server-side rendering and want to minimize JavaScript in your application.  
You're working with frameworks like Ruby on Rails.  
You want a simpler approach to add interactivity to your web pages without the overhead of a full SPA framework.

### some rails commands
```bash
rails g migration AddReadyForInterviewToSkills ready_for_interview:boolean
```

```bash
rails new hired.dev --master
rails new hired.dev --master --css tailwind
rails g scaffold offer link:string title:string company:string description:text salary_from:integer salary_to:integer
rails g scaffold offer link:string title:string company:string description:text
rails g migration add_salary_range_to_offer salary_from:integer salary_to:integer
rails g scaffold skill name:string
rails g model OfferSkill offer:references skill:references required:boolean
rails g migration add_is_ready_to_skill is_ready:boolean default:false
```

### Stimulus
Keep your code from devolving into “JavaScript soup.  
https://stimulus.hotwired.dev/handbook/introduction  
https://stimulus.hotwired.dev/reference

### hello, stimulus
src/controllers/hello_controller.js:7

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    console.log("Controller connected to:", this.element)
  }
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

index.html

```html
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

### targets
mark important elements as "targets"  
use camelCase for name
```html
<input data-hello-target="name" type="text">
```

target can be not input, as long as:  
- it has `value` property
- it has `select()` method

```javascript
static targets = ["name"]

greet() {
  console.log(`Hello, ${this.nameTarget.value}`)
}
```

For each target name in the array  
three properties are created in controller:

```javascript
this.nameTarget: first
this.nameTargets: array of all
this.hasSourceTarget
```

https://stimulus.hotwired.dev/reference/targets

if element is added or removed from controller element:

```javascript
static targets = [ "item" ]

itemTargetConnected(element) {
  console.log(this.itemTargets)
}

itemTargetDisconnected(element) {...}
```

### default actions
```html
<button data-action="clipboard#copy">
```
is same as
```html
<button data-action="click->clipboard#copy">
```

| element           | default action |
|-------------------|----------------|
| a                 | click          |
| button            | click          |
| details           | toggle         |
| form              | submit         |
| input             | input          |
| input type=submit | click          |
| select            | change         |
| textarea          | input          |

### prevent default
```html
<a href="#" data-action="clipboard#copy">Copy to Clipboard</a>
```

prevent default

```javascript
copy(event) {
  event.preventDefault()
  navigator.clipboard.writeText(this.myTarget.value)
}
```

### class names
#### progressive enhancement
deliver basic html and css  
upgrade if browser supports it

```javascript
export default class extends Controller {
  static classes = [ "supported" ]

  connect() {
    if ("clipboard" in navigator) {
      this.element.classList.add(this.supportedClass)
    }
  }
```

html/css

```html
<div
  data-controller="clipboard-progressive"
  data-clipboard-progressive-supported-class="clipboard--supported">

.clipboard-button { display: none; }
.clipboard--supported .clipboard-button { display: initial; }
```

### lifecycle methods

| method       | when                                        |
|--------------|---------------------------------------------|
| initialize() | once ever                                   |
| connect()    | anytime controller is connected to DOM      |
| disconnect() | anytime controller is disconnected from DOM |

### state values
state lives as attributes in the DOM  
controllers are largely stateless

use data-* attributes to store values

```javascript
<div data-controller="slideshow" data-index="1">
domElement.dataset.index
domElement.dataset.index = 21
```

but this way its always stored as string  
stimulus has a tool to autoconvert

```javascript
export default class extends Controller {
  static values = { index: Number }

  previous() {
    this.indexValue--
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.indexValue
    })
  }
```

#### on change callbacks
called once at initialization  
and on every change

```javascript
indexValueChanged() { ... }
myNameValueChanged() { ... }
```

#### default values

```javascript
static values = {
  index: Number,
  effect: { type: String, default: "kenburns" }
}
```

### work with external resources
sometimes we need issue http request  
and respond as the request state changes

or we may want to start a timer  
and stop it when controller is no longer connected

#### loading fragments of html
fetched from the server

```javascript
export default class extends Controller {
  static values = { url: String }

  connect() {
    this.load()
  }

  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
}
```

### action parameters
define parameter

```html
<a
  href="#"
  data-content-loader-url-param="/messages.html"
  data-action="content-loader#load">
  Messages
</a>
```

access it

```javascript
export default class extends Controller {
  load({ params }) {
    fetch(params.url)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
```

### Installing
a. `npm install @hotwired/stimulus`  
b. `<script type="module">`

#### Rails with import map
Rails will auto load all controllers from  
`app/javascript/controllers`

#### naming
| controller file name          | identifier       |
|-------------------------------|------------------|
| date_picker_controller.js     | date-picker      |
| users/list_item_controller.js | users--list-item |

### Hotwire
sending html over the wire  
https://hotwired.dev/  
https://d1d6azhz7lc2s3.cloudfront.net/hotwire-screencast.mp4

### turbo
gives SPA experience  
no javascript

### stimulus
when you need a bit of javascript

```bash
rails new chat --skip-javascript
gem 'hotwire-rails'
bundle
```

a) importmap

```bash
gem 'importmap-rails'
bundle
rails importmap:install
```

b) package.json

```bash
npm init -y
```

then

```bash
rails hotwire:install

rails g scaffold room name:string
rails g model message room:references content:text
rails db:migrate

...
rails server
http://localhost:3000
```

### Turbo frames
reload only part of page

turbo frames decomposes pages  
into independent contexts  
which can be lazy loaded

```erb
<%= turbo_frame_tag "room" do %>
  <%= render "form", room: @room %>
<% end %>
```

way to see turbo frames  
app/assets/stylesheets/application.css

```css
turbo-frame {
  display: block;
  border: 1px dashed darkgrey;
}
```

if links in turbo frame stop to work  
because they don't have matching fram  
break out of frame
```erb
<%= link_to "Back to rooms", rooms_path, 'data-turbo-frame': '_top' %>
```

#### frame for new room message
```erb
<%= turbo_frame_tag "new_message", src: new_room_message_path(@room), target: "_top" %>
```

wrap app/views/messages/new.html.erb with

```erb
<%= turbo_frame_tag "new_message", target: "_top" do %>
<% end %>
```

### how it works
Rails knows when request is made from frame  
and doesn't render layout in such case

### turbo streams
they deliver page changes  
over a socket  
or in response to form submitions

dom changes:  
append  
prepend  
replace  
remove

*if you need more then dom change*  
*connect to stimulus controller*

#### fix chat clearing room's name form

```ruby
respond_to do |format|
  format.turbo_stream
  format.html { redirect_to @room }
end
```

app/views/messages/create.turbo_stream.erb
```erb
<%= turbo_stream.append "messages", @message %>
```

### stimulus controller
autoloader for your controllers  
done with importmaps supported by es shime

unprocessed es6 code is loaded by the browser  
directly via esm

app/javascript/controllers/reset_form_controller.js

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  reset() {
    this.element.reset()
  }
}
```

register in index  
app/javascript/controllers/index.js

```javascript
import ResetFormController from "./reset_form_controller"
application.register("reset_form", ResetFormController)
```

### web socket connection
start with
```erb
<%= turbo_stream_from @room %>
```

check network tab  
type websocket, WS  
`/cable`  
{"type": "welcome"}  
{"command": "subscribe", "identifier": "{\"channel\}..."}]  
{"command": "identifier", "channel": "\" }

add to model  
app/models/message.rb
```ruby
after_create_commit -> { broadcast_append_to room }
```

### connect console to shared adapter
uncomment in Gemfile
```gemfile
gem "redis", ">= 4.0.1"
```
```bash
bundle
```

config/cable.yml  
*(old value is adapter: async)*

```yaml
development:
  adapter: redis
  url: redis://localhost:6379/1
```

start redis
```bash
redis-server
```

restart rails

### model broadcast
```ruby
broadcasts_to :room
```

is same as:

```ruby
after_create_commit -> { broadcast_append_to room }
after_destroy_commit -> { broadcast_remove_to room }
after_update_commit -> { broadcast_replace_to room }
```

Async broadcasts are used.  
And rendering is done by job queue.  
(rendering of partials)

### add broadcast to rooms
because room is used to identify stream:

```ruby
class Room < ApplicationRecord
  has_many :messages
  broadcasts
end
```

app/views/rooms/show.html.erb
```erb
<%= render @room %>
```

app/views/rooms/\_room.html.erb

```erb
<div id="<%= dom_id room %>">
  <p>
    <strong>Name:</strong>
    <%= room.name %>
  </p>
</div>
```



Using Rails for API-only Applications
=====================================
https://guides.rubyonrails.org/api_app.html

```bash
rails new my_api --api
```
this will:  
- set ActionController::API as base of controller
  (not ActionController::Base)  
- configure generators to skip generating view, helpers and assets

```bash
bin/rails g scaffold Group name:string
bin/rails db:migrate
```

### Changing an existing application to API only
config/application.rb
```ruby
config.api_only = true
```

config/environments/development.rb
```ruby
config.debug_exception_response_format = :default
```
or default option when api_only = true
```ruby
config.debug_exception_response_format = :api
```

app/controllers/application_controller.rb  
from:

```ruby
class ApplicationController < ActionController::Base
end
```

to:

```ruby
class ApplicationController < ActionController::API
end
```

### list middleware used
```bash
bin/rails middleware
```

### Rack::Cache
add rack-cache to Gemfile
```gemfile
gem "rack-cache"
```

set config:
```ruby
config.action_dispatch.rack_cache = true
```

then use, here in show action:

```ruby
if stale?(last_modified: @group.updated_at)
  render json: @group
end
```

call to `stale?`  
  will potentially return "304 Not Modified" from action  
  compares value with request header If-Modified-Since

response header Last-Modified  
used when returning non cached value  
(perhaps also when returning cached value)

normally cache works per client basis

#### public mode:
Will cache full response in Rack::Cache per URL key:  
- cache response
- and Last-Modified

```ruby
if stale?(last_modified: @group.updated_at, public: true)
  render json: @group
end
```

I though that reason is:  
  public: false - only optimize transfer (will potentially not send content)  
  public: true-  optimize server, because potentially it will not query database  
... but Rails controller is run in public also  
*checked using debugger*

### Rack::Sendfile
using `send_file` in controller will set X-Sendfile header  
sending is actually done by `Rack::Sendfile`

#### accelerated file sending
Offloads serving files from application server to frontend server.  
Supported by:  
  Apache Http (mod_xsendfile module, X-Sendfile header),  
  Nginx (X-Accel_Redirect header),  
  Lighttpd (supports X-Sendfile natively)

Purpose is to faster close application server thread of handling request.  
And faster release resources allocated for handling request.

Whole thing is generally not noticeable on client side.

#### in rails
If server supports it, Rack::Sendfile will offload file sending to server.

configure name of header used:
```ruby
config.action_dispatch.x_sendfile_header
```

Apache and lighttpd
```ruby
config.action_dispatch.x_sendfile_header = "X-Sendfile"
```
Nginx
```ruby
config.action_dispatch.x_sendfile_header = "X-Accel-Redirect"
```

### JSON request params
ActionDispatch::Request will recognize that client is sending JSON  
and will make it available inside params

To use this, client needs to make request with  
`Content-Type` as `application/json`

### Session Middlewares
Normally API apps don't need sessions  
so by default, these middlewares are disabled:

```ruby
ActionDispatch::Session::CacheStore
ActionDispatch::Session::CookieStore
ActionDispatch::Session::MemCacheStore
```

#### config/application.rb
place to add any relevant options before middleware is built

```ruby
# This also configures session_options for use below
config.session_store :cookie_store, key: '_your_app_session'
# Required for all session management (regardless of session_store)
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

### Other disabled API Middlewares

```ruby
Rack::MethodOverride
ActionDispatch::Cookies
ActionDispatch::Flash
```

Any of them can be added via:
```ruby
config.middleware.use Rack::MethodOverride
```

Or removed:
```ruby
config.middleware.delete ::Rack::Sendfile
```

### Controller Modules
`ActionController::API` comes by default with:  
UrlFor url_for and similar helpers  
Redirecting redirect_to  
Rendering and ApiRendering  
Renderers:All support for render :json and friends  
ConditionalGet support for stale?  
BasicImplicitRender return empty response if no explicit  
StrongParameters support for parameters filtering  
DataStreaming send_file and send_data  
Callbacks support for before_action and similar  
Rescue support for rescue_from  
Instrumentation support for the instrumentation hooks (see instrumentation guide)  
ParamsWrapper wraps parameters hash into nested hash  
Head support for returning a response with no content

get other added modules from plugins:  
(in console)
```ruby
ActionController::API.ancestors - ActionController::Metal.ancestors
```

... return includes:

```ruby
ActionDispatch::Routing::PolymorphicRoutes
ActiveSupport::Benchmarkable
ActionController::RateLimiting
ActionView::ViewPaths
...
```

### Adding Other Modules
AbstractController::Translation: Support for the l and t

Support for basic, digest, or token HTTP authentication:  
ActionController::HttpAuthentication::Basic::ControllerMethods  
ActionController::HttpAuthentication::Digest::ControllerMethods  
ActionController::HttpAuthentication::Token::ControllerMethods

ActionView::Layouts: Support for layouts when rendering  
ActionController::MimeResponds: Support for respond_to  
ActionController::Cookies: Support for cookies

ActionController::Caching: Support view caching for the API controller.  
To use it, specify store in controller:

```ruby
class ApplicationController < ActionController::API
  include ::ActionController::Caching
  self.cache_store = :mem_cache_store
end
```



Getting Started with Rails
==========================
https://guides.rubyonrails.org/getting_started.html

```bash
rails new blog
```

### Controllers
```bash
bin/rails generate controller Comments
bin/rails generate controller Articles index --skip-routes
```

show action  
params

```ruby
def show
  @article = Article.find(params[:id])
end
```

#### create

```ruby
def create
  @article = Article.new(title: params[:title], body: params[:body])

  if @article.save
    redirect_to @article
  else
    render :new, status: :unprocessable_entity
  end
end
```

error code used  
422: Unprocessable Entity

`redirect_to` will cause new request in browser:  
it's used to avoid resend on refresh or going back  
`render` will keep current request but render different view

redirect after delete:
```ruby
redirect_to root_path, status: :see_other
```
303: see other

#### update

```ruby
def update
  @article = Article.find(params[:id])

  if @article.update(article_params)
    redirect_to @article
  else
    render :edit, status: :unprocessable_entity
  end
end
```

#### strong typing
way to have:  
- handy construction of value hash from form
- security that users will not add extra fields

```ruby
private
  def article_params
    params.require(:article).permit(:title, :body)
  end
```

### Routes
list routes:
```bash
bin/rails routes
```
url helpers: look for prefix  
`article_url` => "http://localhost:3000/articles/1"  
`article_path` => "/articles/1"

#### test in console
`
```bash
bin/rails console
app.articles_path ...
```
`  
link_to
```erb
<%= link_to article.title, article %>
```

link_to is sending GET request on hover?  
https://www.reddit.com/r/rails/comments/1b9mr1a/link_to_is_sending_get_request_on_hover/  
probably turbo 8  
(turned on by default?)

links to different method than GET:  
`/articles/4?_method=delete`  
used to be:
```erb
<%= link_to "Delete", @article, method: :delete %>
```
but method: :delete really doesn't work in default settings  
It needs to be turbo_method: :delete
```erb
<%= link_to "Delete", @article, data: { turbo_method: :delete } %>
```

config/routes.rb  
some basics:

```ruby
get "/articles", to: "articles#index"
get "/articles/:id", to: "articles#show"
```

add default routing
```ruby
root "articles#index"
```

#### resources
```ruby
resources :articles
```

will create:

```
articles#index
articles#show
articles#destroy

articles#new
articles#create

articles#edit
articles#update
```

#### nested resources

```ruby
resources :articles do
  resources :comments
end
```

### Migrations
Model names are singular, because an instantiated model represents a single data record.

```bash
bin/rails generate model Article title:string body:text
```
will create:
```ruby
class CreateArticles < ActiveRecord::Migration[7.2]
```

timestamps:
```ruby
t.timestamps
```
creates columns: created_at, updated_at

#### run migrations
```bash
bin/rails db:migrate
```

#### one to many
```bash
bin/rails generate model Comment commenter:string body:text article:references
```
creates:
```ruby
t.references :article, null: false, foreign_key: true
```

#### add column
generate migration that adds a column:
```bash
generate migration AddStatusToArticles status:string
```
will generate

```ruby
def change
  add_column :comments, :status, :string
end
```

### Models
`destroy`: will trigger callbacks  
  before_destroy  
  after_destroy  
`delete`: without callbacks

update all
```ruby
Article.update_all(status: "public")
```

#### one to many
```ruby
belongs_to :article, dependent: :destroy
```
add on other side to be able to navigate
```ruby
has_many :comments
```

optional destroy action
```ruby
dependent: :destroy
```
without it the many side will be left orphan

#### validations
rules that are checked before model is saved

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```

#### enum kind of validation

```ruby
VALID_STATUSES = ['public', 'private', 'archived']
validates :status, inclusion: { in: VALID_STATUSES }
```

### Concerns
mixins for controllers and models  
defined in:
```
app/controllers/concerns
app/models/concerns
```
example of model concern:

```ruby
module Visible extend ActiveSupport::Concern
  VALID_STATUSES = ['public', 'private', 'archived']

  included do
    validates :status, inclusion: { in: VALID_STATUSES }
  end

  class_methods do
    def public_count
      where(status: 'public').count
    end
  end

  def archived?
    status == 'archived'
  end
end
```

included  
needed because validates needs to run at class level

class_methods  
allows to define new class methods

#### using concerns
to use concer in a model:

```ruby
class Article < ApplicationRecord
  include Visible
end
```

### Views
ERB: Embedded Ruby

```erb
<h1>Articles</h1>
<ul>
  <% @articles.each do |article| %>
    <li><%= article.title %></li>
  <% end %>
</ul>
```

ERB evaluate
```erb
<%
```
ERB evaluate and output return
```erb
<%=
```

#### Partials
used to share parts of view
```erb
<%= render "form", article: @article %>
```

A partial's filename must be prefixed with an underscore  
But when rendering, it is referenced without the underscore

render partial from another folder
```erb
<%= render "comments/comment", comment: comment %>
```

render array of partials, one after another
```erb
<%= render @article.comments %>
```

### Form builder

```erb
<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

in case of create
```erb
<%= form_with model: @article do |form| %>
```
will generate:

```html
<form action="/articles" accept-charset="UTF-8" method="post">
  <input type="hidden" name="authenticity_token" value="...">
```

and
```erb
<%= form.label :body %><br>
```
will turn into (notice field name):
```html
<textarea name="article[body]" id="article_body"></textarea>
```

form for model that belongs to another  
(comments belong to article)
```erb
<%= form_with scope: :comment, url: article_comments_path(@article) do |form| %>
```
or shorter and also handle edit:
```erb
<%= form_with model: [@article, @article.comments.build ] do |form| %>
```
which is same as
```erb
<%= form_with model: [@article, @article.comments.new ] do |form| %>
```

#### select field

```erb
<div>
  <%= form.label :status %><br>
  <%= form.select :status, Visible::VALID_STATUSES, selected: article.status || 'public' %>
</div>
```

### Other
#### Autoloading
Rails applications do not use require to load application code.  
don't implictly require anything in `/app`

you only need to require:  
files in `/lib`  
loading gem dependencies that have `require: false`

#### console

```bash
bin/rails console
```
```ruby
article = Article.new(title: "Hello Rails", body: "I am on Rails!")
article.save
Article.find(1)
Article.all
```

#### debugger
in code:
```ruby
debugger
```
then c: continue, n: next (up/down?)  
Type `local_variables` to list local variables.  
Type `instance_variables` to list instance variables.

#### strange syntax
the syntax ClassName[argument] is actually a way to call a class method called [] on ClassName

```ruby
def self.[](version)
  Compatibility.find(version)
end
```

#### basic authentication
```ruby
http_basic_authenticate_with name: "admin", password: "admin", except: [:index, :show]
http_basic_authenticate_with name: "admin", password: "admin", only: :destroy
```

popular authentication add-on:

Devise  
https://github.com/heartcombo/devise  
- full featured
- out of box
- provides set structure and conventions
- built-in views
- some modularity (choose features)
Authlogic  
https://github.com/binarylogic/authlogic  
- more a toolkit
- more flexible
- no built-in views
- less popular
