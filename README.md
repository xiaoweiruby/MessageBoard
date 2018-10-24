# Ruby on Rails Tutorial | Building a Message Board

我们在处理一般的流程的逻辑的时候，其实应该是有一定的项目的打造的节奏，这样才可以比较快速的完成案例体系的梳理。

#一、构建 Message 和 Comment ；

首先构建个博客体系的功能结构，可以完成写文章和写评论；

#二、安装 Bootstap-sass 和 Simple_form 的装饰；

其次是为了让页面变得更加漂亮一些可以使用表单和套装完成调整；

#三、安装 devise 和 active_admin 的用户的对接；

然后是构建一个用户的体系，完成数据和用户体系的对接；

#四、安装 paperclip，will_paginate，acts_as_votable；

完成用户的图片的上传功能体系；

#五、上传 Heroku 和 Aliyun，完成域名的对接；

以上的功能结构基本上属于一个逻辑的体系梳理，会使用上面的案例体系的个体，基本上就属于一个新手应该具备的知识体系；

```

10224  cd newspace
10225  ks
10226  ls
10227  cd MessageBoard
10228  git init
10229  git add .
10230  git commit -m "initial comment"
10231  git remote add origin https://github.com/xiaoweiruby/MessageBoard.git
10232  git push -u origin master
```
git checkout -b messages
```
10233  atom .
10234  rails g model Message title:string description:text
10235  rake db:migrate
10236  rails g controller Messages
10237  rails s
```
app/controllers/messages_controller.rb
```
class MessagesController < ApplicationController
	before_action :find_message, only: [:show, :edit, :update, :destroy]

	def index
		@messages = Message.all.order("created_at DESC")
	end

	def show
	end

	def new
		@message = Message.new
	end

	def create
		@message = Message.new(message_params)
		if @message.save
			redirect_to root_path
		else
			render 'new'
		end
	end

	def edit
	end

	def update
		if @message.update(message_params)
			redirect_to message_path
		else
			render 'edit'
		end
	end

	def destroy
		@message.destroy
		redirect_to root_path
	end

	private
		def message_params
			params.require(:message).permit(:title, :description)
		end

		def find_message
			@message = Message.find(params[:id])
		end
end

```
touch app/views/messages/-form.html.erb
```
<%= simple_form_for @message do |f| %>
	<%= f.input :title, label: "Message Title" %>
	<%= f.input :description %>
	<%= f.button :submit %>
<% end %>

```
touch app/views/messages/index.html.erb
```
<% @messages.each do |message| %>
	<div class="col-md-4">
		<div class="message">
			<h2 class="message-title"><%= message.title %></h2>
			<%= link_to "View Message", message, class: "btn-custom" %>
		</div>
	</div>
<% end %>

```
touch app/views/messages/new.html.erb
```
<div class="col-md-8 col-md-offset-2">
	<h1 class="new-msg-header">New Message</h1>
	<%= render 'form' %>
</div>

```
touch app/views/messages/edit.html.erb
```
<h1>Edit Message</h1>

<%= render 'form' %>

```
touch app/views/messages/show.html.erb
```
<div class="col-md-8 col-md-offset-2">
	<div class="message-show">
		<h2><%= @message.title %></h2>
		<p class="message-posted-by">Posted by <%#= @message.user.email %> <%= time_ago_in_words(@message.created_at) %> ago</p>
		<p class="message-desc"><%= @message.description %></p>
		<!-- Add a Posted By: username at timestamp. -->

		<h3 class="comment-section-header">Comments:</h3>
		<!-- renders the _comment.html.erb partial file. -->
		<p><%#= render @message.comments %></p>

		<h3 class="reply-to-msg">Reply to Message:</h3>
		<!-- renders the comments form -->
		<%#= render 'comments/form' %>


		<!-- Add a conditional so that the 'edit' and 'destroy' links only appear for the user who created the message. -->
		<div class="links btn-group">
			<%= link_to "Back", root_path, class: "btn btn-default" %>
			<%# if @message.user_id == current_user.id %>
				<%= link_to "Edit", edit_message_path(@message), class: "btn btn-default" %>
				<%= link_to "Delete", message_path(@message), method: :delete, data: { confirm: "Are you sure?" }, class: "btn btn-default" %>
			<%# end %>
		</div>
	</div>
</div>

```

