# Hotkey

Keyboard shortcuts for Hotwire apps. No dependencies. No configuration. Just HTML.

## The Problem

Most hotkey libraries want you to:
- Define shortcuts in JavaScript
- Maintain a central registry
- Handle cleanup manually
- Ship kilobytes of code

That's backwards. The shortcut should live where the action lives.

## The Solution

```erb
<%= link_to "Search", search_path, hotkey: :k %>
```

Press `K` anywhere, link gets clicked. Remove the element, listener goes away. Done.

## Installation

```ruby
gem 'hotkey-rails'
```

```bash
bundle install
bin/rails hotkey:install
```

This adds:
- `app/javascript/controllers/hotkey_controller.js` (25 lines)
- `app/helpers/hotkey_helper.rb` (60 lines)
- Styling for `<kbd>` elements

## Usage

### The `hotkey:` option

Works with `link_to`, `button_to`, and `button_tag`:

```erb
<%= link_to "Back", root_path, hotkey: :esc %>
<%= link_to "Search", search_path, hotkey: :k %>
<%= button_to "New Card", cards_path, hotkey: :c %>
```

### Modifier keys

Pass an array:

```erb
<%= button_tag "Save", hotkey: [:ctrl, :enter] %>
<%= link_to "Jump", jump_path, hotkey: [:meta, :j] %>
<%= button_to "Toggle", toggle_path, hotkey: [:shift, :g] %>
```

For `:ctrl`, both `Ctrl` (Windows/Linux) and `Cmd` (Mac) are automatically bound.

### Visual hints

Use `hotkey_hint` to show the shortcut:

```erb
<%= link_to cards_path, hotkey: :c do %>
  Add a card <%= hotkey_hint(:c) %>
<% end %>
```

Renders: `Add a card <kbd class="hide-on-touch">C</kbd>`

The `hide-on-touch` class hides the hint on touch devices where keyboard shortcuts don't apply.

### Tooltip labels

Use `hotkey_label` for platform-aware labels:

```erb
<%= button_tag "Save",
      title: "Save changes (#{hotkey_label(:ctrl, :enter)})",
      hotkey: [:ctrl, :enter] %>
```

Displays "Save changes (⌘Return)" on Mac, "Save changes (Ctrl+Enter)" elsewhere.

### Focus instead of click

```erb
<%= text_field_tag :search,
      placeholder: "Search [F]",
      data: hotkey(:f, action: :focus) %>
```

### Plays nice with other data attributes

```erb
<%= link_to "Edit", edit_path,
      hotkey: :e,
      data: { turbo_frame: "modal", confirm: "Sure?" } %>
```

## Helpers

### `hotkey(*keys, action: :click)`

Returns data attributes hash:

```ruby
hotkey(:c)
# => { controller: "hotkey", action: "keydown.c@document->hotkey#click" }

hotkey(:ctrl, :enter)
# => { controller: "hotkey", action: "keydown.ctrl+enter@document->hotkey#click keydown.meta+enter@document->hotkey#click" }
```

### `hotkey_label(*keys)`

Platform-aware display string:

```ruby
hotkey_label(:c)              # => "C"
hotkey_label(:ctrl, :enter)   # => "⌘Return" (Mac) / "Ctrl+Enter" (other)
hotkey_label(:shift, :g)      # => "⇧G" (Mac) / "Shift+G" (other)
```

### `hotkey_hint(*keys)`

The `<kbd>` element:

```ruby
hotkey_hint(:c)              # => <kbd class="hide-on-touch">C</kbd>
hotkey_hint(:ctrl, :enter)   # => <kbd class="hide-on-touch">⌘Return</kbd>
```

## The Full Helper

