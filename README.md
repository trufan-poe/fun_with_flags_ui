# FunWithFlags.UI

A Web dashboard for the [FunWithFlags](https://github.com/tompave/fun_with_flags) Elixir package.

This package is still a work in progress.


## How to run

`FunWithFlags.UI` is just a plug and it can be run in a number of ways.

### Standalone

wip

### Mounted in Phoenix

The main plug can be mounted inside the Phoenix router with [`Phoenix.Router.forward/4`](https://hexdocs.pm/phoenix/Phoenix.Router.html#forward/4).

```elixir
defmodule MyPhoenixApp.Web.Router do
  use MyPhoenixApp.Web, :router

  pipeline :mounted_apps do
    plug :accepts, ["html"]
    plug :put_secure_browser_headers
  end

  scope path: "/feature-flags" do
    pipe_through :mounted_apps
    forward "/", FunWithFlags.UI.Router, namespace: "feature-flags"
  end
end
```

### Mounted in another Plug application

Since it's just a plug, it can also be mounted into any other Plug application using [`Plug.Router.forward/2`](https://hexdocs.pm/plug/Plug.Router.html#forward/2).

```elixir
defmodule Another.App do
  use Plug.Router
  forward "/feature-flags", to: FunWithFlags.UI.Router, init_opts: [namespace: "feature-flags"]
end
```

### Security

For obvious reasons, you don't want to make this web control panel publicly accessible.

The library itself doesn't provide any auth functionality because, as a plug, it is easier to wrap it into the specific authentication of the host application.

The easiest thing to do is to protect it with HTTP Basic Auth, provided by the [`basic_auth`](https://hex.pm/packages/basic_auth) plug.

For example, in Phoenix:

```elixir
defmodule MyPhoenixApp.Web.Router do
  use MyPhoenixApp.Web, :router

  def my_basic_auth(conn, username, password) do
    if username == "foo" && password == "bar" do
      conn
    else
      Plug.Conn.halt(conn)
    end
  end

  pipeline :mounted_and_protected_apps do
    plug :accepts, ["html"]
    plug :put_secure_browser_headers
    plug BasicAuth, callback: &__MODULE__.my_basic_auth/3
  end

  scope path: "/feature-flags" do
    pipe_through :mounted_and_protected_apps
    forward "/", FunWithFlags.UI.Router, namespace: "feature-flags"
  end
end
```

## Caveats

While the base `fun_with_flags` library is quite relaxed in terms of valid flag names, group names and actor identifers, this web dashboard extension applies some more restrictive rules.
The reason is that all `fun_with_flags` cares about is that some flag and group names can be represented as an Elixir Atom, while actor IDs are just strings. Since you can use that API in your code, the library will only check that the parameters have the right type.

Things change on the web, however. Think about the binary `"Ook? Ook!"`. In code, it can be accepted as a valid flag name:

```elixir
{:ok, true} = FunWithFlags.enable(:"Ook? Ook!", for_group: :"weird, huh?")
```

On the web, however, the question mark makes working with URLs a bit tricky: in `http://localhost:8080/flags/Ook?%20Ook!`, the flag name will be `Ook` and the rest will be a query string.

For this reason this library enforces some stricter rules when creating flags and groups. Blank values are not allowed, `?` neither, and flag names must match `/^w+$/`.


## Installation

The package can be installed by adding `fun_with_flags_ui` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:fun_with_flags_ui, "~> 0.0.1"}]
end
```
