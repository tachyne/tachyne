# Contributing

This repo is the front door: the one-command way to run tachyne. Contributions
are welcome — especially anything that makes a first run smoother.

## Ground rules

- **Try it from scratch before you push.** `docker compose up -d` (and, if you
  touched the overlay, the earth-mode invocation) must bring up a world you can
  actually join on a clean machine.
- **Compose overlays REPLACE `command`, they don't merge it.** If you add a
  flag to the base world service, add it to `docker-compose.earth.yml` too —
  a missing `-nats` there silently disabled the whole plugin bus once.
- **Pin nothing to a private registry.** Images come from `ghcr.io/tachyne/*`
  and must be pullable by anyone.
- **Keep the security note honest.** A fresh server is open by default; any
  change that alters that must update the warning in the README.
- **The READMEs here are a beginner's first impression** — if a change makes a
  documented step wrong, fix the step in the same PR.

## Licensing of contributions

The project is Apache-2.0. Per its section 5, any contribution you
intentionally submit is licensed under the same terms, with no separate CLA.
Please make sure you have the right to submit what you contribute.

## Getting oriented

The README walks the whole stack; the deep design docs live in
[tachyne-world](https://github.com/tachyne/tachyne-world) under `docs/`.
