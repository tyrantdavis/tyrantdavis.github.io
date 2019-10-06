---
layout: project
title: Mocpedia
thumbnail-path: ""
short-description: "&bull; Mocpedia is a production quality SaaS web application which allows users to create public and private Markdown based wikis and share them with other collaborators. &bull; Built on Rails &bull; Incorporates  Stripe, Devise, Redcarpet, Pundit, Sendgrid, Figaro, and Faker"

---


<p align="center">
  <img  src="/assets/images/doUpedia/mocpediaHome.png" alt="home">
</p>

<h3 class="wide w3-center">Summary</h3>
**Mocpedia** is a production quality _SaaS_ web application which allows users to create public and private Markdown based wikis and share them with other collaborators.

<h3 class="wide w3-center">Explanation</h3>

<span class="w3-text-magenta">CRUD</span> application built using _Ruby on Rails_. This build uses some helpful gems including _Stripe, _Devise_, Pundit, Redcarpet,_ and _Faker_.

<h3 class="wide w3-center">Problem</h3>

The aim for this project was to further my backend knowledge and skills of Ruby on Rails and gain greater exposure to some fantastic Ruby gems. Another objective was for me to meet all of the predefined user requirements.

<h3 class="wide  w3-center">Objectives</h3>

* Create basic Rails user scheme
* Giver user the ability to sign up for app using _Devise_
* Create CRUD routes and resources for Rails app
* Authenticate and authorize users
* Integrate Stripe for payments
* Integrate Redcarpet Markdown gem for rendering

<h3 class="wide  w3-center">User Requirements</h3>

1. Allow users to sign up for a free account by providing a user name, password and email
2. Allow users to sign in and out of Mocpedia
3. Allow users with a standard account to create, read, update, and delete public wikis
4. Provide three user roles: admin, standard, or premium
5. Allow user to upgrade account from a free to a paid plan
6. Allow premium user to create private wikis
7. Allow user to edit wikis using Markdown syntax
8. Allow premium user to add and remove collaborators for my private wikis

<h3 class="wide  w3-center">Solution</h3>

The initial stages of the app involved generating a new Rails app, configuring git and Github for collaboration, and adding default gems for the project.

<center class="highlight">
{% highlight Ruby %}

$ rails _4.2.5_ new new-rails-project —skip-test-unit
$ cd new-rails-project
{% endhighlight %}
</center>

I generated a Welcome controller and related views, using HTML and CSS from there. This gave users a landing page to welcome them to the app. I created an index view which indexed all wikis for all to see, including those just visiting the site. A creative and playful **call to action** was included to grab the users attention. It also guides the user to the unusually placed pagination just below.


<p align="center">
  <img  src="/assets/images/doUpedia/index.png" alt="home">
</p>

To sign users up for the app I added user authentication by incorporating _Devise_. I then created a _Devise_ User model.

<center class="highlight">
{% highlight Ruby %}
$ rails g _Devise_ user
{% endhighlight %}
</center>


<p align="center">
  <img  src="/assets/images/doUpedia/signup.png" alt="home">
</p>

_Sendgrid_ was integrated into the app which allowed the app to send confirmation emails. To securely configure Sendgrid username and password the _Figaro_ gem was used. I find myself using this gem quite a bit.  


<p align="center">
  <img src="/assets/images/doUpedia/figaro.png" alt="Figaro Logo" style="width:20%" >
</p>
For sign-in sign-out capabilities I used the _Devise_ helper method **`user_signed_in?`** which determined if a user was signed in and rendered a particular view based on the response. The navigation links **Edit Profile** and **Sign Out** indicated a user was signed in. The user would see the **Sign-up** or **Sign In** navigation links if not signed in.   


<p align="center">
  <img  src="/assets/images/doUpedia/login.png" alt="home">
</p>

I generated a wiki model that references the user, plus a controller, and views for the resource. It enabled users to <span class="w3-text-magenta">Create</span>, <span class="w3-text-magenta">Read</span>, <span class="w3-text-magenta">Update</span>, and <span class="w3-text-magenta">Delete</span> public wikis.

_Faker_ was used to seed the database.


<p align="center">
  <img  src="/assets/images/doUpedia/entry.png" alt="home">
</p>


<center class="highlight">
{% highlight Ruby %}
$ rails g model Wiki title:string body:text private:boolean user:references:index
{% endhighlight %}
</center>

_Pundit_  was used for authorization. I defined three user roles: **standard, premium, admin** using an **_enum_** attribute on the User model. The `before_save` callback implemented standard as the default value for users.

<center class="highlight">
{% highlight Ruby %}
app/models/user.rb
...
before_save { self.role ||= :standard }
...
enum role: [:standard, :premium, :admin]
...
{% endhighlight %}
</center>


For this project I wanted to allow users to edit any public wiki unlike Bloccit where the user had to be an admin. To do so the **#update** method for application policy was modified to:

<center class="highlight">
{% highlight Ruby %}
app/policies/application_policy.rb

  def update?
    user.present? &&( (record.user == user) || user.admin? )
  end
    {% endhighlight %}
</center>

An inner Scope class was added to the _wiki_policy_ to regulate which wikis populated on the index page. The scope was then used to modify the **#index** action in the wikis_controller to show the right wikis and limit the # of wikis shown per page.

<center class="highlight">
{% highlight Ruby %}

def index
  @wikis = policy_scope(Wiki).paginate(page: params[:page], per_page: 6)
end  
   {% endhighlight %}
</center>


_Stripe_ was later integrated for payment processing. This feature was necessary to allow users a path to upgrading to a premium account. A ChargesController was generated with a **#create** action and a **#new**. This controller initializes and then creates Charge objects after receiving **`params: stripeToken`** and **`amount`** , ultimately sending them through Stripes API to complete the transaction. I also designed a user flow for down grading or reverting back to a standard account.

I implemented some privacy controls on wikis to manage private wikis. Controls checked to see if the users role was admin or  premium  before allowing a private wiki to be edited. The control included in wikis partial  **` app/views/wikis/_form.html.erb `** presented a checkbox only viewable by premium users or an admin.

_Redacarpet_ was integrated in order to parse Markdown syntax .  

The last resource generated was the Collaborator. A collaborator model was built to allow users to add and remove collaborators from private wikis from the wiki’s edit page. A relationship between wikis and users was established through the collaborators model by implementing a has many through relationship.  

<center class="highlight">
{% highlight Ruby %}
collaborator.rb

class Collaborator < ActiveRecord::Base
  belongs_to :user
  belongs_to :wiki
end
{% endhighlight %}
</center>


<h3 class="wide  w3-center">Results</h3>

All user requirements met and working as expected.

<h3 class="wide w3-center">Conclusion</h3>

I could have implemented authentication and authorization mechanisms from scratch which is unnecessary, but a very valuable learning tool. I could have generated the views for this app using HAML instead of ERB. I could have added a real-time Markdown editor for wikis.I am always looking to grow so these are definitely features to add to in the future or to future projects. This was a really fun project. Rails is incredible and I love it.
