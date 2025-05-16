SolidQueue Autoscaler

Automatically right-sizes your SolidQueue worker fleet based on queue health, not guesswork.



SolidQueue Autoscaler is a tiny, dependency-free Ruby gem that watches your solid_queue_ready_executions table and tells your hosting platform (Render, Fly, Heroku, etc.) exactly how many worker instances you need. It scales up when jobs pile up and down when the queue drains—saving money without hurting latency.

Built for Rails 7.1+ and Solid Queue >= 0.3, 100 % thread-safe.

⸻

✨ Features
	•	Latency-driven scaling – based on the oldest job waiting, not raw depth.
	•	Proportional growth – the bigger the backlog, the bigger the step-up.
	•	Fast idle shutdown – auto-parks at MIN_INSTANCES after ⏲ 5 min empty.
	•	Hysteresis & distributed lock – no thrashing, no double API calls.
	•	One-liner API – SolidQueue::Autoscaler.recommended_instances.
	•	Zero monkey-patching – pure Ruby, works alongside any Rails scheduler.

⸻

🔧 Installation

bundle add solidqueue-autoscaler


⸻

⚙️ Quick Start

1 · Configure the Autoscaler

Create config/initializers/solidqueue_autoscaler.rb:

SolidQueue::Autoscaler.configure do |config|
  # === Adapter ========================================
  # Uses Render by default; swap in Fly / Heroku adapter as needed.
  config.adapter = SolidQueue::Autoscaler::Adapters::Render.new(
    api_token:  ENV.fetch("RENDER_TOKEN"),              # required
    service_id: ENV.fetch("RENDER_SERVICE_ID"),         # the worker service you want to scale
  )

  # === Optional tuning (sane defaults shown) ==========
  config.min_instances           = ENV.fetch("AUTO_SCALE_MIN",  5).to_i
  config.max_instances           = ENV.fetch("AUTO_SCALE_MAX", 20).to_i
  config.queue_latency_threshold = ENV.fetch("AUTO_SCALE_LATENCY", 30).to_i  # seconds
  config.scale_increment         = ENV.fetch("AUTO_SCALE_INCREMENT", 5).to_i
  config.scale_down_threshold    = 100  # jobs/min decrease = −1 instance
  config.redis_url               = ENV["REDIS_URL"]
end

Why no service_name? The adapter talks to Render’s API using the service_id you pass (or discovers it automatically when running inside Render). No extra naming step required.

2 · Schedule the Autoscaler job with Solid Queue

Add config/solid_queue/recurring.yml:

autoscaler_every_minute:
  class: "SolidQueue::Autoscaler::Job"      # provided by the gem
  cron:  "*/1 * * * *"                     # every minute
  queue: "high"

Solid Queue will enqueue the job once per minute. The job:
	1.	Measures queue latency & depth.
	2.	Calculates a recommended instance count.
	3.	Calls config.adapter.scale!(desired_count).

You’re done—no Cron, no Whenever, no external scheduler.

⸻

🖥 CLI Usage

$ solidqueue-autoscale --dry-run
Queue depth:      3 145
Oldest job age:   48 s
Current workers:  10
Recommended:      15 (scale ✚5)

Add --apply to perform the scale action.

⸻

🛠 API Usage within Rails

count = SolidQueue::Autoscaler.recommended_instances
Rails.logger.info("We ought to be running #{count} workers.")

Plug this into your own platform adapter if you don’t use Render.

⸻

🔌 Adapters

Adapter	Gem dependency	Example init
Render (default)	none	Adapters::Render.new(api_token: ..., service_id: ...)
Fly.io	fly-ruby ≥ 0.1	Adapters::Fly.new(app: ..., process_group: ...)
Heroku	platform-api ≥ 3.0	Adapters::Heroku.new(app: ..., process_type: ...)

Writing your own adapter is <10 LOC—see lib/solidqueue/autoscaler/adapters/base.rb.

⸻

🧪 Rails test helpers

class SolidQueueAutoscalerTest < ActiveSupport::TestCase
  include SolidQueue::Autoscaler::TestHelpers

  test "scales up on large backlog" do
    push_ready_jobs(5_000)
    assert_equal 15, SolidQueue::Autoscaler.recommended_instances
  end
end


⸻

📈 Metrics

SolidQueue::Autoscaler emits the following stats via ActiveSupport::Notifications:
	•	autoscaler.queue.latency – seconds
	•	autoscaler.queue.depth   – jobs
	•	autoscaler.recommended_instances

Hook them up to AppSignal / Datadog / Prometheus in one line:

SolidQueue::Autoscaler.subscribe_with(Appsignal)


⸻

🚦 Roadmap
	•	Predictive scaling for cron bursts
	•	Opt-in PG reltuples depth estimator
	•	Built-in Prometheus exporter

PRs welcome! ❤️

⸻

📜 License

MIT © 2025 Your Name or Company

See LICENSE for full text.