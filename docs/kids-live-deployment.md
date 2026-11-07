# JMod kids' features: live rollout handoff

Status as of 2026-11-07: **not deployed**. The live \`ac-worldserver\` and \`ac-authserver\` still use their original \`acore/ac-wotlk-*:master\` images. The new candidate build and staging DB migration have passed independent validation, but real gameplay is not tested.

## Scope

**Deliver now:** family travel/summon requests, accept/decline, city hubs, standard party invitations, \`/family\` and \`/guide\` UI. The player commands are implemented at \`server/modules/mod-jmod-family/src/JModFamily.cpp\`; the addon at \`jc/JesterConsole/JesterFamily.lua\`.

**Not implemented yet:** forced/automatic group membership on login, a true quest-route arrow or automatic navigation, autonomous playerbots. Do not claim these work. Cross-faction groups require the server config to allow them. \`mod-playerbots\` requires a different core fork and is **out of scope** for this safe cutover.

**Git:** canonical repo is \`https://github.com/Jestel2/JMod-WoW\`. Use branch \`feature/jmod-family-server\` (head \`4713bc71f79e8c7de2b4c06ea56ac2ebd8e943cc\`) or PR #7 against \`feature/bootstrap-admin-jc\`. **Do not deploy \`main\`**: the \`main\` tree does not contain these feature files. **Do not reset** the potentially dirty local \`/opt/JMod-WoW\` checkout.

## Facts already verified

- AzerothCore source pinned to \`9d139488\`; AutoBalance module \`1d707cb\`.
- Successfully compiled image: \`jester-wow-worldserver:family-test\`, SHA \`2046beee5d35c8178933186a3239019226b515c6041c77883ffb70bbefdde020\`.
- Matching images: \`jester-wow-authserver:core-9d139488\` and \`jester-wow-db-import:core-9d139488\`.
- \`jmodfamily\` string found in compiled worldserver; \`AutoBalance.conf.dist\` present.
- Staging MySQL 8.4 import of a copy of \`acore_auth\`, \`acore_world\` and \`acore_characters\` passed with 439 queries applied. Character count was 11 after upgrade.
- \`docker-compose.jmod-upgrade.yml\` in \`/opt/azerothcore/azerothcore-wotlk\` was created and validated; its image targets match the three candidate images.
- Failed live cutovers on November 4 reached the **pre-migration** configuration step, then stopped. Original services restarted; nothing indicated the live SQL migration had run. Recorded backup folders \`~/wow-jmod-cutover-20261104-201937\` and \`~/wow-jmod-cutover-20261104-204521\`.
- The previous cutover script \`~/jmod-cutover.sh\` has a bug: attempting \`Path.write_text\` on root-owned \`env/dist/etc/worldserver.conf\` as the \`tberno\` user. Resolve this *while the original services remain running*, not after stopping them.

## Remote execution plan (for RobotChat with approved SSH/sudo to \`wow\`)

1. Read-only inspect \`git status\` for both \`/opt/JMod-WoW\` and \`/opt/azerothcore/azerothcore-wotlk\` and inspect Compose service definitions. Do not overwrite dirty files.
2. Verify production is still on the old images, backups exist and decompress, all candidate images exist, \`ac-database\` healthy, disk space enough, there are no active players, and \`~/jmod-cutover.sh\` is the intended reviewed cutover script. If any precondition fails, stop.
3. Obtain sudo noninteractively via an approved credential flow. Use \`sudo -n true\` to preflight, but **do not request or expose credentials in output**. Back up \`env/dist/etc/worldserver.conf\` as root to an existing secure backup directory, then modify *only* \`AllowTwoSide.Interaction.Group = 1\`, using root permissions. Handle duplicate active keys (fail, don't guess); verify one active key is set to \`1\`. Cross-faction grouping may be disallowed otherwise.
4. Patch the existing cutover script to **replace** its entire \`ENABLE CROSS-FACTION PARTY INVITES\` block with a read-only verification against the already-updated config; review the patch with \`diff -u\`, run \`bash -n\`. No writable configuration operation should remain *after* the old services stop.
5. Get clear confirmation there are no active players; then execute the reviewed cutover script (the user has already approved proceeding) and capture exit status/logs. Use the three prebuilt image tags, never \`--build\`.
6. Shutdown order: stop \`ac-worldserver\` and \`ac-authserver\` only; leave the existing \`ac-database\`, \`wow-admin\`, \`wow-model-assets\`, and persistent volumes untouched. Fresh database backup must use \`mysqldump --single-transaction --routines --triggers --events\` for the three acore DBs; verify gzip and record character count before migration.
7. Run **only** the pinned \`jester-wow-db-import:core-9d139488\` image against the live DB, with \`docker compose -f docker-compose.yml -f docker-compose.jmod-upgrade.yml run --rm --no-deps --pull never -e AC_DISABLE_INTERACTIVE=1 ac-db-import\`. Observe importer exit=0 and SQL logs before starting new worldserver. If migration fails, stop and do not boot an old binary against a partially migrated database.
8. Start only the prebuilt \`ac-authserver\` and \`ac-worldserver\` with \`docker compose ... up -d --no-deps --no-build --pull never --force-recreate ac-authserver ac-worldserver\`; **do not** invoke unguarded \`docker compose up\`, and **never** \`docker compose down -v\`. Keep the pre-upgrade images tagged \`jester-wow-*:pre-jmod\`.
9. Validate startup logs (\`acore_world\` loaded, modules loaded, auth ready, no missing SQL schema updates), port connectivity 3724 and 8085, character count 11, correct \`docker inspect .Config.Image\` and \`docker compose ps\`. A listening port alone does *not* prove game functionality.
10. Copy the current \`feature/jmod-family-server\` addon into the WoW 3.3.5a \`Interface/AddOns/JesterConsole\` folder on **each** client machine without deleting custom settings. If addons are delivered through a separate launcher/share, inspect and use its existing mechanism first. Do not assume changing \`/opt/JMod-WoW\` changes client installations.
11. With two player characters in a party, test invite acceptance, \`/family\`, \`.jmodfamily go NAME\` -> recipient \`.jmodfamily accept\`, summon reverse direction, \`.jmodfamily decline\`, blocked combat/instance teleport, \`.jmodfamily hub dalaran\`, and \`/guide\` quest objective display. Test cross-faction characters if needed.
12. Report actual status and failures with logs. **No automatic DB restore** if any migration has started; rollback requires stopping the new services, restoring the matching pre-upgrade database snapshot and old images together under an explicit recovery plan. Preserve original backed-up data.

## Read-only diagnostic commands

\`\`\`bash
set -euo pipefail
cd /opt/azerothcore/azerothcore-wotlk
docker compose ps
docker inspect ac-worldserver ac-authserver --format '{{.Name}} {{.Config.Image}} {{.Image}} {{.State.Status}}'
docker image inspect \
  jester-wow-worldserver:family-test \
  jester-wow-authserver:core-9d139488 \
  jester-wow-db-import:core-9d139488 \
  --format '{{.RepoTags}} {{.Id}}'
docker compose -f docker-compose.yml -f docker-compose.jmod-upgrade.yml config --quiet
git -C /opt/JMod-WoW status -sb
git -C /opt/azerothcore/azerothcore-wotlk status -sb
grep -n 'ENABLE CROSS-FACTION\|MIGRATE PRODUCTION\|VERIFY CROSS-FACTION' ~/jmod-cutover.sh
\`\`\`

## Live configuration fix (must be done *before* downtime)

Take a root-owned backup using a dated, secure file name; authorize sudo using RobotChat's approved workflow. The example assumes \`sudo -n\` is available. Stop if authentication is not available.

\`\`\`bash
set -euo pipefail
CONF=/opt/azerothcore/azerothcore-wotlk/env/dist/etc/worldserver.conf
sudo -n test -f "$CONF"
sudo -n cp -p "$CONF" "$CONF.pre-jmod-$(date +%Y%m%d%H%M%S)"
sudo -n python3 - "$CONF" <<'PY'
from pathlib import Path
import re, sys
p = Path(sys.argv[1])
s = p.read_text()
rx = re.compile(r'^[ \t]*AllowTwoSide\.Interaction\.Group[ \t]*=[^\r\n]*$', re.M)
hits = rx.findall(s)
if len(hits) > 1:
    raise SystemExit('Multiple active AllowTwoSide.Interaction.Group settings; aborting')
if hits:
    s = rx.sub('AllowTwoSide.Interaction.Group = 1', s)
else:
    s = s.rstrip('\n') + '\nAllowTwoSide.Interaction.Group = 1\n'
p.write_text(s)
PY
sudo -n grep -nE '^[[:space:]]*AllowTwoSide\.Interaction\.Group[[:space:]]*=' "$CONF"
\`\`\`

Afterward, **inspect and patch** \`~/jmod-cutover.sh\` to remove its privileged write step; do not repeatedly rerun it unchanged. Inspect current root config, stop order, fresh backup output, importer exit status, and image selection immediately before proceeding.
