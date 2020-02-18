---
layout: post
title:      "Sinatra Associations and Joins Table"
date:       2020-02-18 17:38:53 +0000
permalink:  sinatra_associations_and_joins_table
---


For my Sinatra portfolio project I built a pet app that helps with communication between individuals who have shared responsibility of a pet, or multiple pets in the household. This could be family, significant others, roommates, a nanny, dog walker, etc.  Based on the different scenarios it made the most for the pet to belong to the household, and  the household to have many users. The users would also need to have many households. 

Relationships Breakdown:
* A `many-to-many` relationship between the Users and Household: Users can have many households, and household can have many users. 
* A `has_many`/`belongs_to` relationship between the Household and Pet: a pet belongs to a household, and a household has many pets. 
* A `has_many`/`belongs_to` relationship between a Pet and  an Event: a pet has many events. 

I was also going to need `owner_id` foreign keys for households and pets, in order to have editing and deleting functions, as well as tohelp keep track of the pets if a household is deleted. Later I added an owner_id for event as well, so they could be updated and deleted by the owner, and a possible stretch goal to show which users created the event, i.e. walked or fed the dog.

I chose to create the many-to-many relationship between the users and households with a join table. I created a table called UserHousehold that has two foreign key columns: `household_id` and `user_id`, to connect the user and the household together. 

The `owner_id` being set does not create the relationship between the household and the user, so I needed to make that connection after creating the household. Originally I was trying to create my household and then push the household id into the `current_user.household_ids`:
```
@household = Household.create(name: params["household"]["name"], owner_id: current_user.id)
current_user.household_ids << @household.id
```

But this wasn’t saving the `household.id` to the `current_user.hosuehold_ids`. Although the push method saves and persists in a `has_many `association, in a `has_many_through` relationship, those association are **read only** and the push method will not save and persist in a `has_many_through` relationship.  

> An important caveat with going through has_one or has_many associations on the join model is that these associations are read-only. For example, the following would not work following the previous example:
> ```
> @group.avatars << Avatar.new   # this would work if User belonged_to Avatar rather than the other way around
> @group.avatars.delete(@group.avatars.last)  # so would this
> ```
> in other words, on a direct h-m/b-t relationship, the << is destructive -- it saves and persists -- because the collections are not read-only, but in a h-m-through relationship, they are
> ![]([https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html](http://)http://)

So I could refactor to this:
```
@household = Household.create(name: params["household"]["name"], owner_id: current_user.id)
current_user.update(household_ids: current_user.household_ids.push(@household.id))
```

However, if I create the household through the current user by using the methods ActiveRecord gives through the association, `user.households.create(attributes)` will create the association automatically. 

Letting me refactor to one line: 
`current_user.households.create(name: params["household"]["name"], owner_id: current_user.id)`
