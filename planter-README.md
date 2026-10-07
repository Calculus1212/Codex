# XP crop planter for Fabric 1.21.8, version 1

An attachable Java 21 agent that plants Erazion XP Seeds on empty farmland using PlayerInteractBlockC2SPacket requests. Fabric Loader in the intermediary namespace is required; Fabric API is optional.

## Windows CMD

Restart Minecraft before switching from an older agent. Extract planter-package.zip into its own folder. Enter your world, put XP Seeds in your selected hotbar slot, and open CMD in the extracted folder.

```cmd
attach.cmd
```

Find Minecraft's PID and replace 12345:

```cmd
attach.cmd 12345
```

Press P in Minecraft to enable or disable planting. It starts OFF. End stops it until Minecraft restarts. Menus and lost focus pause requests, and switching worlds turns planting OFF. To stop through CMD:

```cmd
attach.cmd 12345 stop
```

Use Java 21 with jdk.attach under the same OS user as Minecraft. If dynamic attachment is disabled, add -XX:+EnableDynamicAgentLoading to Minecraft's JVM arguments and restart. Remove -XX:+DisableAttachMechanism if present. Java 21 normally prints a dynamic-agent warning. You need a Java 21 JDK to rebuild source.

## Planting behavior

The required held item defaults to erazion:experience_seeds. The uploaded Erazion JAR targets Minecraft 1.21.8 and contains this seed's assets and translation key item.erazion.experience_seeds, displayed as Exp Seeds in English and Graine d'XP in French. Item registration code is obfuscated; the agent compares the live registry ID and logs the held and required IDs when P is pressed. A different display language does not affect matching.

The module uses only the selected main-hand stack. Other crops and empty stacks pause planting. It does not switch slots or refill your hand. Prepare soil with a hoe first: only minecraft:farmland is eligible, and the block immediately above it must be air. Existing crops, water, other occupied blocks, dirt and grass are skipped. Moisture, lighting and other Erazion growth/placement rules remain the server's responsibility.

It scans a sphere extending up to 5 blocks from the player's eyes to the top-center hit point of each farmland block, including uneven ground and negative coordinates. This measures interaction distance, not a square or a flat 5-block horizontal disk. Each request uses MAIN_HAND, the UP face, a hit at x+0.5/y+0.9375/z+0.5, and Minecraft's own interaction sequence sender. The selected hotbar slot is synchronized before each batch.

Every eligible target in a batch is requested in one client-thread task, nearest first, without waiting between targets or waiting for each crop to appear. Each block still needs its own packet and the server processes requests individually. A batch is limited by the available seed count and an optional target budget. No digging packets are sent, and the agent does not fabricate local crops or inventory changes.

The default delay after each completed batch is 100 ms. Pending requests reserve seeds and are not repeated for 1000 ms. Reservations release when a crop appears, the ground changes, the target leaves range, or the retry timeout expires. This allows bursts without repeatedly spending the same apparent inventory while server updates are pending. The scheduler keeps only one queued task, avoiding a backlog after a busy frame.

The server decides whether each request succeeds and can reject targets outside its allowed reach or under its custom planting rules. A requested 5-block scan does not increase server reach. Stopping prevents new requests but cannot undo ones already sent.

## Configuration

Defaults:

```cmd
java --add-modules jdk.attach -jar planter-agent.jar 12345 planter-agent.jar "radius=5,budget=0,interval=100,retry=1000,item=erazion:experience_seeds"
```

Radius accepts values greater than 0 and at most 5. Budget 0 sends all eligible targets allowed by held seed count; 1..512 limits targets per batch. Interval accepts 20..1000 ms; retry accepts values from interval through 10000 ms. Item accepts a namespaced registry ID. Changing configuration requires restarting Minecraft and attaching again. P logs the currently held item ID in Minecraft's latest.log for diagnosing mismatches.

## Source and verification

Generated source and mock tests are in source/. Build on Windows with source\build.cmd or Linux with source/build.sh; no external build dependencies are needed. Run source/test.sh on Linux with Java 21 to verify actual JVM attachment against mocked game classes.

Tests cover a 64-target batch, custom seed ID filtering, bare dirt and occupied-soil exclusion, held inventory limits, pending request reservations, placement acknowledgements, refill and replanting, retry timing, exact 5-block boundaries and negative coordinates, packet hand/face/hit positions/sequences, 100 ms scheduling, menus, focus, world changes, task backlog, P/End controls and CMD stop.

Packet, hit-result, registry, inventory and block-state symbols were checked against FabricMC Yarn 1.21.8 mappings. The uploaded Erazion JAR was inspected for metadata, assets and translation keys; it is not included in this package. Runtime tests use mocks and cannot establish live Erazion server acceptance or planting behavior. Live game/server compatibility remains untested.

Package SHA-256: `1481736ae929d6390c0e126172b8d397d8adf9865925fd117e23bb51b22d17cf`

623 checks passed across two actual JVM attachment runs using mock Minecraft classes, including CMD stop.
