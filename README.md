# CLIpso

This is my rough attempt to port as much as I can of [Belafonte](https://github.com/ess/belafonte) to Crystal.

In short, this library is all about making CLI applications.

## Installation


Add this to your application's `shard.yml`:

```yaml
dependencies:
  clipso:
    github: ess/clipso
```


## Usage

CLIpso can be used both to create traditional single-purpose applications (like `cat`, `ls`, `make`, etc), but it can also be used to create "command suite" applications (like `git`, `rails`, `heroku`, etc).

### Traditional Applications ###

As mentioned above, "traditional" applications are single-purpose utilities. They tend to do one thing, with the goal of doing that one thing exceptionally well.

```crystal
require "clipso"

class MyApp < CLIpso::App
  # Every CLIpso application must have a title.
  title "myapp"
  summary "A short description"
  description "This is a much longer description of the app, command, or what have you."
  
  # Switches are boolean flags that can be passed on the command line.
  # At least one of a short flag or a long flag must be given.

  switch :switch_name,              # name the switch (required)
    short: "s",                     # short flag for the switch
    long: "switch",                 # long flag for the switch
    description: "This is a switch" # describe the switch (default "")

  # Options are flags that take an argument. The basic setup for this is the
  # same as that for switches, but with the additional requirement of a
  # display name for the option's argument.

  option :option_name, # name the option (required)
    short: "o",                       # short flag
    long: "option",                   # long flag
    description: "This is an option", # describe the option (default "")
    argument: "option name"           # display name (required)

  # Args are actual arguments to the command. These require a name, and by
  # default are expected 1 time. You can specify any number of explicit
  # occurrences that are greater-than 0, and you can also make them unlimited,
  # but you're not allowed to add any further args after adding an unlimited
  # arg.

  arg :argument_name,
    times: 1

  arg :unlimited_argument,
    times: unlimited

  # The `handle` method that you define for your app is the hook that CLIpso
  # uses to run your code. If you don't define this, your app won't actually
  # do much of anything.

  def handle
    # Do something if the :switch_name switch is active

    if switch_active?(:switch_name)
      stdout.puts "Switch is active!"
    else
      stdout.puts "Switch is not active :/"
    end

    # Do something based on the options that came in
    stdout.puts "We got this option: #{option(:option)}"

    # Do something with the first argument we defined
    stdout.puts arg(:argument_name).first

    # Do something with the unlimited argument we defined
    arg(:unlimited_argument).each do |unlimited|
      stdout.puts "unlimited == '#{unlimited}'"
    end
  end
end

# Actually run the application. In addition to ARGV, the following .new args
# are defaulted:
#
# * stdin (default: STDIN)
# * stdout (default: STDOUT)
# * stderr (default: STDERR)
#
# In a more perfect world, your executable file is basically just a require to
# pull in your app, then this line.

exit MyApp.new(ARGV).execute!
```

### Command Suite Applications ###

Command suite applications (CSAs) are, in the simplest scenario, a collection
of Traditional Applications that are collected under a wrapper to provide a
uniform user interface.

CLIpso supports this sort of application through mounting, which is very
similar in spirit to, say, mounting one Rack application under another. The API is the same as the Traditional App API with two exceptions:

* Use `mount App` to mount an application as a subcommand
* Apps that mount other apps cannot have "unlimited" args

Here's an example:

```crystal
require "clipso"

class InnerApp < CLIpso::App
  title "inner"

  def handle
    stdout.puts "This is the inner app"
  end
end

class OuterApp < CLIpso::App
  title "outer"

  mount InnerApp

  def handle
    stdout.puts "This is the outer app"
  end
end

exit OuterApp.new(ARGV).execute!
```

## Development

TODO: Write development instructions here

## Contributing

1. Fork it ( https://github.com/[your-github-name]/clipso/fork )
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create a new Pull Request

## Contributors

- [[your-github-name]](https://github.com/[your-github-name]) Dennis Walters - creator, maintainer
