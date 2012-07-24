---
layout: getting_started
title: Mix
---

# Mix

Elixir ships with a few applications to make building and deploying projects with Elixir easier and Mix is certainly their backbone.

Mix is a build tool that provides tasks for creating, compiling, testing (and soon deploying) Elixir projects. Mix is inspired by the [Leiningen](https://github.com/technomancy/leiningen) build tool for Clojure and was written by one of its contributors.

In this chapter, you will learn how to create projects using `mix`, install dependencies and create your own tasks.

## 1 Getting started

In order to start your first project, simply use the `mix new` command passing the path to your project. For now, we will create an application called `my_project` in the current directory:

    mix new ./my_project

Mix will create a directory named `my_project` with few files in it:

    .gitignore
    README.md
    mix.exs
    lib/my_project.ex
    test/test_helper.exs
    test/my_project_test.exs

Let's take a brief look at some of these.

### 1.1 mix.exs

This is the file with your projects configuration. It looks like this:

{% highlight ruby %}
defmodule MyProject.MixFile do
  use Mix.Project
  
  def project do
    [
      app: :my_project,
      version: "0.0.1"
    ]
  end

  # Configuration for the OTP application
  def application do
    []
  end
end
{% endhighlight %}

Our `mix.exs` is quite straight-forward. It defines two functions, `project` which should return the project configuration, for example, where to find the source files, the application name and version. And another function named `application` which allow us to generate an application according to the Open Telecom Platform (OTP) that ships with Erlang. We will talk more about these later.

### 1.2 lib/my_project.ex

This file simply contains the definition of our project main module with a `start` function:

{% highlight ruby %}
defmodule MyProject do
  def start do
    :ok = :application.start(:my_project)
  end
end
{% endhighlight %}


The `start` function invokes the erlang module `application` and tells it to start our application.

### 1.3 test/my_project_test.exs

This file contains a stub test case for our project:

{% highlight ruby %}
Code.require_file "../test_helper", __FILE__

defmodule MyProjectTest do
  use ExUnit.Case
  
  test "the truth" do
    assert true
  end
end
{% endhighlight %}

It is important to note a couple things:

1) Notice the file is an Elixir script file (`.exs`). This is convenient because we don't need to compile test files before running them;

2) The first line in our test is simply requiring the `test_helper` file in the same directory as the current file. As we are going to see, the `test/test_helper.exs` file is responsible for starting the test framework;

3) Then we define a test module named `MyProjectTest`, using `ExUnit.Case` to inject default behavior and define a simple test. You can learn more about the test framework in the [ExUnit](/getting_started/ex_unit.html) chapter;

Since this file is a script file (`.exs`) and it also requires `test_helper.exs`, responsible for setting up the test framework, we can execute this file directly from the command line, which is very useful when you want to run a specific test and not the whole test suite, try it:

    $ elixir test/my_project_test.exs

### 1.4 test/test_helper.exs

The last file we are going to check is the `test_helper.exs`, which simply loads our application and sets up the test framework:

{% highlight ruby %}
MyProject.start
ExUnit.start
{% endhighlight %}

And that is it, with our project created. We are ready to move on!

## 2 Exploring

Now that we created our new project, what can we do with it? In order to check the commands available to us, just run the task `help`:

    $ mix help

It will print all the tasks available. You can get further information by invoking `mix help TASK`.

Play around with the available tasks, like `mix compile` and `mix test`, and execute them in your project to check how they work.

## 3 Compilation

Mix can compile our project for us. The default configurations uses `lib/` for source files and `ebin/` for compiled beam files, you don't even have to provide any compilation-specific setup but if you must, some options are available. For instance, if you want to put your compiled files in another directory besides `ebin`, simply set in `:compile_path` in your `mix.exs` file:

{% highlight ruby %}
def project do
  [compile_path: "ebin"]
end
{% endhighlight %}

In general, Mix tries to be smart and compile only when necessary.

