I have worked with Phoenix [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) and [Surface-UI](https://surface-ui.org/) for about a year; I would like to share some of the things I learned the hard way.

## Intro
[Marlus Saraiva](https://github.com/msaraiva) introduced me to a concept from from the React community called `single source of truth`.
​
> There should be a single `source of truth` for any data that changes in a React application. Usually, the state is first added to the component that needs it for rendering. Then, if other components also need it, you can lift it up to their closest common ancestor.
[full reference - Lifting State Up](https://reactjs.org/docs/lifting-state-up.html#:~:text=There%20should%20be%20a%20single,to%20their%20closest%20common%20ancestor.)
​

Once you understand this concept, you will see it over and over again as a pattern.
​
## Setup
Imagine an application called **Dummy** that has a **Calculator** for **Temperature**, where you input values in **Celsius** and show them in **Fahrenheit**.
​
The application will be like this:
​
![Celsius Input Form](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wnuafnzhniajf8k5zxyh.png)
​
![Convert Celsius in Fahrenheit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/px3zyty9ilcv6jp0fh2t.png)
​
First thing first, we should add a new path to the **Router** for our live **Calculator**.
​
**router.ex**

```elixir
scope "/", Dummy do
  pipe_through(:browser)
​
  live("/", Calculator)
end
```
​
Now the route is created, we can work on the [live_view](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#content). We can ignore the `params` and `sessions` in [mount/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:mount/3), and just assign the default Fahrenheit value through the socket.
​
In the [render/1](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:render/1) template, we will call **TemperatureInput** `live_component` and pass the **id** and the **Fahrenheit** value. Finally we will add [handle_info/2](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_info/2).
​
**live/calculator.ex**

```elixir
defmodule DummyWeb.Calculator do
  use DummyWeb, :live_view
​
  alias DummyWeb.TemperatureInput
​
  @impl true
  def mount(_params, _session, socket), do: {:ok, assign(socket, :fahrenheit, 0)}
​
  @impl true
  def render(assigns) do
    ~H"""
    <main class="hero">
      <.live_component module={TemperatureInput} id={"celsius_to_fahrenheit"} fahrenheit={@fahrenheit}/>
    </main>
    """
  end
​
  @impl true
  def handle_info({:convert_temp, celsius}, socket) do
    fahrenheit = to_fahrenheit(celsius)
​
    {:noreply, assign(socket, :fahrenheit, fahrenheit)}
  end
  
  defp to_fahrenheit(celsius) do
    String.to_integer(celsius) * 9 / 5 + 32
  end
  
  defp to_celsius(fahrenheit) do
    (String.to_integer(fahrenheit) - 32) * 5 / 9
  end
end
```
​
Now for the **live_component**.

**live/temperature_input.ex**
​
```elixir
defmodule DummyWeb.TemperatureInput do
  use DummyWeb, :live_component
​
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
​
  def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
    send(self(), {:convert_temp, celsius})
​
    fahrenheit = to_fahrenheit(celsius)
​
    {:noreply, assign(socket, :fahrenheit, fahrenheit)}
  end
  
  defp to_fahrenheit(celsius) do
    String.to_integer(celsius) * 9 / 5 + 32
  end
  
  defp to_celsius(fahrenheit) do
    (String.to_integer(fahrenheit) - 32) * 5 / 9
  end
end
```
​
Looking into the [live_component](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html#content), it only has a form with an input field where the user types a value in **Celsius** degrees and clicks the submit button to send this event to the server, which has a [handle_event/3](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_event/3), and does two things:
​
- send a message to the parent(the live_view).
- calculate the Fahrenheit temperature and assign it through the socket.

​
We are using two LiveView [phx-* bindings](https://hexdocs.pm/phoenix_live_view/bindings.html):
​
- `phx-submit` - it will send the form to the handle event called **to_fahrenheit**.
- `phx-target` - it tells who should handle the event. The default behaviour is always the **live_view**, otherwise we can set the target. Using **@myself** to set the target is our **live_component**.
​

You have most likely noticed that the **handle_info** in **live_view** and the **handle_event** in **live_component** have the same code. The only difference between them is the `send()` function to update the Fahrenheit value.
​
## The Issue
​
We should avoid the duplicity of logic and in our example the formula to convert Celsius to Fahrenheit is duplicated in both the **live_view** and the **component**.
​
To demonstrate what can go wrong, we will use `:time.sleep` to change the formula in just the **live_component**.
​
Change the logic in **live_component**.
​
**live/temperature_input.ex**
​
```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})
​
  # wrong formula
  fahrenheit = to_celsius(celsius)
​
  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```
​
**live/calculator.ex**
​
```elixir
def handle_info({:convert_temp, celsius}, socket) do
  :time.sleep(4000)
​
  fahrenheit = to_fahrenheit(celsius)
​
  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```


https://user-images.githubusercontent.com/1036970/187585732-63547a87-afb7-4ab5-8872-f3ad8d48ac36.mov

Change the logic in **live_view**.
​
**live/temperature_input.ex**
​
```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})
​
  fahrenheit = to_fahrenheit(celsius)
​
  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```
​
**live/calculator.ex**
​
```elixir
def handle_info({:convert_temp, celsius}, socket) do
  :time.sleep(4000)
​
  # wrong formula
  fahrenheit = to_celsius(celsius)
​
  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```


https://user-images.githubusercontent.com/1036970/187586334-3f7bf3bc-e2c2-4f12-92e1-53f2ee71a00b.mov

## The Solution
​
Avoid keeping the logic in more than one place. In this case we decided to keep the logic in the **live_view** only but it didn't have to be in the **live_view**. The goal is to  never duplicated the logic.
​
**live/temperature_input.ex**
​
```elixir
def handle_event("to_fahrenheit", %{"temp" => %{"celsius" => celsius}}, socket) do
  send(self(), {:convert_temp, celsius})
​
  {:noreply, socket}
end
```
​
**live/calculator.ex**
​
```elixir
def handle_info({:convert_temp, celsius}, socket) do
  fahrenheit = to_fahrenheit(celsius)
​
  {:noreply, assign(socket, :fahrenheit, fahrenheit)}
end
```
​
## Wrapping up
​
Keep the logic where the data is defined as a **single source of the truth** to avoid future headaches and difficult to fix bugs.
​
Thank you all, and I hope you enjoy and have fun. So stay tuned for what is coming next.

And a special thanks to *[Adolfo Neto](https://twitter.com/adolfont)*, *[Cristine Guadelupe](https://twitter.com/crisguade)*, *[Mike Kumm](https://twitter.com/mkumm)*, *[Willian Frantz](https://twitter.com/frantz_willian)* for review my blog post.
