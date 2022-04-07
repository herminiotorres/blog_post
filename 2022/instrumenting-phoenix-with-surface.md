# Instrumenting Phoenix with Surface
What is Surface? According to website describe itself as a server-side rendering component library for Phoenix. You can see Surface as a Template DSL to render webpages with a minimal Elixir portion. And why this is good, it because you it be more free to use structures which make more sense and more close to frontend instead of be more functional and close to the language itself. Also, that been said, we can see how to install and use surface with your phoenix project.

Once, you have your phoenix(`mix phx.new my_app`) project, the next step should be add Surface as a dependency in `mix.exs`:
```elixir
def deps do
  [
    {:surface, "~> 0.7.3"}
  ]
end
```

After fetching the dependencies with `mix do deps.get, deps.compile`, you can run the `surface.init` task to update the necessary files in your project. Besides, before that it's good to run `mix help surface.init` first, and check out all the options it can used.

```shell
mix surface.init
Compiling 14 files (.ex)
Generated my_app app

Note: This task will change existing files in your project.

Make sure you commit your work before running it, especially if this is not a fresh phoenix project.

Do you want to continue? [Yn] y
* patching .formatter.exs
* patching .gitignore
* patching assets/js/app.js
* patching config/config.exs
* patching config/dev.exs
* patching lib/my_app_web.ex
* patching mix.exs

Finished running 10 patches for 7 files.
10 changes applied, 0 skipped.

The following dependencies were updated/added to your project:

  * surface_formatter

Do you want to fetch and install them now? [Yn] y
Resolving Hex dependencies...
```

---
Topics I need to cover:
- see the diff to show what were installed.
- create an example and use it, also use sigil_F and .sface extension.
- conclusion
