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

Lets checking in each file since surface have been installed and understand one by one.

The `.formatter.exs`, this means we don't need to take care of it how to format our surface code since the [suface_formatter](https://github.com/surface-ui/surface_formatter) can handle for us, so when you need to format your suface code, you just run in the terminal `mix surface.format`, and done, all your surface code it will formatted properly.

The `assets/js/app.js`, it will import `Hooks`, and pass with liveSocket. And you can check [here](https://surface-ui.org/js_interop) how to use Hooks with surface, and in case you like to understand how to create and use a Hook with LiveView, you can checking out [here](https://hexdocs.pm/phoenix_live_view/js-interop.html).

The `config/config.exs`, it will set a `ErrorTag` form component when your form has some errors and you can see visualy on the page.

The `config/dev.exs`, it will set a reload compilers, and what does mean? it means every time we change and save our surface component, the compiler it will compiles the newest version of our code, and reload the app in case you can see the changes.

The `lib/my_app_web.ex`, it will import the surface for all the views.

The `mix.exs`, it will have the surface compiler, for how to compile the surface code, and also raise any erro when its has, and add at least two dependencies, and it was `surface` itself, and `surface_formater`, but it could add the `surface_catalogue`.
