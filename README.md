# tachyne — quickstart

> tachyne is an unofficial fan project, not affiliated with Mojang,
> Microsoft, or Minecraft's developer/publisher in any way. See the
> Disclaimer at the bottom.

## Project status

**Work in progress.** tachyne is young and moving fast: a full survival game
runs today, but expect rough edges, missing vanilla features, and breaking
changes between updates. **Bug reports are genuinely useful** — please open a
GitHub Issue with your client version/edition and what you saw. Contributions are
welcome too — see each code repo's `CONTRIBUTING.md`.


**One world, every client.** tachyne is a Minecraft-compatible server written
from scratch in pure Go, with a versionless core: the engine simulates the
world and emits typed domain events; per-version gateways render them into
whatever wire format each client speaks. Java 1.21.5–1.21.8, Java 26.2, and
Bedrock all join the same world — no client mods.

## Run it in one command (Docker)

```sh
TACHYNE_OPS=YourPlayerName docker compose up -d
```

- **Java** (1.21.5–1.21.8 or 26.2): connect to `<this-host>:25565`
- **Bedrock** (latest): connect to `<this-host>:19132`

That's the **classic experience**: a single world container, procedurally
generated, effectively infinite, full survival — no sharding, no boundaries.

### Variant: walk the real Cape Town

```sh
TACHYNE_OPS=YourPlayerName docker compose -f docker-compose.yml -f docker-compose.earth.yml up -d
```

Earth mode replaces the terrain generator with real elevation data
(Copernicus GLO-30): greater Cape Town at **1:1 scale**. You spawn in the
city bowl looking up at Table Mountain, in creative so you can fly it.

## Run it on Kubernetes

```sh
kubectl apply -f k8s/tachyne.yaml
```

The same single-world stack as the compose file (change the attach token in
the Secret first). For the **sharded multi-pod world** — one world split
across pods with seamless handover — and earth-mode deployment, see
[tachyne-world](https://github.com/tachyne/tachyne-world) (`deploy/`,
`docs/SHARDING-BUILD.md`, `docs/EARTH.md`).

## How it stitches together

```mermaid
flowchart LR
    subgraph clients [Clients]
        J1["Java 1.21.5–1.21.8"]
        J2["Java 26.2"]
        B["Bedrock (phone/console/PC)"]
    end

    subgraph edge [Front door]
        I["tachyne-ingress<br/>:25565 tcp · :19132 udp"]
    end

    subgraph gateways [Per-version gateways — all Minecraft protocol lives here]
        G1["gw-java-770"]
        G2["gw-java-776"]
        G3["gw-bedrock"]
    end

    subgraph engine [The versionless engine]
        W0["tachyne-world<br/>(shard 0)"]
        W1["tachyne-world<br/>(shard N, optional)"]
    end

    A["tachyne-access<br/>whitelist · bans · roles (optional)"]

    J1 -->|handshake proto 770–772| I
    J2 -->|handshake proto 776| I
    B -->|RakNet/UDP| I
    I --> G1
    I --> G2
    I --> G3
    G1 -->|attach protocol :25500<br/>typed domain events| W0
    G2 --> W0
    G3 --> W0
    G1 -.->|login check| A
    G2 -.-> A
    G3 -.-> A
    W0 <-->|peer mesh :25501<br/>handover · cross-seam shadows| W1
```

The key property: the engine has **no Minecraft wire code at all**. Adding
support for a new Minecraft version means a new translation step in a
gateway — the world, and everyone playing in it, never changes.

## The repos

| Repo | Role |
|---|---|
| [tachyne-world](https://github.com/tachyne/tachyne-world) | the engine: simulation, worldgen, sharding, earth mode |
| [tachyne-common](https://github.com/tachyne/tachyne-common) | shared library: attach protocol, renderer, translation chain, gateway pipeline |
| [tachyne-gw-java-770](https://github.com/tachyne/tachyne-gw-java-770) · [-776](https://github.com/tachyne/tachyne-gw-java-776) · [-bedrock](https://github.com/tachyne/tachyne-gw-bedrock) | per-version client gateways |
| [tachyne-ingress](https://github.com/tachyne/tachyne-ingress) | the front door: version routing + UDP forwarding |
| [tachyne-access](https://github.com/tachyne/tachyne-access) | authorization: whitelist, bans, roles, IP ACL |

Contributions welcome — see each repo's `CONTRIBUTING.md`.

## License

Apache-2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE).

## Disclaimer

tachyne is an unofficial, independent project. It is **not** affiliated with,
endorsed, sponsored, or approved by Mojang Studios, Mojang Synergies AB,
Microsoft Corporation, or any of their subsidiaries — the developer and
publisher of Minecraft have no involvement with this project. "Minecraft" is
a trademark of Mojang Synergies AB. This project contains no Minecraft game
code.
