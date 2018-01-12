---
layout: post
title: "Mock current time in Elixir"
date: 2018-01-12 00:53:15 +0100
comments: true
categories:
- elixir
- phoenix
- functional programming
- testing
- mocking
---

Coming from the Ruby/Rails world I was searching for a way to mock the current
time in my Elixir test suite, something like **Timecop for Elixir**. I didn't
find anything that suited my needs so I decided to give it a go by myself.

<!-- more -->

I had a module that contained following code:

```elixir
defmodule Multipster.Token do
  def encode(user) do
    # [...]
  end

  def decode(token) do
    # [...]
  end

  defp verify_expiration(expiration) do
    expiration > current_timestamp()
  end

  defp get_expiration do
    current_timestamp() + 30 * 60
  end

  defp current_timestamp do
    :os.system_time(:second)
  end
end
```

As you can see it depends on the current time. What if we want to test that
an expired token is invalid? We have to encode the token, **travel in time**, and
try to decode it.

```elixir
defmodule Multipster.TokenTest do
  use ExUnit.Case, async: true
  alias Multipster.Token

  test "return error when token expired" do
    token = Token.encode(%Multipster.User{id: 4})

    # travel in time

    {:error, _} = Token.decode(token)
  end
end
```

How can we travel in time? We have to manipulate the current time.

My main source of knowledge about mocking in Elixir was
[Mocks and explicit contracts](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/)
by José Valim himself. The article suggests to avoid mocking (as a verb)
and instead write **mocks (as a noun)**. I was curious of this approach so I
decided to give it a try.

I changed the implementation of `current_timestamp/0` to:

```elixir
defp current_timestamp do
  Multipster.CurrentTime.get_timestamp()
end
```

`Multipster.CurrentTime` is just an interface that delegates the function
call to a selected implementation:

```elixir
defmodule Multipster.CurrentTime do
  @adapter :multipster
           |> Application.get_env(__MODULE__)
           |> Keyword.fetch!(:adapter)

  def get_timestamp do
    @adapter.get_timestamp()
  end
end
```

The default implementation contains code that we already saw:

```elixir
defmodule Multipster.CurrentTime.Real do
  def get_timestamp do
    :os.system_time(:second)
  end
end
```

We configure it in `config/confix.exs`:

```elixir
config :multipster, Multipster.CurrentTime,
  adapter: Multipster.CurrentTime.Real
```

The only environment where we want to use a different adapter is `test`:

```elixir
config :multipster, Multipster.CurrentTime,
  adapter: Multipster.CurrentTime.Mock
```

In order to implement our mock we have to write code that can keep state and allow
this state to be manipulated from tests. I decided to use `Agent` for that,
because it seemed to be a recommended pattern.

The state that we want to keep in the agent is `is_frozen` and `frozen_value`.
Initially we start with an unfrozen time and we allow to change this state
from tests by providing `freeze/0`, `freeze/1`, and `unfreeze/0` functions.
Of course we also have to implement the required interface - `get_timestamp/0`.
Here we go:

```elixir
defmodule Multipster.CurrentTime.Mock do
  use Agent

  def start_link do
    Agent.start_link(fn ->
      %{is_frozen: false, frozen_value: nil}
    end, name: __MODULE__)
  end

  def get_timestamp do
    state = Agent.get(__MODULE__, fn state -> state end)

    if state[:is_frozen] do
      state[:frozen_value]
    else
      :os.system_time(:second)
    end
  end

  def freeze do
    freeze(:os.system_time(:second))
  end

  def freeze(timestamp) do
    Agent.update(__MODULE__, fn _state ->
      %{is_frozen: true, frozen_value: timestamp}
    end)
  end

  def unfreeze do
    Agent.update(__MODULE__, fn _state ->
      %{is_frozen: false, frozen_value: nil}
    end)
  end
end
```

I was excited when writing this code as it was my first experience with message
passing in Elixir!

We need one more thing to get the agent to work - we have to start it. We will
do it in `test/test_helper.exs`:

```elixir
{:ok, _} = Multipster.CurrentTime.Mock.start_link()
```

And now we can travel in time:

```elixir
defmodule Multipster.TokenTest do
  use ExUnit.Case, async: true
  alias Multipster.Token

  test "return error when token expired" do
    token = Token.encode(%Multipster.User{id: 4})

    Multipster.CurrentTime.Mock.freeze(:os.system_time(:second) + 31 * 60)

    {:error, _} = Token.decode(token)
  end
end
```

It works, yay!

Are we done yet? Not really, there are a few things to improve here.

They say that there's no global state in Elixir, but here we have a similar
problem - an agent holding state that isn't magically reset between tests.
All tests executed after the above one will use the frozen time. We don't want that
so we have to explicitly unfreeze the time.

Initially I went with:

```elixir
test "return error when token expired" do
  token = Token.encode(%Multipster.User{id: 4})

  Multipster.CurrentTime.Mock.freeze(:os.system_time(:second) + 31 * 60)

  {:error, _} = Token.decode(token)

  Multipster.CurrentTime.Mock.unfreeze()
end
```

The problem is that if `{:error, _} = Token.decode(token)` doesn't match we will
never execute `Multipster.CurrentTime.Mock.unfreeze()`.

We need something like an `after` hook from RSpec. `ExUnit.Callbacks.on_exit/2`
should do the job:

```elixir
test "return error when token expired" do
  token = Token.encode(%Multipster.User{id: 4})

  Multipster.CurrentTime.Mock.freeze(:os.system_time(:second) + 31 * 60)
  on_exit &Multipster.CurrentTime.Mock.unfreeze/0

  {:error, _} = Token.decode(token)
end
```

This also helps us maintain a logical order in the test - assertion at the end.

As we have only one agent in the whole application, **concurrently** running multiple tests that mess with the time is **not safe**. Two tests may attempt to
change the current time at the very same moment. This can lead to random,
hard-to-reproduce test failures. We have to specify:

```elixir
use ExUnit.Case, async: false
```

I'd really appreciate input on how to improve my code so that tests can be
safely run concurrently.

That said, this method allowed me to mock the time in my Elixir application, and I
had a lot of fn while writing the code. I hope that you benefited from the blog
post as well.
