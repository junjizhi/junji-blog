---
layout: post
title: "The case for using Elixir try/rescue"
categories: [All, Technical]
tags: [elixir, phoenix, view, try/rescue, error handling, exception, error]
fullview: false
excerpt: Recently I ran into a case where using Elixir try/rescue makes a lot of sense.
comments: true
---

## Introduction

According to the [Elixir doc](https://elixir-lang.org/getting-started/try-catch-and-rescue.html), it is rare to need to use try/rescue. That's partly because the elixir/erlang philosophy of letting it crash.

Recently I ran into a case where using try/rescue makes a lot of sense.

The scenario goes like this: We use Elixir/Phoenix to render the front end from the server side. For a page, we
have view helpers, something like:

```elixir
def render_some_view(params) do
  # Convert params into a %ApiParams{} struct
  transformed_params = transformed_params(params)
  # Fetch data from api
  api_data = call_api!(transformed_params)
  render('partial.html', api_data)
end
```

## The crash

What happened was that, the other team updated the struct of `ApiParams`. It subtly broke the `call_api!/1` call because it didn't have a fallback function to

```elixir
# :field1 and :field2 would not show up together any more
def call_api!(%{ApiParams{field1: v1, field2: v2}) do
  # ...
end
```
As a result, the function returns a `FunctionClauseError` and crashes the view helper.

If this was a back-end process, it would fail and restart the server process. If error reporting is setup properly, dev teams would get notified.

What makes things worse is that, this view helper is used in a landing page to render a section. When this failed, it crashed the entire page, even if all other sections are fine.

Now the business is asking: Can we put something in place to prevent the landing page from crashing?

One section failed? That's fine. Just hide it.

This is where the `try/rescue`	setup comes in:

```elixir
  # Rename to bang function
  def render_some_view!(params) do
    # ...
  end

  def safe_render_some_view(params) do
    try do
      render_some_view!(params)
    rescue
      e in RuntimeError ->
        # Notify error reporting service, e.g., Bugsnag
        notify_reporting_service(e)
        render_fallback_view()
    end
  end
```

This way, we 1) get notified about the failure, 2) tolerate bad views changes without crash the high-stakes pages.

## It's an anti-pattern?
In some sense, yes.

The try/rescue can seem a bit over-defensive, which is a smell, or indicator of dysfunctional teams.

You can argue that, if we had better test coverage, this kind of errors should not happen.

I argue that, when team is large, and the code is evolving fast, putting things in place is necessary, especially for high stake pages. It buys us time and reduces the urgency when things break.

## Alternatives

You can use return tuples, e.g., `{:ok|:error, _}` to inform the callsites about the return status. In our case, we have the bang API call (which isn't always controlled by our team). Also, changing to return tuples required touching all render functions which requires some work.

Although it seems heavy-handed, it is a good defence to frequent code changes to the view by multiple teams.

