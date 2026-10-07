# General top placement module for Fabric 1.21.8, version 3

An attachable Java 21 agent that sends top-face block-use requests using any nonempty item in your selected main hand. It targets only the block layer immediately below your feet. Fabric Loader in the intermediary namespace is required; Fabric API is optional. The package keeps the existing planter-agent.jar name.

## Windows CMD

Restart Minecraft before attaching this update. Extract planter-package.zip into its own folder. Enter your world, select the item or block you want to use in your hotbar, and open CMD in the extracted folder.

```cmd
attach.cmd
```

Find Minecraft's PID and replace 12345:

```cmd
attach.cmd 12345
```

Press P to enable or disable interactions. It starts OFF. End stops the agent until Minecraft restarts. Menus and lost focus pause requests, and switching worlds turns the module OFF. To stop through CMD:

```cmd
attach.cmd 12345 stop
```

Use Java 21 with jdk.attach under the same OS user as Minecraft. If dynamic attachment is disabled, add -XX:+EnableDynamicAgentLoading to Minecraft's JVM arguments and restart. Remove -XX:+DisableAttachMechanism if present. Java 21 normally prints a dynamic-agent warning. A Java 21 JDK is required to rebuild source.

## Placement behavior

There is no seed, item ID, or farmland restriction. Any nonempty main-hand stack can be used, including building blocks, XP Seeds and other crops or items. The module does not switch hotbar slots or refill your hand. Empty stacks pause requests.

Only one support Y layer is scanned: the integer coordinate immediately below the player's actual feet position, calculated as floor(nextDown(playerY)). Feet at Y=64 select support blocks at Y=63. Standing on farmland at feet Y=63.9375 also selects Y=63; a slab at feet Y=63.5 selects Y=63. Negative coordinates work the same way. The layer updates when the player moves up or down. No support blocks on other Y layers are used, even if they are nearby.

Eligible supports must be non-air blocks with a nonempty collision shape whose top lies within their block cell, and the block immediately above must be air. Occupied placement spaces and non-colliding supports such as water are skipped. The packet uses MAIN_HAND, the UP face and the support's collision-shape top-center height, including the lowered top of farmland and slabs. Hit positions must be within 5 blocks of the player's eyes. The selected hotbar slot is synchronized before each batch.

The default and maximum batch size is 70 block-use requests. A whole eligible batch is sent in one client-thread task with no per-target delay and no waiting for placement acknowledgements. The delay after each completed batch stays 100 ms. Unacknowledged positions can be retried in the next 100 ms batch. The module does not reserve inventory or cap requests to the displayed item count. Already placed blocks above supports are skipped as soon as updates reach the client. Only one task can remain queued, preventing a backlog after a busy frame.

Every target still requires a separate PlayerInteractBlockC2SPacket. The server applies the held item's normal right-click behavior: a building block may place, seeds may plant on compatible soil, a tool may act on the support, and an interactable block may open its interface. Holding an arbitrary item does not turn it into a placeable block. Prepare soil first when using seeds. This module does not automatically sneak to override container interaction.

The server controls inventory consumption, collision checks, reach, custom item behavior and whether requests succeed. Up to 70 requests does not guarantee 70 placed blocks when fewer eligible positions or items are available. The agent does not fabricate local blocks or inventory changes and sends no digging actions. Stopping prevents new requests but cannot undo already-sent interactions. Live game/server compatibility remains untested.

## Configuration

Defaults:

```cmd
java --add-modules jdk.attach -jar planter-agent.jar 12345 planter-agent.jar "radius=5,budget=70,interval=100,retry=100"
```

Radius accepts values greater than 0 and at most 5. Budget defaults to 70; 1..70 limits requests per batch and 0 also means 70. Values over 70 are rejected. Interval accepts 20..1000 ms; retry accepts interval through 10000 ms. The old item option has been removed. Restart Minecraft to change configuration or upgrade an attached agent.

## Source and verification

Generated source and game mocks are in source/. Build on Windows with source\build.cmd or Linux with source/build.sh. Run source/test.sh on Linux with Java 21 to verify actual JVM attachment against mocked game classes. No external build dependencies are needed.

Tests cover a 70-request burst, 100 ms repeats, arbitrary blocks/seeds/items, empty hand, occupied spaces, non-colliding supports, the single feet-Y layer, standing on farmland/slabs, fractional and integer negative Y, moving between layers, exact 5-block range boundaries, packet hand/face/top heights/sequences, mock server inventory consumption and placement updates, refill, rebuilding, menus, focus, world changes, task backlog, P/End and CMD stop.

Packet, hit-result, entity feet position, block-state collision shape, voxel shape and inventory symbols were checked against FabricMC Yarn 1.21.8 mappings. Mock attachment tests do not establish live server acceptance or actual placement behavior. The uploaded Erazion JAR is not included or required by the module itself; Erazion items still require Erazion in your game.

Package SHA-256: `a451a830dd34651157f9fc39b28a54380fc68057e621aaa58cfe6935ca6ff021`

2393 checks passed across two actual JVM attachments using mocked Minecraft classes, including arbitrary held items, support layers and CMD stop.
