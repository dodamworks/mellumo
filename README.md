# Mellumo

Open-source, agent-driven social media automation, built on [Postiz](https://github.com/gitroomhq/postiz-app).

**Status: foundation stage.** Mellumo starts from Postiz v2.25.3 and retains its application features. The current application UI still uses upstream branding. Goal-to-completion automation is the direction of development, not a finished capability.

[한국어](README.ko.md) · [Getting started](docs/DEVELOPMENT.md) · [Architecture](docs/ARCHITECTURE.md) · [Roadmap](docs/ROADMAP.md) · [Contributing](CONTRIBUTING.md)

## Direction

Give an AI agent a social media objective. It plans the work, prepares content, uses the publishing tools, follows asynchronous results, and reports what actually completed. The web app provides account setup, objectives, progress and results. Review is configurable rather than a mandatory step for every post.

## Reused foundation

Postiz supplies the calendar/composer, channel integrations, media handling, scheduling, analytics, team features, AI tools, MCP/REST interfaces and Temporal publishing workflows. Availability depends on platform permissions, configuration and external provider access. AI providers and optional SDKs may have separate costs and terms.

Mellumo will connect persistent objectives, task progress, recovery and completion checks around this foundation. It does not yet provide a verified autonomous campaign loop or a unified comments/DM agent.

## Run from this source

See [development setup](docs/DEVELOPMENT.md). `compose.mellumo.yaml` builds the checked-out source; it does not silently pull a Postiz image and present it as a Mellumo build. No official Mellumo image or hosted service is published yet.

## License and origin

Mellumo is an independent derivative of Postiz, under **GNU AGPL version 3**. Original copyright and license notices remain in place. See [LICENSE](LICENSE), [NOTICE](NOTICE), [upstream provenance](docs/UPSTREAM.md) and [third-party boundaries](docs/THIRD_PARTY.md).

Mellumo is not an official Postiz product. Contributions are accepted under the same license; Mellumo does not require the upstream Postiz CLA. Security reports: [SECURITY.md](SECURITY.md).
