# Fabric 1.21.8 redstone ore agent, version 8

Requires Java 21 with jdk.attach, Minecraft 1.21.8 and Fabric Loader in the intermediary namespace. Fabric API is optional.

This version sends START_DESTROY_BLOCK immediately followed by ABORT_DESTROY_BLOCK to the same position and face, targeting normal and deepslate redstone ore in a 15-block radius. START is a block attack request and can still break instantly mineable or creative-mode blocks. The server can reject targets beyond its permitted reach. These requests do not grant a 15-block interaction reach or guarantee that ore stays intact.

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

Each batch scans a sphere centered on the player's eyes, using block-center distance. The default and maximum radius is 15 blocks. It sends one START_DESTROY_BLOCK request followed immediately by one ABORT_DESTROY_BLOCK request for each normal or deepslate redstone ore block inside that sphere. Both lit and unlit states are included. Other block types are skipped. Redstone ore is recognized by Minecraft's RedstoneOreBlock type, including subclasses.

Only ore visible in the client's loaded world can be found. The agent does not ask the server for unknown block positions or load distant chunks. The requested radius does not bypass server range checks.

No STOP_DESTROY_BLOCK packets are sent. There is no delay or block-state check between START and ABORT; cancellation is attempted immediately before moving to the next ore. The agent does not retain a mining target, advance mining progress, force local block removal, or wait for blocks to disappear. Ore still present is attempted again in the next batch. The delay after each completed batch remains 1000 ms.

The selected tool slot is synchronized before each batch. Requests use Minecraft's sequenced-packet sender. All game calls run on the client thread, with only one pending batch. Stopping prevents new requests but does not recall already-sent packets or send any additional cancellation packet beyond each normal START/ABORT pair.

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

source/test.sh attaches to real JVMs with mock Minecraft classes, with and without mock Fabric API. Tests verify normal and deepslate ores, lit states, filtering out other blocks and ores, targets exactly 15 blocks away, exclusion beyond the spherical boundary, negative coordinates, adjacent START/ABORT pairs on the same position and face, increasing packet sequences, optional batch limits, retries, 1000 ms scheduling, pause/stop behavior, and no task backlog. Mock tests cannot establish live server acceptance or nonbreaking behavior. Live game/server compatibility remains untested.

## Download

Download [nuker-package.zip](https://github.com/Calculus1212/Codex/raw/main/nuker-package.zip), extract it, restart Minecraft, and follow the CMD instructions above.

Package SHA-256: `e3e8b58ba7bfbd76c8e7775dad852ea4afd782cc815aa9b66a426a87449bef08`

Version 8 passed 120 checks across three real JVM attachments using mock Minecraft classes. Live Minecraft/server compatibility remains untested.
