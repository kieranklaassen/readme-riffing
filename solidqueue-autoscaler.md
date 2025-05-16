# SolidQueue Autoscaler

Automatically right‑sizes your Solid Queue worker fleet by queue **latency** *or* queue **depth**.

[![Gem Version](https://badge.fury.io/rb/solidqueue-autoscaler.svg)](https://rubygems.org/gems/solidqueue-autoscaler)
[![Build](https://github.com/your-org/solidqueue-autoscaler/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/solidqueue-autoscaler/actions)
[![Ruby](https://img.shields.io/badge/ruby-%3E%3D_3.2.0-brightgreen.svg)](https://www.ruby-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Installation

Add to your **Gemfile**:

```ruby
gem "solidqueue-autoscaler"
```

Then run:

```bash
bundle install
```

---

## Quick Start

```bash
rails g solidqueue:autoscaler:install
```

The installer

* creates `config/initializers/solidqueue_autoscaler.rb` with comments
* appends `config/solid_queue/recurring.yml` so the autoscaler runs each hour
* prints any required env vars (e.g. `RENDER_TOKEN`)

Deploy and you’re done—no extra cron jobs.

---

## Usage

The autoscaler runs hourly via Solid Queue recurring jobs. Use the API for dashboards or manual scaling.

```ruby
# how many workers should we run? (no scaling side‑effect)
SolidQueue::Autoscaler.recommended_instances
```

```ruby
# manual scaling with a custom adapter
count = SolidQueue::Autoscaler.recommended_instances
MyAdapter.scale!(count)
```

---

## Choosing a scaling strategy

| Metric                                | Pros                                                                                                                                                                                | Cons                                                                                                                                                                      |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Latency** (age of oldest ready job) | • Directly reflects what *users* feel—long latency means real waiting time.  <br>• Independent of queue throughput spikes; a short burst that clears quickly won’t trigger scaling. | • Harder to convert into worker count without historical throughput (“n boxes?”).  <br>• A single slow-running job can make the queue look “slow” and cause over‑scaling. |
| **Depth** (number of ready jobs)      | • Simple to reason about—divide backlog by processing rate to size the fleet.  <br>                                              | • Sudden spikes can overscale and thrash resources.  <br>• Doesn’t account for job complexity—100 tiny jobs ≠ 100 video transcodes.                                       |

> **Recommendation**: Start with `:latency`—it’s safer for user‑facing workloads. Switch to `:depth` for high‑throughput, uniform jobs where backlog size maps cleanly to capacity.


## Configuration options

| Option                     | Description                                | Default    |
| -------------------------- | ------------------------------------------ | ---------- |
| `scaling_strategy:`        | `:latency` or `:depth`                     | `:latency` |
| `queue_latency_threshold:` | Oldest job age (sec) that triggers scaling | `30`       |
| `queue_depth_threshold:`   | Ready‑job count that triggers scaling      | `2_000`    |
| `min_instances:`           | Minimum workers to keep                    | `5`        |
| `max_instances:`           | Hard ceiling on workers                    | `20`       |
| `scale_increment:`         | Base step size when scaling up             | `5`        |
| `scale_down_threshold:`    | Jobs/min decrease per −1 instance          | `100`      |

> **Tip:** Switch strategies anytime by editing the initializer and restarting workers.

---

## Upgrading

* **0.x → 1.0** – Re‑run the install generator and review the initializer diff.

---

## Contributing

Everyone is encouraged to improve this project. Fork, make changes, and open a pull request.

---

## License

MIT
