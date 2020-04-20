---
layout: post
title:      "Using Associations and Serializers to Make Code More Efficient"
date:       2020-04-20 02:05:40 +0000
permalink:  using_associations_and_serializers_to_make_code_more_efficient
---


When building my first javascript frontend with a rails backend, I thought it would make the most sense to have the database be the source of truth for my data. That way when a user added data, or deleted data (in this case a journal entry) I would simply have to grab data from that one model in the database and nothing else, and not have to worry about updating the frontend. 

The Landscape for this Journal App:

Entry model:
belongs_to :prompt

Prompt model:
belongs_to :mood
has_many :entries

Mood model:
has_many :prompts
has_many :entries, through: :prompts

Originally I had two fetch requests to retrieve entries and prompts, but with this I needed to hardcode in the moods, which wasn’t difficult because there were only 7, and I was able to use the id’s of mood to grab associated prompt’s mood_id attribute, but I found I wasn’t able to access the mood information in my entries beyond the mood_id. I created a class function in entries class to find the mood object based on the prompt of the entry. 

```
class Entry {

  constructor(data) {
    this.id = data.id
    this.prompt = this.prompt
    this.mood = this.findMood(data)
    this.minutes = data.minutes
    this.content = data.content
    this.created_at = new Date(data.created_at)
  }

  findMood(data) {
    return Mood.all.find(mood => mood.id === data.prompt.mood_id)
  }
} 
```

Then I found I was also having issues displaying prompt and mood data on each entry because I was grabbing entries either by the entries controller index action, where I had the prompt info, or when a user wanted to view entries by mood, data was coming from the mood show page with the associated entries, where I had the prompt_id, so I needed to create another function to find the prompt for that entry, and I needed to control for either having the prompt object available, or just the prompt_id. 

```
class Entry {

  constructor(data) {
    this.id = data.id
    this.prompt = this.findPrompt(data)
    this.mood = this.findMood(data)
    this.minutes = data.minutes
    this.content = data.content
    this.created_at = new Date(data.created_at)
  }

  findPrompt(data) {
    if (!!data.prompt) {
      return data.prompt
    } else {
      return Prompt.all.find(prompt => prompt.id === data.prompt_id)
    }
  }

  findMood(data) {
    if (!!data.prompt) {
      return Mood.all.find(mood => mood.id === data.prompt.mood_id)
    } else {
      return this.findPrompt(data).mood
    }
  }
}
```

And both of these functions needed to run everytime the data was loaded, which was everytime a user wanted to see entries by a particular mood, or created a mood, or deleted a mood. 

There has to be a better way!

Because Mood has_many of both of the other models, Mood actually has access to both prompts and entries through serialization. By “including” :prompts and “entries” to the render json in the Mood controller, the prompts and entries are available through each mood in the index controller action (or of one mood in the show controller action). 

Although you can serialize your data in rails via the render json in the controller, I ended up using a gem to serialize because there wasn’t a way to exclude specific data from the models that are being included (prompts and entries). I used Active Model Serializers which was very easy to use and set up. [ https://github.com/rails-api/active_model_serializers](http://)

This set up allows me to use one fetch request to accomplish what I was retrieving in 3 fetch requests. All prompts, all moods, all entries. And simplified the login in the findMood and findPrompt entry class functions. 

```
class Entry {
  static all = []

  constructor(data) {
    this.id = data.id
    this.prompt = this.findPrompt(data)
    this.mood = this.findMood(data)
    this.minutes = data.minutes
    this.content = data.content
    this.created_at = new Date(data.created_at)
    this.save()
  }

  save() {
    Entry.all.push(this)
  }

  findPrompt(data) {
    return Prompt.all.find(prompt => prompt.id === data.prompt_id)
  }

  findMood(data) {
    return this.findPrompt(data).mood
  }
}
```

Note from the code above I also decided to save the entries in a static ‘all’ method, to reduce the amount of code that needed to run everytime an entry was added, deleted, or viewed by a particular mood. With this change I simply add the single new entry to the static all entries array, with the following line with the return JSON from the posted entry.

 `new Entry(responseJSON)`
 
 And I needed to delete the instance of the entry from the static all array, this the following function, with the returned JSON from the delete fetch request. 
 
```
 function deleteEntryFromAll(id) {
  for (let i = 0; i < Entry.all.length; i++) {
    if (Entry.all[i].id === parseInt(id)) {
      Entry.all.splice(i, 1); i--
    }
  }
}
```

I wanted to make this function a class function, but because the returned JSON from the delete fetch request was only the id of the entry deleted, it would have been more code to create a function to find the entry by it’s id and then pass a delete function to that instance, and then find that instance in the array of all entries.
