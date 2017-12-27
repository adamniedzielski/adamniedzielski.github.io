---
layout: post
title: "Elixir \"with\" syntax and guard clauses"
date: 2017-12-27 16:32:18 +0100
comments: true
categories:
- elixir
- phoenix
- functional programming
- refactoring
---

In this blog post we will take a look at a simple example of refactoring Elixir
code using `with` syntax (called a "special form" in the documentation), and guard
clauses.

<!-- more -->

The code that I started with:

```elixir
def call(conn, _options) do
  user_id = get_session(conn, :user_id)

  if user_id do
    user = Repo.get(User, user_id)
  end

  if user do
    assign(conn, :current_user, user)
  else
    conn
    |> Controller.put_flash(:error, "You have to sign in to access this page.")
    |> Controller.redirect(to: "/sign_in_links/new")
    |> halt
  end
end
```

The algorithm can be described as:

1. look up `user_id` in the session
2. if `user_id` is set find the user in the database
3. if the given user exists assign it as `current_user`
4. if `user_id` is not set or the user doesn't exist in the database display a
flash message and make a redirect

The code looks quite simple, but in fact it's not very **explicit**. While reading it
we have to remember that when `user_id` is `nil` then also `user` will be `nil`,
because the variable is not being assigned at all. That's the implicit part and
I don't like it.

The Elixir compiler even emits a warning here:

```
warning: the variable "user" is unsafe as it has been set inside one of: case, cond, receive, if, and, or, &&, ||. Please explicitly return the variable value instead. For example:

    case integer do
      1 -> atom = :one
      2 -> atom = :two
    end

should be written as

    atom =
      case integer do
        1 -> :one
        2 -> :two
      end
```

We can follow these instructions directly, but we can also take a look at
`with` special form:

```
def call(conn, _options) do
  with {:ok, user_id} <- get_session(conn, :user_id),
       {:ok, user} <- Repo.get(User, user_id)
  do
    assign(conn, :current_user, user)
  else
    {:error, _} ->
      conn
      |> Controller.put_flash(:error, "You have to sign in to access this page.")
      |> Controller.redirect(to: "/sign_in_links/new")
      |> halt
  end
end
```

That's the most common example of the `with` syntax shown in the blog posts that
I found. When you have functions that return `{:ok, _}` or `{:error, _}` it's
really straightforward to apply.

However, my problem is slightly different. `Plug.Conn.get_session/2` returns
a value or `nil`, and `Ecto.Repo.get/3` returns a database record or `nil`.

Now I was a bit puzzled here - how can I adjust the above example to solve **my**
problem?

Fortunately I found the answer in the
[official documentation for `with`](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#with/1).
You can combine `with` with [guards](https://hexdocs.pm/elixir/guards.html).

My first attempt produced:

```elixir
def call(conn, _options) do
  with user_id when !is_nil(user_id) <- get_session(conn, :user_id),
       user when !is_nil(user) <- Repo.get(User, user_id)
  do
    assign(conn, :current_user, user)
  else
    _ ->
      conn
      |> Controller.put_flash(:error, "You have to sign in to access this page.")
      |> Controller.redirect(to: "/sign_in_links/new")
      |> halt
  end
end
```

This resulted in a cryptic error from the compiler:

```
** (CompileError) lib/multipster_web/sign_in/plug.ex:29: invalid expression in guard, case is not allowed in guards
    (elixir) expanding macro: Kernel.!/1
```

When I re-read the documentation for guards I realized that I have to replace
`!` with `not`:

```elixir
def call(conn, _options) do
  with user_id when not is_nil(user_id) <- get_session(conn, :user_id),
       user when not is_nil(user) <- Repo.get(User, user_id)
  do
    assign(conn, :current_user, user)
  else
    _ ->
      conn
      |> Controller.put_flash(:error, "You have to sign in to access this page.")
      |> Controller.redirect(to: "/sign_in_links/new")
      |> halt
  end
end
```

This worked perfectly, hooray! What I really like about this approach is that
the happy path and the error handling path are separated.
