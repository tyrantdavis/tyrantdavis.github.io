---
layout: project
title: taSk!
thumbnail-path: ""
short-description: "&bull; Self-destructing to-do list Rails app &bull; Automatically deletes tasks older than 7 days. &bull; Incorporates Ajax to delete expired tasks &bull; Custom Rake task to automate deletions.
&bull;  Implements Devise, Pundit, Bootstrap, Faker, Figaro, PostgreSQL, Send Grid, Factory Girls.
"
---

{:.center}
<!-- ![]({{ site.baseurl }}/img/taSk/taSkHome.png) -->
<p align="center">
  <img  src="/assets/images/taSk/taSkHome.png" alt="user profile">
</p>

<h3 class="wide w3-center">Explanation</h3>

This application aims to mange to-do list by automatically removing  to-do items older than seven days. Items older than seven days are considered unimportant and should not be on your to-do list.  

<h3 class="wide w3-center">Problem</h3>

This project was meant to expand upon backend knowledge and skills, particularly Ruby on Rails. Primary object objective was for me to meet all of the predefined user requirements.  

<h3 class="wide w3-center">User Requirements</h3>
1. Allow user to sign up for a free account by providing a user name, password and email
2. Allow users to sign in and out of **taSk!**
3. Allow users to see my profile page
4. Allow users to create multiple to-do items
5. Allow users to mark to-do items as complete and have them deleted
6. Allow users to see how old a to-do item is
7. Users to-dos should be automatically deleted seven days after their creation date

<h3 class="wide w3-center">Solution</h3>
To start the project a new Rails app was generated. I then configured git and Github for collaboration. Next I configured the default gems for my project. To make styling the application easier I added _Bootstrap_. The aforementioned steps are the initial steps for every project of course.

Once the foundation for the app was laid I incorporated _Devise_ for authentication. This step involved generating a _Devise_ User model.

<center class="highlight">
rails g devise user
</center>


Rails generated a migration file as a result. To run the migrations I executed the following:
<center class="highlight">
Rake db:migrate
</center>

This allowed users to sign up for the application and send emails for confirmation.

<!-- ![Sign In](/img/taSk/signUp.png) -->
<p align="center">
  <img  src="/assets/images/taSk/signUp.png" alt="user profile">
</p>

For __sign-in sign-out capabilities__ I used the _Devise_ helper method **user_signed_in?** which determined if a user was signed in and rendered a particular view based on the response.

The navigation links _Edit Profile and Sign Out_ indicated a user was signed in.

<!-- ![Signed In](/img/taSk/taskList.png) -->
<p align="center">
  <img  src="/assets/images/taSk/taskList.png" alt="user profile">
</p>

The user saw _Sign-up or Sign In_ navigation links if not signed in.  

<!-- ![Sign Up](/img/taSk/signUp.png) -->
<p align="center">
  <img  src="/assets/images/taSk/signUp.png" alt="user profile">
</p>

To redirect users that are not signed in to the login page   I used the _Devise_ filter method.


<center style="color:grey">app/controllers/application_controller.rb</center>

<center class="highlight">
{% highlight ruby %}
before_action :authenticate_user!
{% endhighlight %}
</center>

To allow signed-in users to see their profile page I generated a UsersController which included a #show action.  To redirect the user to their show view after sign-in I updated the root path in routes.rb to map to the users show view.

<center style="color:grey">config/routes.rb</center>

<center class="highlight">
{% highlight ruby %}
...
  authenticated :user do
      root 'users#show', as: :authenticated_root
  end
...
{% endhighlight %}
</center>
<br>

My next task was to allow a user to create multiple to-do items. An Item model that references a user was generated to address this need. The items controller included only <span class="w3-text-magenta">C</span>reate and <span class="w3-text-magenta">D</span>estroy <span class="w3-text-magenta">CRUD</span> actions.


<center class="highlight">
$ rails g model Item name:string user:references
</center>

By referencing the User model a **belongs_to :user** relationship was created on the Item model.

<center style="color:grey">app/models/item.rb</center>

