---
layout: post
title:      "Using Scopes to Filter an Index Page"
date:       2020-03-22 16:51:43 -0400
permalink:  using_scopes_to_filter_an_index_page
---


While building my project I made the mistake of writing my scope methods after I was half way finished. I assumed I would be using it for additional functionality that would be easy to add on to a close to finished project, so I built the basic structure first. Once I started building my scope methods I saw how they can handle essential functions of my application. 

I started by building a class methods that looked like this:

```
class Book < ApplicationRecord
  ...
  def self.by_author(author_id)
    where("author_id: ?", author_id)
  end
  ...
end
```

Tested, and they worked! Except they weren’t chainable. Scopes have the same functionality as class methods, and they are chainable! I converted my class method to a scope with the following syntax. 

```
  class Book < ApplicationRecord
    scope :by_author, ->(author_id) { where("author_id: ?", author_id)}
    
    ...
  
  end

```

When I went to test, my scopes weren’t working! Then I read in the following in Ruby and Rails Guides: 

> "Be aware that, when given in the Array format, default_scope query arguments cannot be converted to a Hash for default attribute assignment.”  - https://guides.rubyonrails.org/active_record_querying.html#scopes
> 

I switched the : (colon), to a single = (equals sign) instead of the == (double equals) since we are querying ActiveRecord. 

```
  class Book < ApplicationRecord
    scope :by_author, ->(author_id) { where("author_id = ?", author_id)}
    scope :by_genre, ->(genre_id) { where("genre_id = ?", genre_id)}
    scope :by_page_count, ->(page_count) { where("page_count <= ?", page_count)}
    
    ...
  
  end
```

And it worked! And they were chainable. 

The scope` :by_autho`r takes an argument` ->(author_id)`, this is the author_id that the user wants to retrieve, asks ActiveRecord to give us books ‘`where`’ author_id equals the argumetn author_id that was given in params from the user’s form input. Chaining two scopes together returns all of the books that fit within the parameters given. 

I created a form to filter a list of books by author, genre, and page_count.  In the controller, the index method checks to see how many filters were chosen and then collects an array of `@books`, filtering books by chaining each scope method with the params given by the user. 

```
class BooksController < ApplicationController

  def index 
    ...
    if !params[:author].blank? && !params[:genre].blank? && !params[:page_count].blank?
      @books = Book.by_author(params[:author]).by_genre(params[:genre]).by_page_count(params[:page_count])
    ...
  end
```

If you wanted to see all George Orwell’s book, in the Fiction genre, under or equal to 150 pages, the index show page would include Animal Farm, along with any other books of his that are 150 pages or less in the Fiction genre. Although I decided to keep an index page for Authors and Genres, these scope methods essentially preform the same functionality on one page, the Books index page. 
