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

```bash
rails g solidqueue:autoscaler:install
```

The installer

* creates `config/initializers/solidqueue_autoscaler.rb` with comments
* creates `app/jobs/scale_solid_queue_workers_job.rb`
* appends `config/solid_queue/recurring.yml` so the autoscaler runs **every minute**

---

## Usage

The autoscaler job runs every minute via Solid Queue recurring tasks.  You normally don’t need to call it yourself.  If you prefer full control, enqueue your own wrapper job:

```ruby
# app/jobs/scale_solid_queue_workers_job.rb
class ScaleSolidQueueWorkersJob < ApplicationJob
  queue_as :maintenance

  def perform
    desired = SolidQueue::Autoscaler.recommended_instances
    MyAdapter.scale!(desired)
  end
end
```

Add this job to `config/solid_queue/recurring.yml` if you want to override the built‑in schedule.

---

## Choosing a scaling strategy

| Metric                                | Pros                                                                                                                                                                                | Cons                                                                                                                                                                      |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Latency** (age of oldest ready job) | • Directly reflects what *users* feel—long latency means real waiting time.  <br>• Independent of queue throughput spikes; a short burst that clears quickly won’t trigger scaling. | • Harder to convert into worker count without historical throughput (“n boxes?”).  <br>• A single slow-running job can make the queue look “slow” and cause over‑scaling. |
| **Depth** (number of ready jobs)      | • Simple to reason about—divide backlog by processing rate to size the fleet.  <br>• Popular with job‑queue autoscalers like HireFire.                                              | • Sudden spikes can overscale and thrash resources.  <br>• Doesn’t account for job complexity—100 tiny jobs ≠ 100 video transcodes.                                       |

> **Recommendation**: Start with `:latency`—it’s safer for user‑facing workloads. Switch to `:depth` for high‑throughput, uniform jobs where backlog size maps cleanly to capacity.

\------ | ---- | ---- |
\| **Latency** (age of oldest ready job) | • Directly reflects what *users* feel—long latency means real waiting time. ([judoscale.com](https://judoscale.com/blog/scaling-python-task-queues?utm_source=chatgpt.com), [github.com](https://github.com/sidekiq/sidekiq/wiki/Scaling-Sidekiq?utm_source=chatgpt.com))  <br>• Independent of queue throughput spikes; a short burst that clears quickly won’t trigger scaling. ([scaleops.com](https://scaleops.com/blog/kubernetes-autoscaling/?utm_source=chatgpt.com)) | • Harder to convert into worker count without historical throughput (“n boxes?”). ([software.land](https://software.land/throughput-vs-latency/?utm_source=chatgpt.com), [reddit.com](https://www.reddit.com/r/kubernetes/comments/1dfqabf/what_metrics_do_you_use_for_auto_scale/?utm_source=chatgpt.com))  <br>• A single slow-running job can make the queue look “slow” and cause over‑scaling. ([learn.microsoft.com](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements?utm_source=chatgpt.com), [github.com](https://github.com/sidekiq/sidekiq/wiki/Ent-Historical-Metrics?utm_source=chatgpt.com)) |
\| **Depth** (number of ready jobs) | • Simple to reason about—divide backlog by processing rate to size the fleet. ([help.hirefire.io](https://help.hirefire.io/article/52-hirefire-job-queue-size-any-language?utm_source=chatgpt.com), [docs.aws.amazon.com](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-using-sqs-queue.html?utm_source=chatgpt.com))  <br>• Popular with hosted autoscalers like HireFire. ([help.hirefire.io](https://help.hirefire.io/article/46-getting-started?utm_source=chatgpt.com)) | • Sudden spikes can overscale and thrash resources. ([elasticscale.com](https://elasticscale.com/blog/autoscale-ecs-with-sqs-queue-why-target-tracking-beats-step-scaling/?utm_source=chatgpt.com))  <br>• Doesn’t account for job complexity—100 tiny jobs ≠ 100 video transcodes. ([learn.microsoft.com](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements?utm_source=chatgpt.com)) |

> **Recommendation**: Start with `:latency`—it’s safer for user-facing workloads. Switch to `:depth` for high‑throughput, uniform jobs where backlog size maps cleanly to capacity.

---

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
