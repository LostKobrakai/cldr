# Changelog for Cldr v2.0.0-rc.0

This is the changelog for Cldr v2.0.0-rc.0 released on November 10th, 2018.  For older changelogs please consult the release tag on [GitHub](https://github.com/kipcole9/cldr/tags)

### Purpose

Version 2.0 of Cldr is focused on re-architecting the module structure to more closely follow the model set by Phoenix, Gettext and others that also rely on generating a public API at compile time. In Cldr version 1.x, the compile functions were all hosted within the `ex_cldr` package itself which has created several challenges:

* Only one configuration was possible per installation
* Dependency compilation order couldn't be determined which meant that when Gettext was configured a second, forced, compilation phase was required whenever the configuration changed
* Code in the ex_cldr _build directory would be modified when the configuration changed which is not a good idea.

### New structure and configuration

In line with the recommended strategy for configurable library applications, `Cldr` now requires a user module be defined that hosts the configuration and public API.  This is similar to the strategy used by `Gettext`, `Phoenix` and others.  These modules, called backend modules, are defined like this:

    defmodule MyApp.Cldr do
      use Cldr, locales: ["en", "zh"]
    end

For further information on configuration, consult the [readme](/ex_cldr/readme).

### Migrating from Cldr 1.x to Cldr version 2.x

Although the api structure is the same in both releases, the move to a backend module hosting configuration and the public API requires changes in applications using Cldr version 1.x.  The steps to migrate are:

1. Change the dependency in `mix.exs` to `{:ex_cldr, "~> 2.0"}`
2. Define a backend module to host the configuration and public API.  It is recommended that the module be named `MyApp.Cldr` since this will ease migration through module aliasing.
3. Change calls to `Cldr.function_name` to `MyApp.Cldr.function_name`.  The easiest way to do this is to alias the backend module.  For example:

```
defmodule MyApp.SomeModule do
# alias the backend module so that calls to Cldr functions still works
  alias MyApp.Cldr

  def some_function do
    IO.puts Cldr.known_locale_names
  end
end
```
### Breaking Changes

* Configuration has changed to focus on the backend module, then otp app, then global config.  All applications are required to define a backend module.
* The Public API moves to a configured backend module. Functions previous called on `Cldr` should be called on `MyApp.Cldr`.
* The `~L` sigil has been removed.  The public api functions support either a locale name (like "en") or a language tag.
* `Cldr.Plug.AcceptLanguage` and `Cldr.Plug.SetLocale` need to have a config key :cldr_backend to specify the `Cldr` backend to be used.
* The `Mix` compiler `:cldr` is obsolete.  It still exists so configuration doesn't break but its no a `:noop`.  It should be removed from your configuration.
* `Config.get_locale/1` now takes a `config` or `backend` parameter and has become `Cldr.Config.get_locale/2`.
* `Cldr.set_currenct_locale` has changed to be `Cldr.put_current_locale` to be more consistent with idiomatic Elixir conventions.
