<!--
  Ankane‑style README template 🇯🇵
  --------------------------------
  • Write in imperative voice ("Add", "Run", "Create").
  • Keep sentences ≤ 15 words.
  • One code fence per logical example.
  • Delete ALL comments (<!-- ... -->) before publishing.

\-->

# GemName

<!-- ⚠️ Replace `GemName` with your gem’s actual name -->

One‑sentence tagline describing what the gem does.

[![Gem Version](https://badge.fury.io/rb/<gemname>.svg)](https://rubygems.org/gems/<gemname>)
[![Build](https://github.com/<user>/<gemname>/actions/workflows/ci.yml/badge.svg)](https://github.com/<user>/<gemname>/actions)
[![Ruby](https://img.shields.io/badge/ruby-%3E%3D_3.2.0-brightgreen.svg)](https://www.ruby-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Installation

<!-- Always the first real section. Use exact wording below. -->

Add this line to your application’s **Gemfile**:

```ruby
gem "<gemname>"
```

And run:

```bash
bundle install
```

---

## Quick Start

<!-- Provide the fastest path. Avoid prose between code fences. -->

```bash
rails g <gemname>:install   # interactive installer
```

---

## Usage

<!-- Include at least one basic & one advanced example. -->

```ruby
# basic example – returns an array of results
Model.search "term"
```

```ruby
# advanced example – disable misspellings and highlight results
Model.search "term", misspellings: false, highlight: {tag: "<mark>"}
```

### Style Guidelines for Examples

<!-- Keep guideline list short. -->

* Prefer **single‑purpose** code fences.
* Inline comments lowercase, ≤60 chars.
* Two‑space indentation.

### Options

<!-- ≤10 rows. One‑line descriptions. -->

| Option          | Description                | Default |
| --------------- | -------------------------- | ------- |
| `misspellings:` | Enable fuzzy matching      | `true`  |
| `highlight:`    | Return highlighted results | `false` |
| `load:`         | Eager‑load records         | `true`  |

---

## Upgrading

<!-- Bullet each breaking change. -->

* **1.0 → 2.0** – Rename `foo` to `bar`.
* **0.x → 1.0** – Run the install generator and migrate.

---

## Contributing

Everyone is encouraged to help improve this project. Fork, make changes, and open a pull request.

---

## License

MIT

---

<!--
README CHECKLIST – delete after validating ✔️
- Header: name, tagline, ≤4 badges
- Installation section with Gemfile snippet
- Quick Start code block (generator or rake task)
- Usage: basic + advanced examples
- Options table ≤10 rows
- Upgrading notes if needed
- Ends with Contributing & License
- Sentences ≤15 words, imperative voice
- No marketing fluff beyond tagline
-->
