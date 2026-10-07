# Minecraft agent downloads

## General top placement module — Fabric 1.21.8, version 7

[Download planter-package.zip](https://github.com/Calculus1212/Codex/raw/main/planter-package.zip) | [Full placement instructions](planter-README.md)

A desktop settings window opens after attachment. Configure batch delay (20–1000 ms), requests per batch (1–70), and target range (greater than 0, up to 8 blocks), then click Apply settings. Defaults remain 100 ms, 70 requests and 8 blocks. O in Minecraft or `attach.cmd PID gui` reopens the window. The gui command now also installs the default module on first attachment. Hide or close leaves the agent attached; End stops it.

Uses any nonempty main-hand item on top faces of blocks immediately below your feet. Each batch continues where the previous batch stopped, restarting nearest-first only after scanning the full area. Delay and request changes preserve progress; range changes restart the scan. Occupied spaces above supports are skipped.

Restart Minecraft to upgrade, extract the package, attach from CMD, then press P to enable. Menus, empty hands and lost focus pause requests. Crossing into a different player block or support Y layer resets the scan. Settings can be edited live without restarting and last for the current session.

The server controls reach and whether requests place a block, plant a crop or perform another interaction. A burst of 70 requests does not guarantee 70 placements.

Package SHA-256: `5733fa8e7d412aabfca4440ac25ae04573c1f004a40a0627425e706ff5017bc1`

Added tests for GUI-first installation, reopening, old-agent upgrade detection, and target-side error reporting in CMD. Passed 6328 checks across actual JVM attachments with mocked game classes, plus a desktop window smoke test with an actual Apply click, hide, reopen and disposal. Live game/server compatibility remains untested.

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