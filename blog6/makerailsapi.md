

# Simple RESTful API in RAILS

In this blog post I will discuss, how you can create a __RESTful API__ in rails.

There are many resources out there to teach you how API is created in rails. But the problem with all of them is, they include many things that are not required to be included. I had also faced these problems while learning API in rails. Therefore, I want to write this article, so that, you can remain focused about learning how to create API in rails avoiding unnecessary topics. 

In this article, I will create a simple api where Users can be added, deleted or updated in the database.

# Let's start

Open your terminal and create a new app in rails using the following command: 

```bash
$ rails new userapi --api
```

the flag `--api` tells that we only want api specific functionalities and not other unnecessary stuff like, views and others. 

Now, we have to define the endpoints for our api. These endpoints will be used for making requests by the client.

We will define the endpoints for the following:

* creating a new user
* updating user information
* delete a user
* show all users
* get information about a user.

In your app's folder, go to `config/` > `routes.rb` and add the line `resources :users`.
Now your `routes.rb` file should look like, 

```ruby
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  resources :users
end
```

To see what routes are available for your api, go to the root of your app and run the following command on terminal:

```bash
$ rails routes
```

<!-- image link will be given here -->

Now, We need to provide a controller that will decide what action to take for each route(or path)

Run the following command on terminal
```bash
$ rails g controller Users
```

Finally, attach a model (database) to your app that can store user database.

```bash
$ rails g model User username:string contact:string
# output
$ rails db:migrate
# output
```

Now, that we are done setting up everything, Lets quickly define what each of the routes will do.

Open `userapi` > `app` > `controllers` > `users_controller.rb` and enter the following code.

```ruby
class UsersController < ApplicationController
	def index
		@users = User.select("id, username, contact").all
		render json: {message: "current list of users", data: @users }, status: :ok
	end

	def create
		@user = User.new user_params 
		if @user.save
			render json: {message: "user created", data: @user}, status: :ok
		else
			render json: {message: "Error creating user"}, status: :unprocessable_entity
		end
	end

	def show
		@user = User.find params[:id]
		render json: {message: "User details successfully returned", data: @user}, status: :ok
	end

	def update
		@user = User.find params[:id]
		if @user.update user_params
			render json: {message: "User details Updated!", data: @user}, status: :ok
		else
			render json: {message: "Error updating details"}, status: :unprocessable_entity
		end
	end

	def destroy
		@user = User.find params[:id]
		@user.destroy
		render json: {message: "Successfully deleted!"}, status: :ok
	end

	private

	def user_params
		params.permit(:username, :contact)
	end
end
```

The main catch is that instead of sending to a view as in MVC we are rendering a json and that json will be sent to the client.

# A few points 

In the above code the things to take care of are:

`status` must be set according to the result returned from the database processing 
e.g. if in create method @user.save returns nil then we must send a response to user with a json stating the error which in this case is set to `:unprocessable_entity`

To use this api between pcs you must take care of [CORS](https://github.com/cyu/rack-cors) i.e. cross-origin resource sharing

# Lets Test it

run the following command from the root of your of app
```bash
$ rails s
```

Now you can use terminals inbuilt utility `curl` to send `GET`, `POST`, `DELETE` etc requests to server

e.g following commands sends a simple `GET` request to our api

```
$ curl -i -H "Accept: application/json" -H "Content-type: application/json" -X GET "localhost:3000/users"
```

This returns the following result:
 
\* To send some `JSON` data along with the request as for `POST` request, use -d switch and pass the data along with it.

```
$ curl -i -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{"username": "Xhaxs", "contact": "1234567890"}' "localhost:3000/users"
```

<!-- image 1 -->
<!-- image 2 -->
<!-- image 3 -->


