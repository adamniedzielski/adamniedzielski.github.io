---
layout: post
title: "ETag tracking and Elixir"
date: 2017-03-28 19:33:51 +0100
comments: true
categories:
- http
- etag
- caching
- tracking
- elixir
- phoenix
---

I am really fascinated by the idea of **abusing ETag** (caching mechanism built into HTTP protocol) for tracking users. It is not that I want to do in a real production application; I just appreciate how simple and clever the "trick" is. Also, not so many people seem to be aware of it.

<!-- more -->

I do not think that I can do better at explaining how **ETag tracking** works in general than ["Cookieless cookies" article](https://lucb1e.com/rp/cookielesscookies/). Please take a look at it first and then (hopefully) come back here.

Instead of just admiring how clever the method is in theory, I wanted to play a little bit with the idea and create a small application to reproduce it. I decided to combine it with my desire to write some **Elixir** code. Hence, in this blog post I will show how to implement ETag tracking in a Phoenix application.

After running `mix phoenix.new` we can jump straight to the code. Our model is very simple:

```elixir
defmodule ETagTracker.Visitor do
  use ETagTracker.Web, :model

  schema "visitors" do
    field :token, :string
    field :visits, :integer

    timestamps()
  end

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:token, :visits])
    |> validate_required([:token, :visits])
  end
end
```

We have to store a token associated with every visitor. And of course we can store some additional information, in this case the number of previous visits.

The template that we want to render as the main (and only) page is:

```html
<p>
  Number of previous visits: <%= @visits %>
</p>
```

And the interesting code is in the controller:

```elixir
def index(conn, _params) do
  visitor = find_or_initialize_visitor(conn)

  Repo.insert_or_update(
    Visitor.changeset(visitor, %{visits: visitor.visits + 1})
  )

  conn
  |> put_resp_header("etag", visitor.token)
  |> assign(:visits, visitor.visits)
  |> render("index.html")
end
```

We use `put_resp_header` to set `ETag` header to the value that we saved in the database.

`find_or_initialize_visitor` function:

```elixir
defp find_or_initialize_visitor(conn) do
  visitor = case get_req_header(conn, "if-none-match") do
    [value] ->
      Visitor |> Repo.get_by(token: value)
    [] ->
      nil
  end

  visitor || %Visitor{visits: 0, token: generate_token()}
end
```

If the request has `If-None-Match` header set we try to find a matching record in our database. If the header is not set we treat the request as coming from a new visitor and initialize a new record.

`generate_token` simply returns a long, random string:

```
defp generate_token do
  :crypto.strong_rand_bytes(32) |> Base.encode16(case: :lower)
end
```

This is all we need! You can see the application running [here](https://etag-tracker.herokuapp.com/) and the full source code is [here](https://github.com/adamniedzielski/etag_tracker).