Gemfile
gem 'simple_form', '~> 4.0', '>= 4.0.1'
gem 'bootstrap-sass', '~> 3.3', '>= 3.3.7'
```
10238  bundle install
10239  rails generate simple_form:install --bootstrap
```



Gemfile
gem 'devise', '~> 4.5'
```
10247  rails generate devise:install
10248  rails g devise:views
10249  rails g devise User
10250  rake db:migrate
10251  rails s
rails g migration add_user_id_to_messages user_id:integer
rake db:migrate
```
app/views/layouts/application.html.erb
```
<!DOCTYPE html>
<html>
<head>
  <title>MessageBoard</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

	<nav class="navbar navbar-default">
		<div class="container">
			<div class="navbar-header">
				<%= link_to "Message Board", root_path, class: "navbar-brand" %>
			</div>
			<ul class="navbar-nav nav">
				<li><%= link_to "Sign Up", new_user_registration_path %></li>
				<% if user_signed_in? %>
					<li><%= link_to "Sign Out", destroy_user_session_path, method: :delete %></li>
				<% else %>
					<li><%= link_to "Log In", new_user_session_path %></li>
				<% end %>
			</ul>
			<% if user_signed_in? %>
				<p><%= link_to "New Message", new_message_path, class: "navbar-text navbar-link navbar-right" %></p>
			<% end %>
		</div>
	</nav>

<div class="container">
	<p class="notice"><%= notice %></p>
	<p class="alert"><%= alert %></p>
  </div>

	<div class="container">
		<%= yield %>
	</div>

</body>
</html>

```
app/models/user.rb
```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
         has_many :messages
end

```
app/models/message.rb
```
class Message < ApplicationRecord
  belongs_to :user
end
```

app/controllers/messages_controller.rb
```
class MessagesController < ApplicationController
	before_action :find_message, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]
	def index
		@messages = Message.all.order("created_at DESC")
	end

	def show
	end

	def new
		@message = current_user.items.build
	end

	def create
		@message = current_user.items.build(message_params)
		if @message.save
			redirect_to root_path
		else
			render 'new'
		end
	end

	def edit
	end

	def update
		if @message.update(message_params)
			redirect_to message_path
		else
			render 'edit'
		end
	end

	def destroy
		@message.destroy
		redirect_to root_path
	end

	private
		def message_params
			params.require(:message).permit(:title, :description)
		end

		def find_message
			@message = Message.find(params[:id])
		end
end

```
```
rails c
Message.connection
@message = Message.last
@message = Message.find(1)
@message.user_id = 1
@message.save
exit
```