```ruby
# app/helpers/hotkey_helper.rb

module HotkeyHelper
  def hotkey(*keys, action: :click)
    keys = keys.flatten.map(&:to_s)

    actions = if keys.include?("ctrl")
      chord = keys.join("+")
      meta_chord = keys.map { |k| k == "ctrl" ? "meta" : k }.join("+")
      "keydown.#{chord}@document->hotkey##{action} keydown.#{meta_chord}@document->hotkey##{action}"
    else
      "keydown.#{keys.join("+")}@document->hotkey##{action}"
    end

    { controller: "hotkey", action: actions }
  end

  def hotkey_label(*keys)
    keys.flatten.map do |key|
      case key.to_s
      when "ctrl"  then mac? ? "⌘" : "Ctrl+"
      when "shift" then mac? ? "⇧" : "Shift+"
      when "alt"   then mac? ? "⌥" : "Alt+"
      when "enter" then mac? ? "Return" : "Enter"
      when "esc"   then "Esc"
      else key.to_s.upcase
      end
    end.join.gsub(/[⌘⇧⌥]\+/, '\0'.chop)
  end

  def hotkey_hint(*keys)
    tag.kbd hotkey_label(*keys), class: "hide-on-touch"
  end

  # Extended Rails helpers
  def link_to(name = nil, options = nil, html_options = nil, &block)
    html_options, options, name = options, name, block if block_given?
    html_options ||= {}

    if (keys = html_options.delete(:hotkey))
      html_options = merge_hotkey(html_options, keys)
    end

    super(name, options, html_options, &block)
  end

  def button_to(name = nil, options = nil, html_options = nil, &block)
    html_options, options = options, name if block_given?
    html_options ||= {}

    if (keys = html_options.delete(:hotkey))
      html_options = merge_hotkey(html_options, keys)
    end

    super(name, options, html_options, &block)
  end

  def button_tag(content_or_options = nil, options = nil, &block)
    if block_given?
      options = content_or_options || {}
    else
      options ||= {}
    end

    if (keys = options.delete(:hotkey))
      options = merge_hotkey(options, keys)
    end

    super(content_or_options, options, &block)
  end

  private

  def mac?
    request.user_agent&.match?(/Mac/)
  end

  def merge_hotkey(html_options, keys)
    keys = Array(keys)
    hotkey_data = hotkey(*keys)

    html_options[:data] ||= {}
    html_options[:data][:controller] = [html_options[:data][:controller], hotkey_data[:controller]].compact.join(" ")
    html_options[:data][:action] = [html_options[:data][:action], hotkey_data[:action]].compact.join(" ")
    html_options
  end
end
```

## The Stimulus Controller

```javascript
// app/javascript/controllers/hotkey_controller.js

import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  click(event) {
    if (this.#shouldHandle(event)) {
      event.preventDefault()
      this.element.click()
    }
  }

  focus(event) {
    if (this.#shouldHandle(event)) {
      event.preventDefault()
      this.element.focus()
    }
  }

  #shouldHandle(event) {
    return !event.defaultPrevented &&
           !event.target.closest("input, textarea, [contenteditable]")
  }
}
```

## The CSS

```css
kbd {
  border: 1px solid;
  border-radius: 0.3em;
  box-shadow: 0 0.1em 0 currentColor;
  font-family: ui-monospace, monospace;
  font-size: 0.8em;
  font-weight: 600;
  opacity: 0.7;
  padding: 0 0.4em;
  text-transform: uppercase;
  vertical-align: middle;
}

.hide-on-touch {
  @media (any-hover: none) {
    display: none;
  }
}
```

Uses `currentColor` — works everywhere without color configuration.

## How It Works

The magic is Stimulus action descriptors:

```
keydown.k@document->hotkey#click
   │     │    │        │     │
   │     │    │        │     └── method to call
   │     │    │        └── controller name
   │     │    └── listen at document level (global)
   │     └── the specific key
   └── event type
```

When the element is removed from the DOM, Stimulus automatically removes the listener. No cleanup code. No memory leaks. No registry.

## Before & After

```erb
<%# Before: 94 characters %>
<%= link_to "Back", path, data: { controller: "hotkey", action: "keydown.esc@document->hotkey#click" } %>

<%# After: 38 characters %>
<%= link_to "Back", path, hotkey: :esc %>
```

That's Rails.

## Credits

Pattern extracted from [Fizzy](https://fizzy.dev) by 37signals.

## License

MIT
