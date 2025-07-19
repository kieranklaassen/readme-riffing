# Rails JS Logger

Bridge JavaScript console output to Rails logs for unified debugging.

[![Gem Version](https://badge.fury.io/rb/rails-js-logger.svg)](https://rubygems.org/gems/rails-js-logger)
[![Build](https://github.com/your-org/rails-js-logger/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/rails-js-logger/actions)
[![Ruby](https://img.shields.io/badge/ruby-%3E%3D_3.2.0-brightgreen.svg)](https://www.ruby-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Why?

* **LLM monitoring** – AI assistants read JavaScript errors server‑side without browser MCP access.
* **Development first** – built for local development, works great in production too.
* **Unified logs** – JavaScript errors and logs appear alongside Rails logs.
* **Zero dependencies** – vanilla JavaScript capture, no npm required.
* **Production‑ready** – batching, sampling, and filtering prevent log flooding.
* **Developer experience** – `console.log` works identically in dev and production.
* **Error context** – stack traces include source maps when available.

---

## Installation

Add to your **Gemfile**:

```ruby
gem "rails-js-logger"
```

Then run:

```bash
bundle install
```

---

## Quick Start

```bash
rails g rails_js_logger:install   # adds routes + initializer + JS snippet
```

The installer:
* mounts `/rails/js_logs` endpoint
* adds JavaScript capture code to `application.html.erb`
* creates `config/initializers/rails_js_logger.rb`

---

## Usage

### Basic logging

```javascript
// your existing js code
console.log("User clicked checkout", {cart_id: 123})
console.error("Payment failed", error)
```

```ruby
# appears in rails logs as
[JSLogger] [log] User clicked checkout {"cart_id":123}
[JSLogger] [error] Payment failed TypeError: Cannot read property...
```

### Structured context

```javascript
// attach user/request context
window.railsJSLogger.setContext({
  user_id: <%= current_user&.id || 'null' %>,
  request_id: "<%= request.uuid %>"
})
```

```ruby
# logs include context
[JSLogger] [user=42,req=abc-123] [warn] Deprecation warning
```

### Error boundaries

```javascript
// automatically captured
window.addEventListener('error', (e) => {
  // rails-js-logger intercepts this
})

// also React error boundaries
componentDidCatch(error, info) {
  console.error("React error", {error, info})
}
```

---

## Configuration

| Option               | Description                          | Default       |
| -------------------- | ------------------------------------ | ------------- |
| `endpoint:`          | Path for log ingestion               | `/rails/js_logs` |
| `batch_size:`        | Logs per request                     | `10`          |
| `batch_interval:`    | Milliseconds between flushes         | `5000`        |
| `log_level:`         | Minimum level to capture             | `:debug`      |
| `sanitize_keys:`     | Keys to redact                       | `[:password, :token]` |
| `sample_rate:`       | Production sampling (0.0–1.0)        | `1.0`         |
| `max_payload_size:`  | Reject payloads over (bytes)         | `100_000`     |

---

## Advanced

### Custom formatting

```ruby
# config/initializers/rails_js_logger.rb
RailsJSLogger.configure do |c|
  c.formatter = ->(entry) {
    "[#{entry[:level]}] #{entry[:message]} #{entry[:context].to_json}"
  }
end
```

### Filtering sensitive data

```ruby
c.before_log = ->(entry) {
  entry[:data].deep_transform_keys!(&:to_s)
  entry[:data].except!("creditCard", "ssn")
  entry
}
```

### Conditional capture

```javascript
// disable for specific pages
window.railsJSLogger.disable()

// or conditionally
if (window.location.pathname === '/admin') {
  window.railsJSLogger.enable()
}
```

---

## Contributing

Everyone is encouraged to improve this project. Fork, make changes, and open a pull request.

---

## License

MIT