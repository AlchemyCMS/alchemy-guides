---
prev: elements
next: render_images
---

# Essences

## Overview

Essences store the actual content of your site. They are stored as instances of `Alchemy::Content` on each `Alchemy::Element` instance.

Alchemy comes with a lot of predefined essences for the regular needs of a website project. Combine them like a chemestry kit into [elements](elements).

Essences are Rails models. It is pretty easy to [add your own essence class](create_essences) as well.


## Definition

Essences are defined as `contents` on an [element definition](elements.html#defining-elements).

### Global content settings

When defining contents, you need to provide a `name` and essence `type`. You can set hints and default values as well.

~~~ yaml
# config/alchemy/elements.yml
- name: article
  contents:
    - name: headline
      type: EssenceText
      hint: This is the headline
    - name: color
      type: EssenceText
      settings:
        input_type: color
    - name: copy
      type: EssenceRichtext
      default: Lorem ipsum dolor
      as_element_title: true
~~~

* **name** `String` _required_

  A lowercased **unique** (per element) name of the content. Separated words needs to be underscored.

* **type** `String` _required_

  An essence type this content is from. Alchemy has lots of built in essences for [simple text](#essencetext), [rich text](#essencerichtext), [images](#essencepicture), [booleans](#essenceboolean) and more.

* **hint** `String|Symbol|Boolean`

  A hint for the user in the admin frontend that describes what the essence is used for. The hint is translatable if you provide an I18n translation `Symbol` instead of a `String`. You may also set it to `true` to default to the I18n key `alchemy.content_hints.your-content-name`.

* **default** `String`

  The default text to prefill newly created elements. You may also use a symbol to set it to the I18n key `alchemy.default_content_texts.your-symbol-name`

* **as_element_title** `Boolean`

  For the displayed element title, the first content essence is used. Use this setting to override this behaviour and show other content as element title.

* **settings** `Hash`

  A set of options to configure the essence. Each essence has its own set of options listed below.

### Individual essence settings

Each essence type can have its own type of settings.

To configure these settings you have to pass them into its `settings` key in the `elements.yml`.

~~~ yaml {5-6}
- name: my_element
  contents:
    - name: headline
      type: EssenceText
      settings:
        linkable: true
~~~

## EssenceText

Stores plain text of 255 chars max.

Use this for a headline, or a product name. The editor is renderd as a single lined input field. The view output will be sanitized and HTML escaped.

### Settings


* **linkable** `Boolean`

  If set to `true`, the user can add a link to the text.

* **input_type** `String`

  Change the [input type](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#Form_%3Cinput%3E_types) of the form field displayed to the content editors.

### Example

~~~ yaml
- name: button
  type: EssenceText
  settings:
    linkable: true
~~~

## EssenceRichtext

Used to store paragraphs of formatted text.

The editor is rendered as a textarea with embedded Tinymce Editor.

### Settings

You can customize the Tinymce editor of a single element instance.

* **tinymce** `Hash`

### Example

~~~ yaml
- name: text
  type: EssenceRichtext
  settings:
    tinymce:
      style_formats:
        - title: 'Subheadline'
          block: 'h3'
~~~

::: tip INFO
See the [Tinymce customization guide](customize_tinymce) for all available options
:::

## EssencePicture

Store references to pictures the editor assigns from the library.

The editor partial is rendered as a picture editor with a lot of options (i.e. image cropper).

The view partial renders the assigned picture, resizes it and crops it if needed.

::: tip INFO
See the [rendering images](render_images) guide for further information on the powerful image rendering engine of Alchemy.
:::

### Settings

* **size** `String`

  The size the image should be downsized to. I.e. 400x300

* **crop** `Boolean` (default: false)

  Crop the image to given size. This also enables the build in cropper tool.

* **srcset** `Array<String>`

  A list of sizes of this image uses as sources list. Best used with the `sizes` setting for implementing responsive images.

* **sizes** `Array<String>`

  A list of screen sizes the image sources should be used for. Best used with the `srcset` setting for implementing responsive images.

## EssenceDate

Use this to store a `DateTime` value. Renders a datepicker in the editor partial.

The view output is passed through Rails' I18n Library, so it is fully localizable.

### Settings

* **date_format** `String|Symbol`
  Either a `String` with the format of [Rubys `strftime`](https://ruby-doc.org/stdlib-2.6.1/libdoc/date/rdoc/Date.html#method-i-strftime) or a `Symbol` of a [date/time format](https://guides.rubyonrails.org/i18n.html#adding-date-time-formats) defined in your locale files.

## EssenceHtml

Useful to store HTML code (i.e. a embed, or tracking code).

::: warning
The view renders the raw, not sanitized or escaped output.
**So be careful!**
:::

## EssenceBoolean

Stores a `Boolean` value in the database. Renders a checkbox in the editor partial.

## EssenceSelect

Renders a select box in the editor partial and stores the value as `String`.

Useful for letting your user select from a limited set of choices.

### Settings

* **select_values** `Array`

  A list of values your users can select from. Use [a two dimensional array](https://guides.rubyonrails.org/form_helpers.html#the-select-and-option-tags) for having value and text pairs.

### Example

~~~ yaml
- name: width
  type: EssenceSelect
  settings:
    select_values: ['200', '300', '400']
~~~

## EssenceLink

Stores a url in the database. Useful for linking things, where the editor should not set the linked text itself.

::: tip
If you want the linked text to be editable by the editor use the [**EssenceText**](#essencetext) with `linkable: true` option instead.
:::

## EssencePage _available since Alchemy 4.4_

References an [Alchemy::Page](pages).

Renders a select box in the editor partial.

Useful for contact form follow up pages or other use cases where you want to reference another page.

### Example

~~~ yaml
- name: contact_form
  contents:
    - name: follow_up_page
      type: EssencePage
~~~

## Rendering essences within your element

Similar to [rendering elements within layouts](elements.html#render_elements_in_your_layout), essences are rendered on an element. However, essences don't have their own partial views by default.

Page layouts use the `<%= render_elements %>` helper to load an elements partial, elements use the `<%= element_view_for() do ... end %>` helper to expose a block that makes it easy to access the essences (instances of `Alchemy::Content`) of the element.

For example. With an `article` element like this:

~~~ yaml
# config/alchemy/elements.yml
- name: article
  contents:
    - name: headline
      type: EssenceText
    - name: color
      type: EssenceText
      settings:
        input_type: color
    - name: copy
      type: EssenceRichtext
~~~

The `_article.html.erb` template generated by `rails alchemy:elements --skip` would look like this:

~~~erb
<%- cache(article) do -%>
  <%= element_view_for(article) do |el| -%>
    <div class="headline">
      <%= el.render :headline %>
    </div>
    <div class="color">
      <%= el.render :color %>
    </div>
    <div class="copy">
      <%= el.render :copy %>
    </div>
  <%- end -%>
<%- end -%>

~~~

Note how `element_view_for` allows you to call `el.render` on essences within the block.

::: tip
`element_view_for` wraps the block in a `<div>` with a set ID and class by default, but this is [customizeable](https://github.com/AlchemyCMS/alchemy_cms/blob/main/app/helpers/alchemy/elements_block_helper.rb#L137) if you pass in arguments:

`<%= element_view_for(article, tag: :span, id: 'custom_id', class: 'custom_class') do |el| -%>`
:::

Without using the `element_view_for` helper, you can still access essences:

`article.content_by_name('headline')&.ingredient`

But the `el.render` helper takes care of generating the appropriate DOM elements to display the essence based on its type. It is recommended you rely on these helpers unless you are comfortable with the structure of the essence model you are trying to render.

Just like `element_view_for`, you can pass `options` and `html_options` to `el.render`:

~~~erb
<%= el.render :headline, options = {}, html_options = {} %>
~~~

You can check out the [`ElementViewHelper#render` helper](https://github.com/AlchemyCMS/alchemy_cms/blob/b8cc62493693a070f7457081760708bf28c13e34/app/helpers/alchemy/elements_block_helper.rb#L31) for details.

::: warning NOTE
Options are not universally applied by all element types. For example, `ElementRichText` ignores `html_options` since it's already in HTML format and isn't wrapped in any special div. And you can't prevent the `ElementPicture` from generating an `<img>` tag.
:::

### Options

The `options = {}` hash is used for formatting the child elements' tags. You can pass `false` to not include tags inside the wrapper element. For more, see the source code [here](https://github.com/AlchemyCMS/alchemy_cms/blob/56e95a07446fb6832676acfc1e8c7047d1b80495/app/helpers/alchemy/elements_block_helper.rb#L128) and [here](https://github.com/AlchemyCMS/alchemy_cms/blob/main/app/helpers/alchemy/elements_helper.rb#L193)

### Html_options

The `html_option = {}` are:
* `:tag` - The HTML tag to be used for the wrapping element
* `:id` - The wrapper tag's DOM ID.
* `:class` - The wrapper tag's DOM class.
