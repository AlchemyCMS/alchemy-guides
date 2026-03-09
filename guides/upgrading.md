---
prev:
  text: Configuration
  link: configuration
next: false
---

# Updating AlchemyCMS

This guide describes how to update AlchemyCMS to a new version.

Updating Alchemy is mostly three steps.

1. [Update the gem](#update-the-gem)
2. [Run the upgrade task](#run-the-upgrade-task)
3. [Follow ups](#follow-ups)

If you are upgrading to a new major version, read the [Major version upgrades](#major-version-upgrades) section first.

## Update the gem

If you use Alchemy from a git branch, point it to the next stable release.

~~~diff
...
-gem 'alchemy_cms', github: 'AlchemyCMS/alchemy_cms', branch: '5.2-stable'
+gem 'alchemy_cms', github: 'AlchemyCMS/alchemy_cms', branch: '5.3-stable'
...
~~~

If you use Alchemy from rubygems, point it to the next stable release.

~~~diff
...
-gem 'alchemy_cms', '~> 5.2'
+gem 'alchemy_cms', '~> 5.3'
...
~~~

If you are using `alchemy-devise` for authentication, then update the gem as well. The major and minor versions of the `alchemy_cms` and the `alchemy-devise` gems match.

~~~diff
...
-gem 'alchemy_cms', '~> 5.2'
+gem 'alchemy_cms', '~> 5.3'
-gem 'alchemy-devise', '~> 5.2'
+gem 'alchemy-devise', '~> 5.3'
...
~~~

Now update via bundler

~~~bash
bundle update alchemy_cms alchemy-devise
~~~

If your project uses other Alchemy-related gems such as `alchemy-dragonfly-s3`, `alchemy-pg_search`, `alchemy_i18n` or `alchemy-sentry`, make sure to update those as well.

## Run the upgrade task

Now you can run the upgrade task. While upgrading, you will get information about the process on your screen.

~~~bash
bin/rails alchemy:upgrade
~~~

and follow the on screen instructions.

### Update the default config

If new configuration options have been introduced, have a look at the `git diff` after running the upgrade task. Update your configuration file to keep your individual settings and add or remove keys as necessary.

If you have also `alchemy-devise` installed you need to run the alchemy devise installer again.

~~~bash
bin/rails g alchemy:devise:install
~~~

The installer will overwrite your `config/initializers/devise.rb`. Review the diff carefully afterwards and restore any project-specific settings.

### Run single upgrade tasks

Upgrade tasks can also be run on their own. For example to run only the `alchemy:upgrade:config` task:

~~~bash
bin/rails alchemy:upgrade:config
~~~

A list of all upgrade tasks can be listed with

~~~bash
bin/rails -T alchemy:upgrade
~~~

## Follow ups

Most of the time the upgrader does all the work for you. But sometimes the upgrade needs some manual work. This will be noted as TODOs at the end of the upgrade task.

Please follow them carefully.

## Major version upgrades

If you are upgrading to a new major version, make sure to fix all deprecation warnings first and run all upgrades in the current version before upgrading to the next major version.

Major versions remove deprecated features and old upgrade tasks, so unaddressed deprecations may become breaking changes after the upgrade.

To find out about breaking changes, have a look at the [release notes](https://github.com/AlchemyCMS/alchemy_cms/releases).

Otherwise, major version upgrades follow the same upgrade process as minor versions.

## Finally

Please always verify the upgrade by looking through the `git diff` and by running your test suite for customization you made. Also the [CHANGELOG](https://github.com/AlchemyCMS/alchemy_cms/blob/main/CHANGELOG.md) is a good place to look for changes.
