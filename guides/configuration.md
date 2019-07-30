---
prev: /render_images.html
next: /upgrading.html
---

# Configuration of AlchemyCMS

Alchemy configuration is done in the `config/alchemy/config.yml` file.

Use

~~~ bash
bin/rake alchemy:install
~~~

or

~~~ bash
bin/rails g alchemy:install
~~~

to generate it.

## The configuration file

Please [have a look into the source code](https://github.com/AlchemyCMS/alchemy_cms/blob/master/config/alchemy/config.yml). The file is pretty well documentated and much more up to date then this guide.

## Environment related configurations

Beside using that one configuration file you also can seperate certain config settings for specific environments.

For example: You want to enforce SSL in production, while in your development environment you don't.

Its easy to achieve by setting up a second configuration file for development which contains nothing but the ssl enforcing setting.

Your apps default config:

~~~
your_app/config/alchemy/config.yml
~~~

Environment specific config:

~~~
your_app/config/alchemy/development.config.yml
~~~

::: tip NOTE
When running your application in development mode both config files get merged, while the environment specific config takes precedence over the default config.
:::

## TinyMCE
Configuring TinyMCE Editor is pretty simple, all you can do is to create file:

~~~
config/initializers/tinymce.rb
~~~

copy the tinymce list plugin folder to 
~~~
vendor/assets/javascripts/tinymce/plugins
~~~

and add the following content to config/initializers/tinymce.rb

~~~
Alchemy::Tinymce.init = {
    plugins: Alchemy::Tinymce.plugins + ['lists'],
    toolbar: [
        'bold italic underline | strikethrough subscript superscript | numlist bullist indent outdent | removeformat | alignleft aligncenter alignright | fullscreen',
        'pastetext charmap hr | undo redo | alchemy_link unlink anchor | code'
    ]
}
~~~

the example above, add the lists plugin to TinyMCE and configure toolbar. The toolbar configuration can be personalized with preferred functions supported by the editor.
Another example of configuration could be:

~~~
Alchemy::Tinymce.plugins += ['textcolor']
Alchemy::Tinymce.init = {
  toolbar: [
    'bold italic underline | strikethrough subscript superscript | numlist bullist indent outdent | removeformat | fullscreen',
    'pastetext | charmap code | undo redo | alchemy_link unlink | help | forecolor backcolor'
  ],
}
~~~





