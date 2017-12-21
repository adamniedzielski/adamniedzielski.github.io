---
layout: post
title: "Case insensitive column in Phoenix"
date: 2017-12-21 18:17:21 +0100
comments: true
categories:
- elixir
- phoenix
- ecto
- postgres
---

I wanted to have a case insensitive database column to store email address in the
[open source application I'm working on](https://github.com/adamniedzielski/multipster).
I'm using PostgreSQL and there's a nice extension that does exactly that -
citext - so I gave it a try. I spent some time figuring things out so now I'm
sharing the knowledge in this blog post.

<!-- more -->

The instructions were tested in Elixir 1.5.3 / Phoenix 1.3.0 application.

We start by enabling `citext` extension. Run:

```
mix ecto.gen.migration EnableCitextExtension
```

and populate the migration with:

```elixir
def change do
  execute "CREATE EXTENSION citext", "DROP EXTENSION citext"
end
```

I really like this way of writing custom, yet reversible, SQL!

We apply the migration with:

```
mix ecto.migrate
```

Now we can create our new `User` model with case insensitive email:

```
mix phx.gen.schema User users email:string
```

Please note that here we still specify `string` - the generator will not allow
us to use `citext`.

Let's open the migration file and change the content to:

```elixir
def change do
  create table(:users) do
    add :email, :citext, null: false

    timestamps()
  end

  create index(:users, [:email], unique: true)
end
```

Here the specified column type is `citext`. We're also adding a `NOT NULL`
constraint and a unique index.

When you open the corresponding generated model it will look similar to:

```elixir
defmodule Multipster.User do
  use Ecto.Schema
  import Ecto.Changeset
  alias Multipster.User

  schema "users" do
    field :email, :string

    timestamps()
  end

  @doc false
  def changeset(%User{} = user, attrs) do
    account
    |> cast(attrs, [:email])
    |> validate_required([:email])
  end
end
```

That's fine for now. The important thing here is the specified type - `string`.
In the schema we describe how the database value should be transformed to a
value in the Elixir land. Ecto doesn't know about `citext` type so we have to
keep `string` here.

Let's fire up IEx and see if our code works:

```
iex -S mix
```

```elixir
Multipster.Repo.insert!(%Multipster.User{email: "test@example.com"})

Multipster.Repo.get_by(Multipster.User, email: "test@example.com")
Multipster.Repo.get_by(Multipster.User, email: "TEST@example.com")
```

The last query also works - our search by email is case insensitive! How about
duplicate email addresses and our unique index?

```elixir
Multipster.Repo.insert!(%Multipster.User{email: "TEST@example.com"})
```

The above command fails - our unique index is also case insensitive.

However, there was one more small thing that annoyed me. Take a look:

```elixir
Multipster.Repo.insert!(%Multipster.User{email: "ANOTHER@example.com"})
user = Multipster.Repo.get_by(Multipster.User, email: "another@example.com")
user.email # => "ANOTHER@example.com"
```

The returned value is exactly as provided by the user when creating the account.
It is not converted to lower case. What if we want to somehow change that
behaviour? Fortunately Ecto has a very convenient way of defining custom types:

```elixir
defmodule EmailType do
  @behaviour Ecto.Type
  def type, do: :string

  def load(data) do
    {:ok, String.downcase(data)}
  end

  def cast(data) do
    {:ok, data}
  end

  def dump(data) do
    {:ok, data}
  end
end
```

Now we can use it:

```elixir
schema "users" do
  field :email, EmailType

  timestamps()
end
```

You can read more about implementing custom types in
[the documentation](https://hexdocs.pm/ecto/Ecto.Type.html).
It's fn :)
