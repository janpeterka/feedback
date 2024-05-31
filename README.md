# feedback v0.0.0
_what? v0.0.0? is that even legal?_  

I don't care.
This project is in my head and some parts of code in different project for some time, and I wanted to create a place where I will thoroughly describe it before it will become reality.  
Writing about **what it needs to** do and **how developer will interact with it** helps me make sense of it and how it will be implemented.

# premise
Collect feedback in Rails app with ease

## what does that mean?
You probably want to collect feedback from your users.  
It allows them to report bugs and problems, and give you ideas about what can be improved.  
That helps you to understand their needs and wants and make your app the best it can be.  
But how to collect feedback?  

There are for sure many ways:
- just stick link to Google form somewhere for them!
- connect some supercomplex tool like Adoric or I don't know.
- tell them to create issues in your Github/Gitlab/Gitwhatnot repository (as if that will happen).
- create your own in-app solution

I went with the last one.  
Having everything in one place is nice - for me and for users.  
I can easily customize everything.  
I have full control.  

Ok, but creating this in every new app I write is..  
tedious.

I don't like that.  
I want to have plug-and-play solution.

And here we are - `feedback` plugin for Rails apps!  
Without further ado:

# overview

## needs
User wants to **post feedback** (possibly with attachments - screenshots of bug or similar)  
User wants to **see status** of feedback, and possible reactions from _someone (you)_  
  -> User wants to **be notified** if something new happens  
User should be able to **provide additional information** (add note/reaction to feedback)  

Admin should be **notified** on new feedback.  
Admin can change status, ask questions, explain situation,..  

## level 1 - local Feedback::Post and Feedback::Comment

### how it works
Core of this gem is `Feedback::Post` ActiveRecord class, which represents users feedback (duh).
They provide that through from on `/feedback/new` page (by default).

`Feedback::Post` has following attributes:
  - timestamps
  - `author_id`
    this is expected to point to `User` class, but if your applications uses some other thing (like `Member`), this needs to be configured (more on that later)
  - `status` - enum with `new, open, closed, completed` states
  - `text` - I decided **not** to make this `ActionText`, as not everyone uses that, and it's not really important to have that smart text. Maybe I will add support for `ActionText` later.
  - attachments - which **are** `ActiveStorage`. If you don't use that, it should be possible to just use form without file section, so nothing can be uploaded.

On `feedback/`, user can see all their feedbacks (filtering included), and go to detail.

There can be followups on posts - so we have `Feedback::Comment`. That is extremely simple - it has timestamp, author and text.

Admin has their own endpoint - `feedback/admin/`.


### how to use this in my app
:one: Generate migrations  
:two: Migrate  
:four: Configure  
:three: Mount engine  

### permissions
Only author and admins are permitted to view feedback post.

### configurations
Class of author (default: `User`)  
Method to specify current author (default: `current_user`)  
  -> I expect you to use `current_user`, but if that's not a case, you can override that by setting `current_feedback_author` in your `ApplicationController` (will this work? it should if we inherit `Feedback::ApplicationController`   from `::ApplicationController`)   [!! not currently implemented]

## level 2 - notifications

For notification, I was thinking about how it could integrate with existing notifications in my app, but then I realized that's nonsense. If I want that, I should write this as part of my app, not use gem/engine! Feedback should be as separate from rest of application as possible.
So we need separate notifications. Let's keep it simple:
`Feedback::Notification` has `timestamp`, `recipient_id`, `message` and `read_at`.
It's generated in these moments:
- on creation - for admin
- on status change - for author
- on new comment

It's marked read on opening detail of given feedback post, or manually from "dashboard"

## level 3 - external systems
Maybe you don't want feedback in your app. Maybe you want it in your Gitlab, where you collect and manage all issues!
Ok, then maybe you want some other solution. But maybe I have something for you too!

As I have this need too, I came up with `Synchronizers` - classes that allow you to synchronize in-app feedback with external system.
Out-of-box will be `Gitlab` connector (as I need that), more can be added.

This makes things bit more complicated though. We need some additional columns - `external_url` and `external_id`, and `last_synchronized_at`. Both on Post and Comment.
Then we need `ActiveJob` to manage our synchronization, as we really want this as background job (trust me - all this came from frustration as sometime users couldn't post feedback from app to Gitlab (or read it), as API was not working or was slow. so we decided for in-app-first approach).
There will also be some periodical job to get new comments from our external place.  

Keeping things two-way synchronized can be tricky. There can be mistakes. The systems are always bit different. I don't recommend it. But here you are. 
_Later I'll add some instructions on how to create different Synchronizer and how to configure them, I have no idea now_


### references
[Rails guide on Engines](https://guides.rubyonrails.org/engines.html)  
[How to set customization](https://medium.com/@asimadnan/using-a-model-provided-by-the-application-inside-rails-engine-cddc119749d2)  

existing (outdated) gems:  
[feedback]([url](https://github.com/jsboulanger/feedback)) (~12 years no activity)
[pointless-feedback]([url](https://github.com/vigetlabs/pointless-feedback)) (~2 years no activity)