You can also note that, after you compile for the first time, Mix generates an `my_project.app` file inside your `ebin` directory. This file specifies an Erlang application and it holds information about your application, for example, what are its dependencies, which modules it defines and so forth. In our `MyProject.start` function, when we call `:application.start(:my_project)`, Erlang will load the `my_project.app` file and process it. For instance, if there are any dependencies missing, it will let us now.

## 4 Tasks

In Mix, a task is simply an Elixir module named with a `Mix.Tasks` prefix and a `run/1` function. For example, the `compile` task is a module named `Mix.Tasks.Compile`.

Here is a simple example task:

{% highlight ruby %}
defmodule Mix.Tasks.Hello do
  use Mix.Task

  @shortdoc "This is short documentation, see"

  @moduledoc """
  A test task.
  """
  def run(_) do
    IO.puts "Hello, World!"
  end
end
{% endhighlight %}

This defines a task called `hello`. In order to make it a task, it defines a `run` function that takes a single argument that will be a list of binary strings which are the arguments that were passed to the task on the command line or from another task calling this one.

When you invoke `mix hello`, this task will run and print `Hello, World!`. Mix uses its first argument to lookup the task module and execute its `run` function.

You're probably wondering why we have a `@moduledoc` and `@shortdoc`. Both are used by the `help` task for listing tasks and providing documentation of them. The former is used when `mix help TASK` is invoked, the latter in the general listing with `mix help`.

Besides those two, there is also `@hidden` that, when set to true, marks the task as hidden so it does not show up on `mix help Task`. Any task without `@shortdoc` also won't show up.

### 4.1 Common API

When writing tasks, there are some common mix functionality we would like to access. There is a gist:

* `Mix.project` - Returns the project configuration under the function `project`; Notice this function returns an empty configuration if no `mix.exs` file exists in the current directory, this allows many mix functions to work even if a `mix.exs` project is not defined;

* `Mix.Project.current` - Access the module for the current project, this is useful in case you want to access special functions in the project. It raises an exception if no project is defined;

* `Mix.shell` - The shell is a simple abstraction for doing IO in Mix. Such abstractions make it easy to test existing mix tasks. In the future, the shell will provide conveniences for colored output and getting user input;

* `Mix.Task.run(task, args)` - This is how you invoke a task from another task in Mix; Notice that if the task was already invoked, it works as no-op;

There is more to the Mix API, so feel free to [check the documentation](/docs/latest/Mix.html), with special attention to [`Mix.Task`](/docs/latest/Mix.Task.html) and [`Mix.Project`](/docs/latest/Mix.Project.html).

### 4.2 Namespaced Tasks

While tasks are simple, they can be used to accomplish complex things. Since they are just Elixir code, anything you can do in normal Elixir you can do in Mix tasks. You can distribute tasks however you want just like normal libraries and thus they can be reused in many projects.

So, what do you do when you have a whole bunch of related tasks? If you name them all like `foo`, `bar`, `baz`, etc, eventually you'll end up with conflicts with other people's tasks. To prevent this, Mix allows you to namespace tasks.

Let's assume you have a bunch of tasks for working with Riak.

{% highlight ruby %}
defmodule Mix.Tasks.Riak do
  defmodule Dostuff do
    ...
  end

  defmodule Dootherstuff do
    ...
  end
end
{% endhighlight %}

Now you'll have two different tasks under the modules `Mix.Tasks.Riak.Dostuff` and `Mix.Tasks.Riak.Dootherstuff` respectively. You can invoke these tasks like so: `mix mongodb.dostuff` and `mix mongodb.dootherstuff`. Pretty cool, huh?

You should use this feature when you have a bunch of related tasks that would be unwieldly if named completely independently of each other. If you have a few unrelated tasks, go ahead and name them however you like.

## 5 Lots To Do

Mix is still very much a work in progress. Feel free to visit [our issues tracker](https://github.com/elixir-lang/elixir/issues) to add issues for anything you'd like to see in Mix and feel free to contribute.