# SolidQueue Autoscaler

Automatically right-sizes your Solid Queue worker fleet by queue *latency*.

[![Gem Version](https://badge.fury.io/rb/solidqueue-autoscaler.svg)](https://rubygems.org/gems/solidqueue-autoscaler)
[![Build](https://github.com/your-org/solidqueue-autoscaler/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/solidqueue-autoscaler/actions)
[![Ruby](https://img.shields.io/badge/ruby-%3E%3D_3.2.0-brightgreen.svg)](https://www.ruby-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Installation

Add this line to your application’s **Gemfile**:

```ruby
gem "solidqueue-autoscaler"
```

And run:

```bash
bundle install
```

---

## Quick Start

```bash
rails g solidqueue:autoscaler:install
```

The installer:

* Creates `config/initializers/solidqueue_autoscaler.rb` with helpful comments
* Appends `config/solid_queue/recurring.yml` so the autoscaler runs each hour
* Prints any required environment variables (for example `RENDER_TOKEN`)

Deploy and you’re done—no extra cron jobs needed.

---

## Usage

The installer schedules the autoscaler to run **hourly** via Solid Queue recurring jobs. You normally don’t need to call it yourself, but the API is handy for custom dashboards or manual scaling.

```ruby
# ask how many workers you should run (does not scale)
SolidQueue::Autoscaler.recommended_instances
```

```ruby
# scale manually with your own adapter
count = SolidQueue::Autoscaler.recommended_instances
MyAdapter.scale!(count)
```

### Options

| Option                     | Description                                | Default |
| -------------------------- | ------------------------------------------ | ------- |
| `min_instances:`           | Minimum workers to keep                    | `5`     |
| `max_instances:`           | Hard ceiling on workers                    | `20`    |
| `queue_latency_threshold:` | Seconds oldest job may wait before scaling | `30`    |
| `scale_increment:`         | Base step size when scaling                | `5`     |
| `scale_down_threshold:`    | Jobs/min decrease per −1 instance          | `100`   |

---

## Upgrading

* **0.x → 1.0** – Re‑run the install generator and review your initializer diff.

---

## Contributing

Everyone is encouraged to help improve this project. Fork, make changes, and open a pull request.

---

## License

MIT
