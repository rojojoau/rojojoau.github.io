---
layout: post
title:  "Elixir Ecto - could not find migration runner process"
date:   2016-03-08 22:33:35 +1100
---

Over the last few weeks I've had an issue with a project using [the Phoenix
Framework](http://phoenixframework.org), in which I was getting an odd error
message whenever I ran `mix ecto.migrate`.

```elixir
(RuntimeError) could not find migration runner process for #PID<0.46.0>
(ecto) lib/ecto/migration/runner.ex:70: Ecto.Migration.Runner.prefix/0
(ecto) lib/ecto/migration.ex:665: Ecto.Migration.__prefix__/1
(ecto) lib/ecto/migration.ex:270: Ecto.Migration.create/1
(stdlib) erl_eval.erl:669: :erl_eval.do_apply/6
(elixir) lib/code.ex:321: Code.load_file/2
(ecto) lib/ecto/migrator.ex:221: anonymous fn/4 in Ecto.Migrator.migrate/4
(elixir) lib/enum.ex:1088: Enum."-map/2-lists^map/1-0-"/2
(ecto) lib/mix/tasks/ecto.migrate.ex:63: anonymous fn/4 in
Mix.Tasks.Ecto.Migrate.run/2
(elixir) lib/enum.ex:604: Enum."-each/2-lists^foreach/1-0-"/2
(elixir) lib/enum.ex:604: Enum.each/2
(mix) lib/mix/cli.ex:58: Mix.CLI.run_task/2
```

I had recently rebuilt my machine and moved to [Elixir](http://elixir-lang.org) v1.2, and suspected that this was the issue, although that assumption was incorrect.

It turns out that the root cause was a poorly formatted migration, where I
had some of the migration code outside of the `change/0` function.

#### Replicating the issue

Once I understood the problem it was simple to create a migration that
replicated the issue, and also easy to fix.

This migration below replicates the issue:

```elixir
defmodule Example.Repo.Migrations.Failure do
  use Ecto.Migration

  def change do
    create table(:testing) do
      add :name, :string
    end
  end

  create index(:testing, [:name])
end
```

Coincidentally, another use had the same problem and [raised an issue about
it](https://github.com/elixir-lang/ecto/issues/1298) a few hours earlier.

#### Potential improvements

I think that we could improve the error message that we're receiving when this
issue occurs. That would help anyone in the future who encounters this issue to
resolve it quickly.

Also, it would be nice to have an option for verbose output when running
[Ecto](https://github.com/elixir-lang/ecto) migrations, so that we can see where
the migration get's to before it fails. Much of my debugging involved adding log
messages to Ecto while performing the migration.
