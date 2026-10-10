# Minecraft agent downloads

## Replanter — Fabric 1.21.8, version 9

[Download planter-package.zip](https://github.com/Calculus1212/Codex/raw/main/planter-package.zip) | [Full replanter instructions](planter-README.md)

Version 9 fixes missing settings windows when Minecraft reports headless Java. In that case the CMD Java process displays the controls, independently of Minecraft's desktop-window restrictions. The supplied attach.cmd enables desktop windows for the controller only; no launcher JVM argument changes are needed. Settings are applied to the running agent through local Java attachment.

Configure batch delay, maximum block-use requests, range and the 7×7 checkbox pattern in the settings dialog. Defaults remain 100 ms, 70 requests, range 8 and all 49 cells selected. The pattern moves with the player, keeps its compass orientation and targets the support layer under the feet. Batches continue after the last target; only a completed pass restarts nearest-first. Delay and request-count edits preserve scan progress; pattern and range edits restart it. Uncheck Use checked cells to restore the full-radius scan. Click Apply settings to commit edits.

Restart Minecraft to upgrade. Extract the new package into a separate folder, run attach.cmd to find Minecraft's new PID, then run attach.cmd PID gui. Keep CMD open while using the external window. Closing that window leaves the agent attached; reopen with the same gui command to recover the applied settings and pattern. P toggles interactions; End stops. O can reopen an in-process window when Minecraft supports desktop windows; for headless Minecraft use the CMD command. Menus, empty hands and lost game focus pause requests. Settings last for the session. Keep the agent package outside the mods folder.

Hold seeds to plant on compatible prepared soil or another item to use its normal block interaction. Occupied spaces are skipped. The module does not harvest crops. The server controls reach, collisions, inventory and successful placement.

Package SHA-256: `46538f77d1964ca28857419b9905b2b5abb28f92b1d5ee8ae73b083a19a0185f`

Passed 6,519 mock placement, live-settings and pattern checks, upgrade/error diagnostics, actual desktop checkbox and Apply clicks, and the exact external GUI launcher route attached to a separate JVM forced into headless mode. The external test verifies live pattern packets, request limits, reopening with retained settings and automatic closure after End. Live Minecraft/server compatibility remains untested. Source and tests are included in the download.

---

# Fabric 1.21.8 redstone ore agent, version 9

Requires Java 21 with jdk.attach, Minecraft 1.21.8 and Fabric Loader in the intermediary namespace. Fabric API is optional.

This version sends ABORT_DESTROY_BLOCK only, targeting normal and deepslate redstone ore in a 15-block radius. ABORT cancels a mining action; without a preceding START it generally does not hit or activate redstone ore. The agent does not initiate or finish block breaking. Server behavior and acceptance of these requests may vary, and the requested radius does not grant a 15-block interaction reach.

## Upgrade and use from Windows CMD

Restart Minecraft before attaching this update. Extract nuker-package.zip into a new folder and open CMD there. Start Minecraft and enter your world.

```cmd
attach.cmd
```

Find Minecraft's PID, then replace 12345:

```cmd
attach.cmd 12345
```

N toggles requests; End stops the agent until Minecraft restarts. It starts OFF, pauses in menus or when unfocused, and turns OFF on world changes. Switching game mode keeps it enabled.

## Redstone-only requests

Each batch scans a sphere centered on the player's eyes, using block-center distance. The default and maximum radius is 15 blocks. It sends one ABORT_DESTROY_BLOCK request for each normal or deepslate redstone ore block inside that sphere. Both lit and unlit states are included. Other block types are skipped. Redstone ore is recognized by Minecraft's RedstoneOreBlock type, including subclasses.

Only ore visible in the client's loaded world can be found. The agent does not ask the server for unknown block positions or load distant chunks. The requested radius does not bypass server range checks.

No START_DESTROY_BLOCK or STOP_DESTROY_BLOCK packets are sent. The agent does not retain a mining target, advance mining progress, force local block removal, or wait for blocks to disappear. Ore still present is attempted again in the next batch. The delay after each completed batch remains 1000 ms.

The selected tool slot is synchronized before each batch. Requests use Minecraft's sequenced-packet sender. All game calls run on the client thread, with only one pending batch. Stopping prevents new requests but does not recall already-sent packets.

## Configure

Default launcher settings:

```cmd
java --add-modules jdk.attach -jar nuker-agent.jar 12345 nuker-agent.jar "radius=15,budget=0,interval=1000"
```

Radius accepts values greater than 0 and at most 15. Budget 0 covers all matching ores; a positive budget from 1 to 512 limits matching targets per batch, nearest first. Interval accepts 5 to 1000 milliseconds and is the delay after each completed batch. Restart Minecraft to change an attached agent's configuration.

Stop through CMD:

```cmd
attach.cmd 12345 stop
```

Run under the same OS user as Minecraft. Use Java 21 with jdk.attach. If dynamic loading is disabled, add -XX:+EnableDynamicAgentLoading to Minecraft's JVM arguments and restart. Remove -XX:+DisableAttachMechanism if present. Java 21 normally prints a dynamic-agent warning.

## Source and verification

Source and mock tests are in source/. Build on Windows with source\build.cmd or on Linux with source/build.sh. Requires a Java 21 JDK and no external build dependencies. RedstoneOreBlock, block-state access, and packet symbols were checked against FabricMC Yarn 1.21.8. The uploaded client-intermediary.jar exceeded the 32 MiB transfer limit and was not inspected.

source/test.sh attaches to real JVMs with mock Minecraft classes, with and without mock Fabric API. Tests verify normal and deepslate ores, lit states, filtering out other blocks and ores, targets exactly 15 blocks away, exclusion beyond the spherical boundary, negative coordinates, ABORT-only actions throughout every batch and control transition, increasing packet sequences, optional batch limits, retries, 1000 ms scheduling, pause/stop behavior, and no task backlog. Mock tests cannot establish live server acceptance or redstone activation. Live game/server compatibility remains untested.

## Download

Download [nuker-package.zip](https://github.com/Calculus1212/Codex/raw/main/nuker-package.zip), extract it, restart Minecraft, and follow the CMD instructions above.

Package SHA-256: `081e7c1e231740bfd63e2ef79806827775ffbc208087d7065c8e9f4019cc76c5`

Version 9 passed 191 checks across three real JVM attachments using mock Minecraft classes. The compiled agent was also checked to contain ABORT_DESTROY_BLOCK only. Live Minecraft/server compatibility remains untested.
