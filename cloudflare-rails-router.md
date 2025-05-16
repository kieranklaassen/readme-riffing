# Cloudflare Rails Router

Same‑domain edge routing between a Rails app and any marketing stack.

[![Gem Version](https://badge.fury.io/rb/cloudflare_rails_router.svg)](https://rubygems.org/gems/cloudflare_rails_router)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Why?

* **One canonical domain** – visitors and crawlers only see `yourdomain.com`, boosting SEO and trust.
* **First‑party cookies** – referral and attribution data remain intact; no third‑party domains.
* **Zero redirects** – Cloudflare proxies directly; Core Web Vitals stay green.
* **Marketing agility** – content teams can ship landing pages on Webflow, Framer, or static hosting without touching Rails.
* **Edge decision** – cookie check happens at Cloudflare’s POP; Rails isn’t hit until needed.
* **SEO control** – optionally serve *all* search‑engine requests from the marketing origin even while users hit the app.

---

## Installation

Add to your **Gemfile**:

```ruby
gem "cloudflare_rails_router"
```

Then run:

```bash
bundle install
```

---

## Quick Start

```bash
rails g cloudflare:router:install   # copies Worker + initializer
wrangler deploy                     # pushes Worker to Cloudflare
```

---

## Usage

### Redirect anonymous visitors to marketing

```ruby
class ApplicationController < ActionController::Base
  before_action :redirect_marketing! unless user_signed_in?

  private

  def redirect_marketing!
    cloudflare_redirect_to(:marketing)
  end
end
```

`cloudflare_redirect_to` relies on `marketing_origin` from your initializer—no string arguments needed.

```ruby
# config/initializers/cloudflare_rails_router.rb
CloudflareRailsRouter.configure do |c|
  c.app_origin = "https://yourdomain.com"
  c.marketing_origin = "https://marketing.framer.com"

  c.cookie_ttl = 10.minutes
  c.crawlers_to = :marketing
end
```

### Clearing/Refreshing the flag

The marketing cookie expires automatically after `cookie_ttl`.  If you need to refresh it sooner you can add `?cm=1` to any URL on the Rails **app** origin. A cloudflare middleware will delete the cookie and continue the request normally (user ends up back in the app or redirected to marketing with any new cookies).


