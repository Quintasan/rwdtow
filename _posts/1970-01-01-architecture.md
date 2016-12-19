---
layout: post
title: Architecture
order: 60
---

MVC is not an app architecture. This may not be obvious due to the reign of MVC-based frameworks, which teach us to use routing + model + view rendering as the way to build web apps. Unfortunately, business logic has no place in this list.

The "fat model, skinny controller" approach does not solve the issue either. It just sweeps the dust under the carpet, and you will still suffer from fat models used in numerous contexts. Changing code for one usage context will break another usage context.

As a general rule in OOP code, you should break code into smaller classes with smaller responsibilities. Ideally, this code should follow SOLID principles (**S**ingle Responsibility, **O**pen-Closed, **L**iskov Substitution, **I**nterface Segregation, and **D**ependency Inversion).

In the book "Practical Object-Oriented Design in Ruby: An Agile Primer," Sandi Metz proposes that just being `SOLID` is not enough – code should also be `TRUE`.


> If you define easy to change as
>
>  * Changes have no unexpected side effects
>  * Small changes in requirements require correspondingly small changes in code
>  * Existing code is easy to reuse
>  * The easiest way to make a change is to add code that in itself is easy to change.
>
> Then the code you write should have the following qualities. Code should be
>
>  * **T**ransparent - The consequences of change should be obvious in the code that is changing, and in the distant code that relies upon it.
>  * **R**easonable - The cost of any change should be proportional to the benefits the change achieves.
>  * **U**sable - Existing code should be usable in new and unexpected contexts.
>  * **E**xemplary - The code itself should encourage those who change it to perpetuate these qualities.

TODO: Describe approaches like Form Model, Service Object, Context

