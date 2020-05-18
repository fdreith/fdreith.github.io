---
layout: post
title:      "Using JS Date Objects in React-Redux with Rails Backend  "
date:       2020-05-18 16:56:50 +0000
permalink:  using_js_date_objects_in_react-redux_with_rails_backend
---



I built a task app that allows users to create tasks and assign them to other users. To create a due date for tasks I used the react-datepicker package [https://www.npmjs.com/package/react-datepicker](http://). This allows users to pick a date from a calendar. 

I was surprised to learn that I did not need to convert the javascript Date object to a string before sending it to Rails, Rails seems to convert it to a Rails Date object seamlessly. The tasks received from Rails read as though they were converted to javascript format, however, they were a string object instead of a Date object. 

To convert the string to a Date object for the Date Picker was simple. The Date Picker requires you to have an initial state with the value set to` new Date()`. So for my edit form, I simply set it to `new Date(this.props.task.due_date)` to convert the string to a Date Object. 

```
class EditTask extends Component {

  state = this.props.task ?
    {
      content: this.props.task.attributes.content,
      due_date: new Date(this.props.task.attributes.due_date),
      user_id: this.props.task.attributes.user.id,
      owner_id: parseInt(this.props.currentUser.id)
    } :
    initialState
…
}
```
However, I wanted the tasks to be ordered by due_date. This meant I needed to convert all of the task’s due_date attribute to a Date object. I started by attempting to use ActiveRecord scope methods, but because I was getting the tasks when I fetched to get the current_user through the user has_many :tasks association, I was unable to get a default scope method to work. There is a way to apply a scope method to a has_many relationship: 
[https://stackoverflow.com/questions/11636541/use-a-scope-by-default-on-a-rails-has-many-relationship](http://)

Unfortuanately I was unable to find a way to make that a default scope method, and I was unable to get it to work with my serializer because you have to call the scope method.

So I created my own function and put it in my tasks reducer since I wanted the dates to be converted in my Redux Store:

```
const convertDates = (tasks) => {
  if (Array.isArray(tasks)) {
    return tasks.map(task => {
      task.attributes.due_date = new Date(task.attributes.due_date)
      return task
    })
  } else {
    tasks.task.attributes.due_date = new Date(tasks.task.attributes.due_date)
    return tasks
  }
}
```

I converted the tasks anywhere the Redux Store was updating with data from my backend to ensure all dates were converted to a Date object. 

To sort the entries I wrote a function for the Tasks container to sort and then pass down the sorted arrays of tasks to the three children who handle displaying the different types of tasks.

```
const sortByDate = (tasks) => {
  return tasks.sort(function (a, b) {
    const dueDateA = a.attributes.due_date
    const dueDateB = b.attributes.due_date
    if (dueDateA < dueDateB) {
      return -1
    }
    if (dueDateA > dueDateB) {
      return 1
    }
    return 0
  })
}

const mapStateToProps = state => {
  return ({
    currentUser: state.currentUser,
    users: state.users,
    myTasks: sortByDate(state.tasks.myTasks),
    assignedTasks: sortByDate(state.tasks.assignedTasks),
    completedTasks: sortByDate(state.tasks.completedTasks)
  })
}
```

**Displaying Dates:**
**
Displaying dates was a little tricker than it is in Rails, because you need to pull each aspect of the date that you want to display. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date](http://)

```
const displayDateAndTime = (date) => {
  const month = date.getMonth() + 1
  const hour = date.getHours()
  const min = date.getMinutes()
  return `${hour}:${min} on ${month}-${date.getDate()}-${date.getFullYear()}`
}
```

Month: `date.getMonth()` —returns the month, starting with 0 as January, so you need to add a 1 
Day: `date.getDate()` — returns the date of the month (1-31)
Year: `date.getYear()` displays the year in 3 digits, so I used date.getFullYear()
Time: `date.getTime()` — returns in seconds, so I used `date.getHours()` and `date.getMinutes()`
Weekday: `date.getDay() `— returns the day of the week (0-6), so I wrote the following function to display the day of the week.

```
const displayDate = (date) => {
  const days = ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat']
  const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sept', 'Oct', 'Nov', 'Dec']
  const weekday = days[date.getDay()]
  const month = months[date.getMonth()]
  const day = date.getDate()
  const year = date.getFullYear()
  return `${weekday}, ${month} ${day} ${year}`
}
```

The above function also shows how I was able to display the name of the month. 

