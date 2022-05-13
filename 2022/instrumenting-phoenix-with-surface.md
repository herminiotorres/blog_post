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

So, last not least, let's go and implements our tradicional "hello world" in Surface.

We need to create a new route on `lib/my_app_web/router.ex`, and should be called:
```diff
scope "/", MyAppWeb do
  pipe_through :browser

+ live "/count", CounterLive
end
```

Create the `live` folder in `lib/my_app_web/live/`, and then create your `counter_live.ex`.

And the code inside this file should be like this:
```elixir
defmodule MyAppWeb.CounterLive do
  use Surface.LiveView

  data count, :integer, default: 0

  def render(assigns) do
    ~F"""
    <div>
      <h1 class="title">
        {@count}
      </h1>
      <div>
        <button class="button is-info" :on-click="dec">
          -
        </button>
        <button class="button is-info" :on-click="inc">
          +
        </button>
      </div>
    </div>
    """
  end

  def handle_event("inc", _value, socket) do
    {:noreply, update(socket, :count, &(&1 + 1))}
  end

  def handle_event("dec", _value, socket) do
    {:noreply, update(socket, :count, &(&1 - 1))}
  end
end
```

Lets see line by line and understand each one of them.

The `use Surface.LiveView`, it just a wrapper component around `Phoenix.LiveView`. In LiveView we normally use `use MyAppWeb, :live_view`.

The [`data`](https://surface-ui.org/data) macro its the private API for Surface components in general, this means the component doesn't receive any external data when its called, and because this is a live_view itself it make sense has only private API. Also, we could skip to create the [`mount/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:mount/3) function, since we don't doing nothing complext that much to feel necessarily to have a mount function.

The [`render/1`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:render/1) function it will rendering our template, and instead of using the `HEEX` sigil(`~H`), we re been using the Surface sigil(`~F`). And one of the ergonomics for Surface templates its we can use `{}` for embeded code, and not just only for embeded thing in HTML attributes for tags. And it has `:on-click` handle event for `inc` and `dec`, and to get more context for this handle event directive checking out [here](https://surface-ui.org/events#using-the-:on-[event]-directive).

And this its how its look like our first suface page:
![](./surface-count-live.mov)
