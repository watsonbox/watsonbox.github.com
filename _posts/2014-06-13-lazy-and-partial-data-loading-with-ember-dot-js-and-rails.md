---
layout: post
title: "Lazy and Partial Data Loading with Ember.js and Rails"
date: 2014-06-13 14:43:13 +0200
permalink: /blog/2014/06/13/lazy-and-partial-data-loading-with-ember-dot-js-and-rails/
tags: [Ember.js, Ruby, Rails]
---

While learning [Ember.js](http://emberjs.com/), I couldn't find all the info laid out clearly in one place on these subjects, so thought I'd write up my findings. I'm using Ember.js 1.5.1, Ember Data 1.0.0-beta.7+canary.f482da04 (!!), and Rails 4.1.0.

## Lazy Loading Relationships

Most of the [Rails/Ember guidelines](http://emberjs.com/api/data/classes/DS.ActiveModelSerializer.html) out there suggest that related data should normally be sideloaded, which is great and helps reduce the number of HTTP requests required, or data duplication (in the case of embedded data). To sideload data, set up the relationship and Rails serializer as follows:

`app/assets/javascripts/models/project.js.coffee`

```coffeescript
App.Project = DS.Model.extend
  name: DS.attr('string')
  description: DS.attr('string')
  invoices: DS.hasMany('invoice')
```

`app/serializers/project_serializer.rb`

```ruby 
class ProjectSerializer < ActiveModel::Serializer
  attributes :id, :name
  has_many :invoices, embed: :ids, include: true # Sideload relationship
end
```

Example JSON response:

```json
{
  "projects": [
    { "id": 1, "name": "Project 1", "description": "...", "invoice_ids": [1, 2] }
  ],
  "invoices": [
    { "id": 2, "reference": "INV-002", "date": "2014-06-11", "project_id": 1 },
    { "id": 1, "reference": "INV-001", "date": "2014-04-04", "project_id": 1 }
  ]
}
```

### Async Loading

Often we'd prefer to lazily load the associated data only when it's referenced. Ember Data calls this an async relationship. Simply modify the above by omitting the `include` serializer option and adding `async` to the relationship:

`app/assets/javascripts/models/project.js.coffee` (partial)

```coffeescript
invoices: DS.hasMany('invoice', { async: true })
```

`app/serializers/lazy_project_serializer.rb` (partial)

```ruby
has_many :invoices, embed: :ids
```

Example JSON response:

```json
{
  "projects": [
    { "id": 1, "name": "Project 1", "description": "...", "invoice_ids": [1, 2] }
  ]
}
```

When the `invoices` relationship is accessed, Ember Data will automatically make a request to `/invoices?ids[]=1&ids[]=2` (or presumably wherever that route is defined), so the Rails `InvoicesController` must be set up to restrict returned data based on the `ids` parameter.

*Update 30/12/14: Since Ember Data v1.0.0-beta.9 [has many coalescing has become opt-in](http://emberjs.com/blog/2014/08/18/ember-data-1-0-beta-9-released.html). This means setting `coalesceFindRequests: true` on the REST adapter for the above behavior. Thanks to CamonZ for pointing this out.*

Note that if an association is set to `async`, but sideloaded data exists in the server response, Ember Data will simply use that data and not attempt to make another request. This is useful, allowing data to be sideloaded for a detail view because we know we're going to need it, but not for a list view where it might not be used. When moving from list to detail, the invoices will be loaded, but when arriving directly on the detail page, only the project will be loaded. Specify a different serializer in Rails for each action:

`app/controllers/projects_controller.rb`

```ruby
class ProjectsController < ApplicationController
  respond_to :json

  def index
    respond_with Project.all, each_serializer: LazyProjectSerializer
  end

  def show
    respond_with Project.find(params[:id])
  end
end
```

### Async Loading From Links

Another possibility, which gives more control over association endpoints, and avoids having to pass a bunch of IDs around, is to provide links for the relationships in the JSON response. For example:

`app/serializers/project_serializer.rb`

```ruby
class ProjectSerializer < ActiveModel::Serializer
  attributes :id, :name, :links

  def links
    { invoices: project_invoices_path(id) }
  end
end
```


## Loading Partial Models

Imagine that we have a dropdown list of projects in the page navbar. In addition to lazy loading related data, we might also want to omit attributes we know we're not going to need yet. In this case a large project description might be a candidate for ommission from the list view, especially if the list of projects is large. The most comprehensive description of this problem I was able to find is [here](http://discuss.emberjs.com/t/loading-partial-models-then-filling-them-with-ember-data/819), and includes links to related discussions.

Firstly we need to identify the list data as partial by adding a `partial` attribute and setting it in a `PartialProjectSerializer`. We also ommit the invoices and description:

`app/assets/javascripts/models/project.js.coffee`

```coffeescript
App.Project = DS.Model.extend
  name: DS.attr('string')
  description: DS.attr('string')
  partial: DS.attr('boolean')
  invoices: DS.hasMany('invoice', { async: true })
```

`app/serializers/partial_project_serializer.rb`

```ruby
class PartialProjectSerializer < ApplicationSerializer
  attributes :id, :name, :partial

  def partial
    true
  end
end
```

We now have two problems to solve:

1. Reload a complete model for the detail view if we have only a partial model.
2. Don't allow partial list data to overwrite a complete model if it comes in afterwards. You can simulate this in your dev environment using a threaded web server such as Puma and setting a delay on the resource index.

For the first we can use `setupController` on the project route. Modifying the `model` hook won't work when a model is passed for example to `link-to` because it [doesn't get called](http://emberjs.com/guides/routing/asynchronous-routing/#toc_beforemodel-and-aftermodel).

`app/assets/javascripts/routes/project_route.js.coffee`

```coffeescript
Facture.ProjectRoute = Ember.Route.extend
  model: (params) ->
    @store.find 'project', params.project_id

  setupController: (controller, model) ->
    # If the model is partial, we'll refresh it (from the full project resource)
    model.reload() if model.get 'partial'
    controller.set 'model', model
```

It would be nice if we could solve the second problem by asking Ember Data to retrieve and *merge* data from the server (using something like `Store#find_and_update`), but that doesn't appear to be possible. It is however possible to update individual records using [`Store#update`](http://emberjs.com/api/data/classes/DS.Store.html#method_update).

I've been getting these projects in the `ApplicationRoute`, which now looks like this:

`app/assets/javascripts/routes/application.js.coffee`

```coffeescript
Facture.ApplicationRoute = Ember.Route.extend
  setupController: (controller) ->
    $.getJSON '/projects.json', (data) =>
      # Update records in the store
      projects = data['projects'].map (project) =>
        # Don't merge partial=true
        if existing_project = @store.getById('project', project.id)
          project.partial = false unless existing_project.get('partial')

        @store.update 'project', project

      # Set all records on the controller
      controller.set 'projects', projects
```
