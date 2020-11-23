---
layout: post
title: "Phoenix Views Outside of Phoenix"
---

# Phoenix Views Outside of Phoenix

Author: ...Paul

Email is a funny thing.  Its generation looks and feels a lot like a web UI,
in that you end up rendering templates, but it originates from the business
logic layer, not the UI.  So its generation doesn't really belong in your
Phoenix application.  But you still probably want to take advantage of all
that great Phoenix View and maybe even HTML functionality...

None of the documentation seems to talk much about that.  The 
[Bamboo.Phoenix](https://hexdocs.pm/bamboo/Bamboo.Phoenix.html)
package assumes you're installing this right into a "normal" non-umbrella
Phoenix package.

If you've decided to use the `--umbrella` option, though, your business
logic is in a different project in the umbrella, and your web application
depends on that.  If you try to have the business logic depend back on the
Phoenix part, you get a cyclical dependency.

Although the Phoenix package is big -- remember, this is Elixir.  It's all
just functions.  So when you get down to the core of it, it's just about
figuring out what modules and functions you need, and then using them.

### Using Phoenix Only For Its Tooling

It feels wrong, dirty, and heavyweight (and if anyone has a better solution,
please email me!) but you can still pull in Phoenix and not go through all the
setup you would for a full web application.

Just go ahead and add the packages to your `mix.exs` file:

```
     {:bamboo_smtp, "~> 3.0.0"},
      # Phoenix only included to use views in Bamboo (see Mailer)
      {:phoenix, "~> 1.5.5"},
      {:phoenix_html, "~> 2.11"},
```

Just adding the libraries to your package doesn't "do" anything magical;
your app won't behave any differently because you don't have any configuration
for an endpoint in place.  So it's safe.  It's a big package, so it doesn't
"feel" safe, but it really is.

### Replacing The Boilerplate

If you already have a Phoenix app up, you probably noticed your View modules
all start with just `use MyAppWeb, :view`.  This magic is just a prefab set
of macros back in `my_app_web.ex`, automagically generated when you created
the application, that look something like this:

```
  def view do
    quote do
      use Phoenix.View,
        root: "lib/bento_web/templates",
        namespace: BentoWeb

      # Import convenience functions from controllers
      import Phoenix.Controller,
        only: [get_flash: 1, get_flash: 2, view_module: 1, view_template: 1]

      # Include shared imports and aliases for views
      unquote(view_helpers())
    end
  end

  defp view_helpers do
    quote do
      # Use all HTML functionality (forms, tags, etc)
      use Phoenix.HTML

      # Import basic rendering functionality (render, render_layout, etc)
      import Phoenix.View

      import BentoWeb.ErrorHelpers
      import BentoWeb.Gettext
      alias BentoWeb.Router.Helpers, as: Routes
    end
  end
```

We don't need all that junk in our Mailer -- we're not going to use flash and
stuff, so we can whittle that down and just put the stuff we want/need into
our View module:

```
defmodule Bento.EmailView do
  use Phoenix.View, root: "lib/bento/templates", namespace: Bento
  use Phoenix.HTML
  import Phoenix.View
end
```

That's it.  Add the `lib/my_app/templates` directory that you've defined there
(which means it doesn't have to be in `templates` at all, but could be in
`email_templates` or whatever you feel like, nested as far down as you want.

### Follow the Rest of the Directions

After that, all the directions work just fine, you define your Mailer module
to use the Phoenix module and tell it what view to use:

```
defmodule Bento.Mailer do
  use Bamboo.Mailer, otp_app: :bento
  use Bamboo.Phoenix, view: Bento.EmailView
```

Now you're set to go, and everything else in the documentation works just as
it says.

Of course, YMMV.  You may need/want to pull in the Gettext module for i18n,
for example.

### Conclusion

Don't be afraid to dig into the guts of an existing package and figure out
how it works.  It might be a lot more straightforward than you think.






