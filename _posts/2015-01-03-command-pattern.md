---
layout: post
title: "Command Pattern"
category: "Behavioral Patterns"
tags: ["Intro", "Command Pattern"]

---
{% include JB/setup %}

*The command pattern*
is one of many Design Patterns first introduced by the [Gang of Four](http://en.wikipedia.org/wiki/Design_Patterns).
The book Design Patterns: Elements of Reusable Object-Oriented Software is one of the few de facto books about software
that has become required reading for many reasons. The command pattern is one of the first covered in the book, I believe it to be one of the most
useful, and have seen it implemented many times in my professional career.

Full code for the following tutorial can be found here: [Command Pattern](https://github.com/Alistanis/DesignPatterns/tree/master/app/models/command), and
API documentation generated from that code is here: [Command Pattern API docs](http://alistanis.github.io/DesignPatterns/doc/Patterns/Command.html).

## Getting started

I'll begin by assuming you have a development environment set up for Ruby.
MacOSX comes with a system Ruby which, on Yosemite, is 2.0, but prior to Yosemite was 1.8.7. I recommend installing and using [RVM](https://rvm.io/) to manage different
versions of Ruby, but this code should work just fine on 2.0 either way.
At the time of this writing my dev environment is as follows:

* MacOSX 10.10.1
* RVM 1.26.7
* Ruby 2.1.5
* Homebrew 0.9.5
* Rubymine 7.0.2 (my personal preference)

All of the code is tested and run for this environment, and it should work on Linux and Windows, but I haven't tested it. Please let me know if you have any trouble
running the samples and I'll do my best to assist you.

## Starting out

The command pattern is defined as "a behavioral design pattern in which an object is used to represent and encapsulate all
the information needed to call a method at a later time." Generally a command class will contain at least one method - execute, though often it will contain others as well,
the most popular being undo. Undo is an example of the Memento pattern and has a lot of great uses. Could you imagine not being able to undo an accidental deletion
of an entire page of text? Chances are good the underlying code in your favorite text editor uses some variation of Command and Memento.

Today we're going to build a Base Command Class and then we'll build a Create File class that will serve as our first concrete command. I'll also cover a few points
on inheritance and [Proc Objects](http://ruby-doc.org/core-2.2.0/Proc.html) and [blocks](http://rubylearning.com/satishtalim/ruby_blocks.html),
so if you're new to Ruby you should still be able to follow along. If you have trouble with any of the concepts here,
feel free to leave a comment below and I'll try to clear it up.

Let's start defining our base class and its first few methods and fields - note - more detailed comments are contained in the actual code and the API docs.

    {% highlight ruby %}

class Command

    # stores the description of the command
    attr_accessor :description
    # the most recent status of the command
    attr_accessor :status

    # Initializes the Command Class
    # Designed to be called by a child class in its initialize method
    #
    # * +description+ - The description of the sub class
    #
    # Examples
    #   => #inside child class
    #   => def initialize
    #   =>  desc = 'A new child class of Command'
    #   =>  super(desc)
    #   => end
    def initialize(description)
      @description = description
      @status = 'initialized'
    end

    # Called by the child class' execute function
    #
    # * +function+ - The actual command that will be executed
    #
    # Examples
    #
    #   => #inside child class
    #   => def execute
    #   =>  function = Proc.new do # or Proc.new {}
    #   =>    #code to be executed
    #   =>  end
    #   =>  super(function) # (calls the super class' execute function defined below)
    #   => end
    def execute(function)
      @status = "Command began: #{@description}"
      function.call
      @status = "Execution completed: #{@description}"
    end

    # Called by the child class' undo function.
    #
    # * +function+ - The command that will be undone
    #
    # Examples
    #
    #   => #inside child class
    #   => def undo
    #   =>  function = Proc.new do
    #   =>    #code to undo execute function
    #   =>  end
    #   => super(function)
    #   => end
    def undo(function)
      @status = "Undo began: #{@description}"
      function.call
      @status = "Undo completed: #{@description}"
    end
end

    {% endhighlight %}

We've defined a base class that will act as an interface for the other commands we're going to implement. It has two fields, a description and a status,
both of which are initialized in the initialize method. It has two other methods: execute and undo. Both of those methods take an argument called 'function'
that will be a [Proc Object](http://ruby-doc.org/core-2.2.0/Proc.html) defined in our child classes.

Note: It is not required to implement additional functionality into the parent class (Command) to follow the Command Pattern as it's traditionally written.
All of our functions could be written in child classes and we could use the Command class as a true abstract class,
but I like my commands to have a little bit more flexibility - for instance, think about how easy
it would be to add logging functionality to every command simply by changing a few things in the parent class.

Now let's look at implementing a concrete Command in a child class, CreateFile. CreateFile will inherit from our Command class:

    {% highlight ruby %}

class CreateFile < Command

    # Initializes the CreateFile Class
    #
    # * +path+ - The file path to write the file
    # * +data+ - The data to write to the file
    #
    # Examples
    #
    #   => create_file_cmd = CreateFile.new(file_path, data_to_write)
    def initialize(path, data)
      super("Create File: #{path}")
      @path = path
      @data = data
    end

    # Creates a file and writes data to it; will overwrite data
    #
    #
    # Examples
    #
    #   => create_file_cmd.execute
    def execute
      function = Proc.new do
        f = File.open(@path, 'w')
        f.write @data
        f.close
      end
      super(function)
    end

    # Undoes file creation;
    #
    # Examples
    #
    #   => create_file_cmd.undo
    def undo
      function = Proc.new do
        File.delete(@path)
      end
      super(function)
    end

end

    {% endhighlight %}

As we can see above, we have implemented all of the same named functions in the superclass (Command) in the child class (CreateFile). In those functions we explicitly
call the superclass function, which is designed to provide the same functionality to any Command we'd like to implement. In the execute and undo functions, we've created
a Proc object that contains the instructions for writing some data to a file and undoing that write respectively.

Note: The function could be passed into the superclass other ways as well

Parent class examples of alternate methods:

    {% highlight ruby %}

# as a block and an explicit call - block is required
def execute(&block)
  block.call
end

# as a block that yields - block is required
def execute(&block)
  yield
end

# as a block that yields, block is optional
def execute
  yield if block_given?
end

    {% endhighlight %}

Child class examples:

    {% highlight ruby %}
# calling the execute method looks the same for all three parent class examples
execute{ puts 'Hello!' }

# for the original example:
function = Proc.new { puts 'Hello!' }
execute(function)

# or this way
function = Proc.new do
  puts 'Hello!'
end
execute(function)
    {% endhighlight %}

For a more in depth look at Procs, blocks, and Lamdas in Ruby, this is the best writeup I've seen on it:

* [http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/)

Because we implemented our concrete class as a child class of the Command class, we can call super inside of our execute function.
In Ruby, if you write a method in a child class that has the same name as a method in the superclass, you can call the superclass method inside the child class simply
by calling super:

    {% highlight ruby %}

class ChildCommand < Command

  def execute
    function = Proc.new { puts 'Hello!' }
    super(function) # calls Command.execute(function)
  end

end
    {% endhighlight %}

If we don't call super, the superclass function will be overridden by whatever we define in the child class - example below.

    {% highlight ruby %}
class A
  def initialize
    puts 'Hello'
  end
end

class B < A
  def initialize
    puts 'Hi'
  end
end

b = B.new
=> Hi
    {% endhighlight %}

Above, class B inherits from class A, and we can see that when class B is instantiated, its output is 'Hi' intead of 'Hello' because the superclass(A) initialization
function (constructor) has been overridden by the child class(B).



## Conclusion
Now we've seen how the basic command pattern works by defining an abstract interface for executing commands and for undoing them.

Sources:

* [Design Patterns: Elements of reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Object-Oriented-Professional-Computing/dp/0201634988)
* [Game Programming Patterns](http://gameprogrammingpatterns.com/) - Absolutely excellent book by Bob Nystrom.
* [Ruby Design Patterns](http://designpatternsinruby.com/) - Admittedly, though my implementation is different, I took the idea for FileIO and the command pattern from another excellent book by Ross Olsen.
* Countless Stack Overflow and Google searches throughout the years


## Up next:

Next time we'll cover a few more commands, a command list, and then we'll write a full suite of Rspec tests to make sure our commands do what we expect them to!