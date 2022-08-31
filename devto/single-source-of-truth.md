Hey all, I have been working with Phoenix [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) and [Surface-UI](https://surface-ui.org/)  for almost a year, and I have learned some good practice which I would like to share. I hope you enjoy it!

The first concept I like to talk about is that which came up in the React community, which is called a `single source of truth`, and here is the definition from the React documentation:

> [There should be a single `source of truth` for any data that changes in a React application. Usually, the state is first added to the component that needs it for rendering. Then, if other components also need it, you can lift it up to their closest common ancestor.](https://reactjs.org/docs/lifting-state-up.html#:~:text=There%20should%20be%20a%20single,to%20their%20closest%20common%20ancestor.)

I don't have any background with front-end libraries and have never used React before. I ran into this problem a few times before [Marlus](https://github.com/msaraiva) told me about this concept and where it was coming from.

Don't underestimate this concept, and you will probably see it a couple times.

## Setup
Imagine an application called **Dummy** that has a **Calculator** for **Temperature**, where you input values in **Celsius** and show them in **Fahrenheit**.

The application will be like this:

![Celsius Input Form](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wnuafnzhniajf8k5zxyh.png)

![Convert Celsius in Fahrenheit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/px3zyty9ilcv6jp0fh2t.png)

### router.ex

```elixir
scope "/", Dummy do
  pipe_through(:browser)

  live("/", Calculator)
end
```

First thing first, we should add a new path to the **Router** for our live **Calculator**.

### live/calculator.ex

```elixir
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

Since the route was created, the next step is to work on it in the [live_view](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#content). And to keep it simple, we will ignore the `params` and `sessions` in [mount/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:mount/3), and only assign the default Fahrenheit value through the socket.

In the [render/1](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:render/1) template, we will call **TemperatureInput** `live_component` and pass the id and the **Fahrenheit** value. Let's continue, and later on, we will come back to the [handle_info/2](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_info/2) function.

### live/temperature_input.ex

```elixir
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

Looking into the [live_component](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html#content), it only has a form with an input field where the user types a value in **Celsius** degrees and clicks the submit button to send this event to the server, which has a [handle_event/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_event/3), and does two things:

- send a message to the parent(the live_view).
- calculate the Fahrenheit temperature and assign it through the socket.

Just to give more context, we are using two LiveView [phx-* bindings](https://hexdocs.pm/phoenix_live_view/bindings.html), and they are:

- `phx-submit` - it will send the form to the handle event called **to_fahrenheit**.
- `phx-target` - it tells who should handle the event. The default behaviour is always the **live_view**, otherwise we can set the target. Using **@myself** to set the target is our **live_component**.

You have most likely noticed that the **handle_info** in **live_view** and the **handle_event** in **live_component** have the same code. The only difference between them is the `send()` function, where we can send messages between processes, and we are using the `send()` function to send messages to the **live_view** to update the Fahrenheit value.

## The Issue

At the moment, the formula to convert Celsius to Fahrenheit is duplicated on both sides in the live_view and the component. And we must avoid the duplicity of logic.

And what if the formula to convert only changes one side? I will use the `:time.sleep` and change the formula.

### change the logic in live_component

#### live/temperature_input.ex

```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})
  
  # wrong formula
  fahrenheit = (String.to_integer(celsius) - 32) * 5 / 9

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

#### live/calculator.ex

```elixir
def handle_info({:convert_temp, celsius}, socket) do
  :time.sleep(4000)

  fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

https://user-images.githubusercontent.com/1036970/187585732-63547a87-afb7-4ab5-8872-f3ad8d48ac36.mov

### change the logic in live_view

#### live/temperature_input.ex

```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})
  
  fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

#### live/calculator.ex

```elixir
def handle_info({:convert_temp, celsius}, socket) do
  :time.sleep(4000)
  
  # wrong formula
  fahrenheit = (String.to_integer(celsius) - 32) * 5 / 9

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

https://user-images.githubusercontent.com/1036970/187586334-3f7bf3bc-e2c2-4f12-92e1-53f2ee71a00b.mov


## The Solution

To avoid keeping the logic in both places and getting some side-effects, we will keep the logic in the live_view only.

### live/temperature_input.ex

```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})

  {:noreply, socket}
end
```

### live/calculator.ex

```elixir
def handle_info({:convert_temp, celsius}, socket) do
  fahrenheit = String.to_integer(celsius) * 9 / 5 + 32

  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```

## Wrapping up

We should keep the logic where the data is defined as a single source of the truth, and not always be the `live_view`. Keep that in mind.

Thank you all, and I hope you enjoy and have fun. So stay tuned for what is coming next.
