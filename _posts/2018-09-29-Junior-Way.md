---
layout: post
title: Junior way - How to start the adventure with Ruby on Rails?
---

Recently I was preparing a list of things to learn for my friend who decided to change his profession. There are a lot of books and tutorials, but it would be wise to plan the learning process. I thought it would be a good idea for my first post.

![_config.yml]({{ site.baseurl }}/images/posts/junior-way/technology_meme.jpg)
{: style="text-align: center;"}

First of all, to make all this sense, you must first know where you are going to. Is Ruby on Rails good for you? So what can you do as Ruby dev?
* Build Internet applications from scratch
* Implement and develop business logic
* Organize the databases
* Create APIs

Technologies you should learn:
* Ruby and his frameworks - Ruby on Rails, Sinatra
* HTML, basic CSS
* At least one SQL database (PostgreSQL)
* The basis of design patterns
* Version control system (GIT)

The last stage of learning should be a project that you can put in your CV.

## So, how can I start?

Run your internal patience. When I started my journey with Ruby, I closed myself for half a year at home and spend a lot of time with books and code editor. It is impossible to create a ready-made prescription, probably not everyone has the predisposition to become a programmer. I will show you my recipe, my way to becoming a Junior Ruby developer.

### Stage 1
When I started learning Ruby, I already knew the basics of PHP, SQL, HTML, CSS, if you don't - start from HTML & CSS. 
#### TODO: HTML website with included CSS file.

### Stage 2
Set your work environment. If you are using MacOS - install [iTerm](https://www.iterm2.com). This tool will become an inseparable element of your work. I recommend you to set a hotkey for iTerm:

![_config.yml]({{ site.baseurl }}/images/posts/junior-way/hotkey.png)
{: style="text-align: center;"}

Thanks to this you will have quick access to the terminal. 
You will need a code editor - you can install at the beginning Atom or Sublime. I also recommend you to make a one catalog for all your new projects.

### Stage 3
![_config.yml]({{ site.baseurl }}/images/posts/junior-way/github.png)
{: style="text-align: center;"}
Before you start learning Ruby - learn GIT! It will be extremely useful in creating and debugging your future applications.
* Create your account at [GitHub](http://github.com)
* Read what GIT is and what it's needed for (it's really important!)
* [The official guide](https://guides.github.com)
* [Interactive course](https://learngitbranching.js.org)

#### TODO: Create your first repository on GitHub and upload there to the 'master' branch your HTML project. Then create a locally new branch, make changes to the code and push it to a remote repository. After that on GitHub you have to enter to your new branch and make a Pull Request to the master branch.

### Stage 4
Ruby installation through the version manager: you can choose RBenv or RVM, I use the [first one](http://rbenv.org) - [here you have tutorial](ihttps://gist.github.com/stonehippo/cc0f3098516fb52390f1) except that you can use the newer version of ruby (2.5.1 at this moment) instead 1.9.3

Create the hello_world.rb file in your editor. Enter in it: 
```ruby
puts 'Hello world'
```
You can start the application by going to the directory with your application in terminal and then entering 'ruby hello_world.rb'. The terminal will print  the result. 

Well done - you wrote the first program that every programmer starts with!

#### TODO: Create a directory for the new application. Write it, remember that the name contains the extension '.rb' <- ruby. For GIT practice, put this program in your github repository.

### Stage 5
It's time to meet Ruby!

I and many programmers I know have started learning [from one guide(PL)](http://www.apohllo.pl/dydaktyka/ruby/intro)

There is a lot of books about first steps with Ruby. I have in my collection two:
* The Book of Ruby - Huw Collingbourne
* Learn Ruby the Hard Way - Zed A. Shaw and Rob Sobers

#### TODO: Create a directory for the new application. Write an application that will use at least one class, one 'if' statement, one loop, one mathematic operation, one operation on string. Then push it into your github repository.

### To be continued.
If you will know Ruby well, it's time to get to know its frameworks.