* [Gem: Active Interaction](https://github.com/orgsync/active_interaction)
* [SOLID Design Principles](https://www.practicingruby.com/articles/solid-design-principles)
* [Monolith to Microservices](https://medium.com/@ccverak/from-monolithics-to-microservices-e87841ce11fd)

### [Trailblazer](http://trailblazer.to)

Trailblazer is a powerful architectural framework. It can be used with any Ruby web framework, and has special adapters for Rails. It provides the missing pieces to organize business logic.

* Operations are composable entities that encapsulate an action with a context, validations, and permission checks. Almost everything you would normally write in the controller should be placed here.
* Active Record models are only used for simple finding, saving, and managing relations. They are limited to a single responsibility: data persistence operations.
* Forms are provided per operation, unbound from the single context of a Fat Model.
* Cells are small, encapsulated pieces of reusable view logic. They replace messy app helpers.
* Representers describe presentation rules for serializing and deserializing documents. These are used in a variety of places, from the internal parameter representation of Operations to the representation of data in a JSON API.


**Operation example**

```ruby
# CRUD action Operation
class Comment::Create < Trailblazer::Operation
  include Model
  model Comment, :create

  contract do
    property :body, validates: {presence: true}
  end

  def process(params)
    validate(params[:comment]) do
      contract.save
    end
  end
end

# Run Operation
op = Comment::Create.(comment: {body: "MVC is so 90s."})
# Get a Model from it
model = op.model
```

**Cell example**

```ruby
#Cell class
class Comment::Cell < Cell::ViewModel
    property :body
    property :author

    def show
      render
    end

  private
    def author_link
      link_to author.email, author_path(author)
    end
  end

# Template
- # app/concepts/comment/views/show.haml
  %li
    = body
    By #{author_link}

# Testing
  describe Comment::Cell do
    it do
      html = concept("comment/cell", Comment.find(1)).()
      expect(html).to have_css("h1")
    end
  end
```

**Representable example**

```ruby
# Class
class SongRepresenter < Representable::Decorator
  include Representable::JSON

  property :id
  property :title

  property :artist, decorator: ArtistRepresenter
end

# Serialize
SongRepresenter.new(song).to_json
#=> {"id": 1, title":"Fallout", artist:{"id":2, "name":"The Police"}}

# Restore object
song = Song.new # nothing set.

SongRepresenter.new(song).
    from_json('{"id":1,title":"Fallout",artist:{"id":2,"name":"The Police"}}')

song.artist.name #=> "The Police"
```


Trailblazer (#Trbr) concepts are somewhat difficult to understand and use properly at first, but they definitely make more and more sense as you become familiar with them. It is very hard to describe the whole Trailblazer philosophy in a short text. Nick Sutterer, the author of Trbr, has quite a lot of documentation with detailed descriptions, and has written a book that covers building a Rails app with Trbr, step-by-step.

It is definitely worth a try if you want to start making your Ruby apps better.

* [Trailblazer](http://trailblazer.to)
* [Trailblazer Book](https://leanpub.com/trailblazer)


### [ROM.rb](http://rom-rb.org/) (Ruby Object Mapper)

Ruby Object Mapper (ROM) is a Ruby persistence library with the goal of providing powerful object mapping capabilities without limiting the full power of your datastore.

* Isolate the application from persistence details
* Provide minimum infrastructure for mapping and persistence
* Provide shared abstractions for lower-level components
* Provide simple use of the underlying datastore when desired

All ROM components are stand-alone – they are loosely coupled, can be used independently, and follow the single responsibility principle. A single object that handles coercion, state, persistence, validation, and the all-important business logic, rapidly becomes complex. Instead, ROM provides the infrastructure that allows you to easily create small, dedicated classes for handling each concern individually, and then tie them together in a developer-friendly way.

```
TODO: ROM example
```

* [rom-rb](http://rom-rb.org/)

### [dry-rb](http://dry-rb.org/)

Dry-rb Is a collection of next-generation Ruby libraries, each intended to encapsulate a common task while remaining decoupled and reusable.

TODO: Add extended dry-rb gems description

* [dry-rb](http://dry-rb.org/)
* [list of all dry gems](http://dry-rb.org/gems/)

### [Rectify](https://github.com/andypike/rectify)

The Rectify gem provides some lightweight classes that make it easier to build Rails applications in a more maintainable way. It is built on top of several other gems and adds improved APIs to make things easier.

Currently, Rectify consists of the following concepts:

* Form Objects
* Commands
* Presenters
* Query Objects

You can use these separately or together, to improve the structure of your Rails applications.

The main problem that Rectify tries to solve is where your logic should go. Commonly, business logic is either placed in the controller or the model, and the views are filled with too much logic as well. The opinion of Rectify is that these places are incorrect and that your models, in particular, are doing too much.

Rectify's opinion is that controllers should just be concerned with HTTP related things, and models should just be concerned with data relationships. The problem then becomes how and where you implement validations, queries, and other business logic.

Using Rectify, Form Objects contain validations and represent the input data of your system. Commands then take a Form Object (as well as other data) and perform a single action, which is invoked by a controller. Query objects encapsulate a single database query, and any logic it needs. Presenters contain the presentation logic in a way that is easily testable, and keeps your views as clean as possible.

Rectify is designed to be very lightweight and allows you to use some or all of its components. We also advise that you use these components where they make sense, not just blindly everywhere. More on that later.

Here is an example controller that shows details about a user, and also allows a user to register an account. This creates a user, sends some emails, does some special auditing, and integrates with a third party system:

```ruby
class UserController < ApplicationController
  include Rectify::ControllerHelpers

  def show
    present UserDetailsPresenter.new(:user => current_user)
  end

  def new
    @form = RegistrationForm.new
  end

  def create
    @form = RegistrationForm.from_params(params)

    RegisterAccount.call(@form) do
      on(:ok)      { redirect_to dashboard_path }
      on(:invalid) { render :new }
      on(:already_registered) { redirect_to login_path }
    end
  end
end
```

The RegistrationForm Form Object encapsulates the relevant data that is required for the action, and the RegisterAccount Command encapsulates the business logic of registering a new account. The controller is clean, and business logic now has a natural home:

```
HTTP             => Controller   (redirecting, rendering, etc)
Data Input       => Form Object  (validation, acceptable input)
Business Logic   => Command      (logic for a specific use case)
Data Persistence => Model        (relationships between models)
Data Access      => Query Object (database queries)
View Logic       => Presenter    (formatting data)
```


### Learn OOP design

By learning to design small pieces – objects, in OOP – and put them together, you automatically learn how to make good app architecture overall. There is no a magical library that will suddenly make your code better. Good code comes from the combination of many tiny aspects.

* [Video: SOLID Object-Oriented Design by Sandi Metz](https://www.youtube.com/watch?v=v-2yFMzxqwU)
* [Video: All the Little Things by Sandi Metz](https://www.youtube.com/watch?v=8bZh5LMaSmE)
* [Video: Nothing is Something by Sandi Metz](https://www.youtube.com/watch?v=9lv2lBq6x4A)
* [Video: Therapeutic Refactoring by Katrina Owen](https://www.youtube.com/watch?v=J4dlF0kcThQ)
* [Video: Overkill by Katrina Owen](https://www.youtube.com/watch?v=GWEEPt8VvmU)
* [Book: Objects on Rails by Avdi Grimm](http://objectsonrails.com/)
* [Book: Practical Object-Oriented Design in Ruby (POODR) by Sandi Metz](http://www.poodr.com/)