git checkout -b comments
rails g model Comment content:text message:references user:references
touch app/controllers/comments_controller.rb
```
<div class="comment">
	<p class="comment-content"><%= comment.content %></p>
	<p class="comment-posted-by">Posted by <%= comment.user.email %> <%= time_ago_in_words(comment.created_at) %> ago</p>

	<div class="links">
		<%# if comment.user_id == current_user.id %>
			<%= link_to "Edit", edit_message_comment_path(comment.message, comment) %>
			<%= link_to "Delete", message_comment_path(comment.message, comment), method: :delete, data: { confirm: "Are you sure?" } %>
		<%# end %>
	</div>
</div>

```
touch app/views/comments/-form.html.erb
```
<%= simple_form_for([@message, @message.comments.build]) do |f| %>
	<%= f.input :content, label: "Comment" %>
	<%= f.button :submit %>
<% end %>

```
touch app/views/comments/edit.html.erb
```
<h1>Edit Comment</h1>
<%= simple_form_for([@message, @comment]) do |f| %>
	<%= f.input :content, label: "Comment" %>
	<%= f.button :submit %>
<% end %>

```
```
class CommentsController < ApplicationController
	before_action :find_message, only: [:create, :edit, :update, :destroy]
	before_action :find_comment, only: [:edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]

	def create
		# creates a comment with respect to the message
		@comment = @message.comments.create(comment_params)
		@comment.user_id = current_user.id

		if @comment.save
			redirect_to message_path(@message)
		else
			render 'new'
		end
	end

	def edit
	end

	def update
		if @comment.update(comment_params)
			redirect_to message_path(@message)
		else
			render 'edit'
		end
	end

	def destroy
		@comment.destroy
		redirect_to message_path(@message)
	end

	private

		def comment_params
			params.require(:comment).permit(:content)
		end

		def find_message
			@message = Message.find(params[:message_id])
		end

		def find_comment
			@comment = @message.comments.find(params[:id])
		end

end

```


app/assets/stylesheets/application.scss
```
@import "bootstrap-mincer";
@import "bootstrap";

/* General styles */

body {
 background-color: #fff;
 font-family: 'Playfair Display', serif;
}

h1, h2, h3, h4, h5, h6 {
 font-family: 'Muli', sans-serif;
 font-weight: 300;
}

.btn-custom {
 background-color: #fff;
 color: #0a8fd5;
 font-family: 'Muli', sans-serif;
 font-weight: 300;
 border-radius: 8px;
 padding: 10px 20px;
 &:hover {
   color: #383838;
   transition: 0.5s;
   background-color: #fff;
   text-decoration: none;
 }
}

.btn:hover {
 transition: 0.5s;
 color: #383838;
 background-color: #fff;
}


/* Navbar styles */

.navbar {
 font-family: 'Muli', sans-serif;
 text-transform: uppercase;
}

.navbar-default {

 padding-top: 5px;
 padding-bottom: 5px;
 background-color: #fff;

 .navbar-nav > li > a {
   color: #222;
 }

 .navbar-link {
   color: #222;
 }

 .navbar-brand {
   color: #222;
   letter-spacing: 1.1px;
 }

}


/* Links styles */

.links {
 margin-top: 20px;
 .btn-default {
   color: #0a8fd5;
 }
}


/* Message styles */

.message {
 box-sizing: border-box;
 background-color: #383838;
 margin-bottom: 25px;
 margin-right: 10px;
 margin-left: 10px;
 padding: 1.4em 1.4em;
 height: 150px;
 border-bottom: 4px solid #0a8fd5;

 .message-title {
   font-size: 1.5em;
   font-family: 'Muli', sans-serif;
   font-weight: 300;
   margin-top: 0;
   color: white;
   margin-bottom: 25px;
 }

 .btn-custom {
   margin-top: 20px;
 }

}

.message:nth-child(4n) {
 clear: left;
}

/* Message show-page styles */

.message-show {
 box-sizing: border-box;
 background-color: #383838;
 padding: 2.0em 1.5em;
 border-bottom: 4px solid #0a8fd5;
 color: #fff;
 font-family: 'Muli', sans-serif;
 font-weight: 300;

 h2 {
   margin-top: 0;
 }

 .message-desc {
   font-size: 1.1em;
   line-height: 1.75;
   margin-bottom: 50px;
 }

 .message-posted-by {
   color: #eee;
   font-size: 0.9em;
 }

 .reply-to-msg {
   margin-top: 50px;
 }

}


/* Comment styles */

.comment {
 border-bottom: 1px solid #fff;
 padding-top: 15px;
 padding-bottom: 15px;

 .comment-content {
   font-size: 1.25em;
 }

 .comment-posted-by {
   color: #eee;
   font-size: 0.8em;
 }

 .links {
   margin-top: 15px;
   a {
     color: #0a8fd5;
     margin-right: 15px;
   }
 }
}
```
