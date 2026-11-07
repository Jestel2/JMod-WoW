# WoW live baseline — 2026-11-07

This is a read-only runtime snapshot to accompany the [world-simulation roadmap](world-simulation-roadmap.md). It supersedes earlier “not deployed” statements for the family feature, but it does not claim that every gameplay path was tested.

## Current live state

- Tyler reports the Admin panel is live and the addon opens the `/family` menu.
- Read-only Docker inspection found `ac-worldserver` running `jester-wow-worldserver:family-test`, image SHA `sha256:2046beee5d35c8178933186a3239019226b515c6041c77883ffb70bbefdde020`. A binary-string probe found `JModFamily`, `jmodfamily`, and `mod-jmod-family`, confirming the family server code is compiled into this image. Individual family commands still need gameplay checks.
- `ac-authserver` runs `jester-wow-authserver:core-9d139488`; `ac-database` is healthy on MySQL 8.4.
- The live source checkout at `/home/tberno/azerothcore-upgrade-20261103` is detached at `9d139488` and contains only `mod-autobalance` under `modules/`. The running worldserver binary probe found no `RandomBot`, `Playerbot`, `mod-playerbots`, `CityLife`, or `mod-playerbots-city-life` marker.
- A dated pre-upgrade database archive exists at `/home/tberno/wow-jmod-cutover-20261107-141641/pre-upgrade.sql.gz`; `gzip -t` passed. No restore test has been performed.
- The live VM has 4 CPUs and about 60 GiB free disk. Two instantaneous samples showed worldserver at 59–86% CPU and about 1.95 GiB RAM. Do not start a large bot population or compile a core on this VM.
- The separate build workspace has 4 CPUs, 7.2 GiB RAM and 27 GiB free disk. Any candidate build there needs conservative parallelism and disk monitoring.

## Farming and city population candidates

The upstream project documentation and current remote refs confirm:

- [Playerbots core fork](https://github.com/mod-playerbots/azerothcore-wotlk/tree/Playerbot), ref `e7e2e288e7d9ec30e2dfe6f417178a95986fcbb2`.
- [mod-playerbots](https://github.com/mod-playerbots/mod-playerbots), ref `405f81196161017eebae3ee7a612d786a3ed2a9d`. Its README requires the custom Playerbot core fork; it says the standard AzerothCore repository will not work.
- [City Life](https://github.com/chenyifeiai/mod-playerbots-city-life), ref `ce6a05df450f04c4961d09254ad73cf11f1afee2`.

City Life uses already-online random bots as residents at capitals, towns, and social hubs. It does not create the bot pool or perform auction-house trading. Farming and population generation therefore start with the Playerbot core and module, not City Life.

The current core pin `9d139488` is not the Playerbot fork. Do not drop `mod-playerbots` into the live core checkout. First build a separate candidate, port and compile the existing JMod family and AutoBalance modules against that candidate, and apply Playerbot/City Life SQL only to a disposable database copy.

## Work performed

Only read-only inspection was performed on the live VM. No containers, configuration, or databases were changed and no bots were started. The detailed working handoff remains in the user vault at `.tekton-vault/Projects/JesterTek/WoW/World-Simulation.md`.