<center class="highlight">
{% highlight ruby %}
...

  belongs_to :user
...
{% endhighlight %}
</center>

 An **has_many :items** relationship was established on the **User** model.

<center style="color:grey">app/models/user.rb</center>

<center class="highlight">
{% highlight ruby %}
...
  has_many :items, dependent: :destroy
...
{% endhighlight %}
</center>

Next I generated an items controller with a _#create_ action and ensured that it was associated with a user. I created a form partial in items directory which renders the form allowing users to submit new items. I created a partial named \_item.html.erb in the times directory to show the body of each item associated with a user. Because the items partial renders a single item it was necessary to call it multiple times to render the partial for each item. I implemented an each loop to resolve that. <span class="w3-text-magenta">D</span>on’t<span class="w3-text-magenta">R</span>epeat<span class="w3-text-magenta">Y</span>ourself!

<!-- ![User Show As Root](/img/taSk/fullListNoLogo.png) -->
<p align="center">
  <img  src="/assets/images/taSk/fullListNoLogo.png" alt="user profile">
</p>

My next focus was allowing users to mark items as complete and have them removed immediately. To do so I used a **:delete**  link to complete a to-do item.

<center style="color:grey">app/views/items/&#95; item.html.erb</center>

<center class="highlight">
{% highlight erb %}
<%= link_to "", item, method: :delete, class: 'btn btn-danger', autofoucs: true %>
{% endhighlight %}
</center>

I used _Bootstrap_'s button class that create a custom **delete** button that links to the _#delete_ action. _Ajax_ was used to delete to-do items without reloading the page .

This app is all about time frames. SO to display the time remaining on a to-do item before it was automatically deleted I used a Rails helper method called **distance_of_time_in_words** which displays the number of days since an item was generated.

<center style="color:grey">app/views/items/&#95;item.html.erb</center>

<center class="highlight">
{% highlight erb %}
...

  <%= distance_of_time_in_words(Time.now, item.created_at + 7.days ) %> left <br>

...
{% endhighlight %}
</center>

In order to automatically **delete expired to-do items** I used the **_Rake_** utility. It is used to automate administrative tasks in **Rails** apps. Building a custom task was necessary to accomplish what was needed. I used the Rails task generator to generate the new task from the terminal.

<center class="highlight">
$ rails g task todo delete_items
</center>
The above command created the file lib/tasks/todo.rake. The file included the following:

<center style="color:grey">lib/tasks/todo.rake</center>

<center class="highlight">
{% highlight ruby %}
namespace :todo do
  desc "TODO"
  task delete_items: :environment do
  end

end
{% endhighlight %}
</center>

**namespace :todo** organized new tasks under the **todo** namespace.
Task **delete_items:** defined new deleted_items tasks.
Desc “TODO” supplied documentation for a task.



To ensure items older than seven days were deleted I modified the created task.

<center style="color:grey">lib/tasks/todo.rake</center>

<center class="highlight">
{% highlight ruby %}
   namespace :todo do
 -   desc "TODO"
 +  desc "Delete items older than seven days"
     task delete_items: :environment do
        Item.where("created_at <= ?", Time.now - 7.days).destroy_all
    end

 end
{% endhighlight %}
</center>

To run the Rake task and delete items older than seven days

<center class="highlight">
rake todo:delete_items
</center>

<h3 class="wide w3-center">Results</h3>
All user requirements met and working as expected.

<h3 class="wide w3-center">Conclusion</h3>

I had fun with this project.  I have made several apps using Rails to this point. I cannot get enough of Rails. Incorporating rake was very interesting. I could have made the users show page private. I will do this knowing I would want this feature as a user. I could have implemented user authentication from scratch instead of using a gem. I could have automated delete Rake tasks to run daily.

<h3 class="wide w3-center"> Post Completion Updates</h3>
<h4 class="wide">	 Added Features</h4>

1. <span class="strikethrough">Users show page private.</span>
2. <span class="strikethrough">Automated delete Rake task to run daily.</span>
