Hi all, I've been thinking of improve my writing, and I came here to share what I've learned for almost a year using [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) and [Surface-UI](https://surface-ui.org/). I hope you enjoy it!

I will deliver some small bites to get focus on one thing per time. In the beginning, I decided to talk about this concept where came up the React community, which is called a `single source of truth`, and here is the definition from the React documentation:


> [There should be a single `source of truth` for any data that changes in a React application. Usually, the state is first added to the component that needs it for rendering. Then, if other components also need it, you can lift it up to their closest common ancestor.](https://reactjs.org/docs/lifting-state-up.html#:~:text=There%20should%20be%20a%20single,to%20their%20closest%20common%20ancestor.)

Maybe this could tell you something or not. But don't waste to much on this definition, because we will understand with an example.

Looking for my past when I started to learn LiveView... I stayed for a while only using [live_view's](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#content) and assign data through the socket, and creating some [handle_events/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_params/3), pretty much this, right? So far so good.

Next, I started to learn about this new thing called the Stateful and Stateless component, which was it bit hard to understand these concepts in such a way that I can explain to someone else. And these terms have a lot of baggage, and these terms were renamed to call Stateful components [live components](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html#content), and Stateless components as [function components](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#content), which I prefer most.


So, the difference is live components can handle their one state, and function components not. And what does this mean? Remember the handle_events live components could have their own, instead, you put all of them in live_views. Unfortunately, function components don't have handle_event which means live_view will take care of it.

Now, since we are on the same page, let's continue, and please bear with me.

Imagine we have this **Dummy** App, which have it a  `Calculator` for temperature, which you input some value in **_Celsius_** and show it in **_Fahrenheit_**. 

Here are some images before we dig in some code:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wnuafnzhniajf8k5zxyh.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/px3zyty9ilcv6jp0fh2t.png)


First thing, first, we should add a new path to `Router` for our live `Calculator`.

```elixir
# lib/dummy_web/router.ex

scope "/", Dummy do
  pipe_through(:browser)

  live("/", Calculator)
end
```

Since we create our route. We can work on it in our `live_view`. And to keep it simple, we ignore the `params`, and `sessions` in [mount/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:mount/3), and only assign through the `socket`, the initial Fahrenheit value.

In the [render/1](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:render/1) template, we will called our `TemperatureInput` `live_component`, passing the `id`, and the `fahrenheit`. Let's continue and we came back to this [handle_info/2](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_info/2) function after.

```elixir
# lib/dummy_web/live/calculator.ex

defmodule DummyWeb.Calculator do
  use DummyWeb, :live_view

  alias DummyWeb.TemperatureInput

  @impl true
  def mount(_params, _session, socket), do: {:ok, assign(socket, :fahrenheit, 0)}

  @impl true
  def render(assigns) do
    ~H"""
    <main class="hero">
      <.live_component module={TemperatureInput} id={"celsius_to_fahrenheit"} fahrenheit={@fahrenheit}/>
    </main>
    """
  end

  @impl true
  def handle_info({:convert_temp, celsius}, socket) do
    fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

    {:noreply, assign(socket, :fahrenheit, fahrenheit)}
  end
end
```

Looking into our `live_component`, it only has a form with the input field where the user writes a number in Celsius degree and clicks the `submit` button to send this event to the server. On the server-side we have this [handle_event/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_event/3), which it does two things:

- send a message to the parent(the `live_view`).
- calculate the Fahrenheit temperature and assign it through the socket.

Just to give more context, we are using two LiveView [phx-* bindings](https://hexdocs.pm/phoenix_live_view/bindings.html), and they are:

- `phx-submit` - it will send the form to the handle event called `to_fahrenheit`.
- `phx-target` - his tells who sounds handle the event if it's the `live_view` or the `live_component`, and the `@myself` means the `live_component` should be handled.

```elixir
# lib/dummy_web/live/temperature_input.ex

defmodule DummyWeb.TemperatureInput do
  use DummyWeb, :live_component

  def render(assigns) do
    ~H"""
    <div>
      <div class="row">
        <.form let={f} for={:temp} phx-submit="to_fahrenheit" phx-target={@myself} >
          <div>
            <%= label f, "Enter temperature in Celsius" %>
            <%= text_input f, :celsius %>
          </div>
          <div>
            <%= submit "Submit" %>
          </div>
        </.form>
      </div>
      <p>temperature in Fahrenheit: <%= @fahrenheit %></p>
    </div>
    """
  end

  def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
    send(self(), {:convert_temp, celsius})

    fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

    {:noreply, assign(socket, :fahrenheit, fahrenheit)}
  end
end
```

Doing a step back, and looking into the `handle_info/2` in the `live_view`, we can see they are almost the same. But if I change one of them per time and use the `:timer.sleep(5000)`, where we can highlight the issue, when we have more than one place to tell us the truth about the `Fahrenheit` value.

MOV VIDEO 1

MOV VIDEO 2


To prevent that, do we need to think `Fahrenheit` was defined in `live_view` or `live_component`? Since was defined in the `live_view` and passed inside of the `live_component`, the `live_view` should be the source of our truth, and only he should update the value. In another way to explain that is having one place, to tell the truth, you prevent to forgot to update in one of the side, otherwise, you will hook into this issue, and I hope you get the point.

Here is how the `handle_info/2`, should be in the `live_view`:

```elixir
def handle_info({:convert_temp, celsius}, socket) do
  fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

The whole logic how to calculate the temperature between Celsius and Fahrenheit, now only live in the live_view.

And in our `live_component`, the `handle_event` should be like this:

```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})

  {:noreply, socket}
end
```

And the `live_component` only sent a message to the parent(live_view) to convert the temperature. Also it wasn't necessarily to assign new values in the socket, and waiting to the live_view update the data assigns in the socket and our component will be re-rendering and get the Fahrenheit value.

Thank you all, and I hope you enjoy and have fun. So stay tuned for what is coming next.
