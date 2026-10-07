# General top placement module for Fabric 1.21.8, version 6

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

A separate desktop settings window opens automatically after attachment. Use Alt+Tab to switch between Minecraft and this window. Press O in the game to reopen it, or run `attach.cmd 12345 gui`.

Press P to enable or disable interactions. It starts OFF. End stops the agent until Minecraft restarts. Menus and lost focus pause requests, and switching worlds turns the module OFF. To stop through CMD:

```cmd
attach.cmd 12345 stop
```

Use Java 21 with jdk.attach under the same OS user as Minecraft. If dynamic attachment is disabled, add -XX:+EnableDynamicAgentLoading to Minecraft's JVM arguments and restart. Remove -XX:+DisableAttachMechanism if present. Java 21 normally prints a dynamic-agent warning. A Java 21 JDK is required to rebuild source.

## Placement behavior

There is no seed, item ID, or farmland restriction. Any nonempty main-hand stack can be used, including building blocks, XP Seeds and other crops or items. The module does not switch hotbar slots or refill your hand. Empty stacks pause requests.

Only one support Y layer is scanned: the integer coordinate immediately below the player's actual feet position, calculated as floor(nextDown(playerY)). Feet at Y=64 select support blocks at Y=63. Standing on farmland at feet Y=63.9375 also selects Y=63; a slab at feet Y=63.5 selects Y=63. Negative coordinates work the same way. The layer updates when the player moves up or down. No support blocks on other Y layers are used, even if they are nearby.

Eligible supports must be non-air blocks with a nonempty collision shape whose top lies within their block cell, and the block immediately above must be air. Occupied placement spaces and non-colliding supports such as water are skipped. The packet uses MAIN_HAND, the UP face and the support's collision-shape top-center height, including the lowered top of farmland and slabs. Hit positions must be within the configured range (8 blocks by default) of the player's eyes. The selected hotbar slot is synchronized before each batch.

The default and maximum batch size is 70 block-use requests. A whole eligible batch is sent in one client-thread task with no per-target delay and no waiting for placement acknowledgements. The default delay after each completed batch is 100 ms and can be changed in the settings window. The module remembers its position in the nearest-first target list: each batch continues after the last examined target, including skipped positions, until the full configured area has been scanned. Failed nearby requests cannot prevent farther targets from being attempted. The last batch of a pass may contain fewer than 70 requests. The nearest-first list restarts on the following batch after a complete pass, never midway through the last batch.

Menus, lost focus and an empty hand pause the scan without advancing or resetting its cursor. Fractional movement inside the same X/Z player block also retains progress; each hit is still checked against your current eye position and configured radius. Crossing into a different X/Z block, changing the support Y layer, toggling the module or switching worlds resets the sweep for the new area. The target list is rebuilt at the start of each pass, and block/occupancy data is checked live as each position is visited.

The default retry cooldown is 100 ms, but a position is revisited only after the current full sweep finishes. The module does not reserve inventory or cap requests to the displayed item count. Already placed blocks above supports are skipped as soon as updates reach the client. Only one task can remain queued, preventing a backlog after a busy frame.

Every target still requires a separate PlayerInteractBlockC2SPacket. The server applies the held item's normal right-click behavior: a building block may place, seeds may plant on compatible soil, a tool may act on the support, and an interactable block may open its interface. Holding an arbitrary item does not turn it into a placeable block. Prepare soil first when using seeds. This module does not automatically sneak to override container interaction.

The server controls inventory consumption, collision checks, reach, custom item behavior and whether requests succeed. Up to 70 requests does not guarantee 70 placed blocks when fewer eligible positions or items are available. The agent does not fabricate local blocks or inventory changes and sends no digging actions. Stopping prevents new requests but cannot undo already-sent interactions. Live game/server compatibility remains untested.

## Live settings window

The window provides three controls:

- **Delay between batches:** 20–1000 ms; default 100 ms.
- **Maximum requests per batch:** 1–70; default 70.
- **Target range:** greater than 0 and at most 8 blocks; default 8.

Edit the values and click **Apply settings** to update the running module. No restart or reattachment is needed for settings changes. A batch already running uses its original settings. The delay applies to the next scheduled batch; an already pending wait finishes first. The delay follows a completed batch, rather than promising an exact wall-clock rate.

Changing delay or request count keeps the current sweep position. Changing range restarts the nearest-first scan on the next active batch. Applying settings does not enable the module: P remains its toggle. GUI changes also set the retry cooldown to the selected batch delay.

**Defaults** fills in 100 ms, 70 requests and 8 blocks; click **Apply settings** to commit. **Hide** and the window close button hide the window while keeping the module attached. O or `attach.cmd 12345 gui` reopens it with current values. End or CMD stop disposes the window. Requests pause while Minecraft is unfocused, including while you edit the separate settings window. Values last for the current Minecraft session and are not saved to disk.

The desktop window requires Java's desktop/Swing support. A headless runtime retains command-line settings. Minecraft's normal Windows Java 21 runtime includes desktop support.

## Command-line configuration

Defaults:

```cmd
java --add-modules jdk.attach -jar planter-agent.jar 12345 planter-agent.jar "radius=8,budget=70,interval=100,retry=100"
```

Radius accepts values greater than 0 and at most 8. Budget defaults to 70; 1..70 limits requests per batch and 0 also means 70. Values over 70 are rejected. Interval accepts 20..1000 ms; retry accepts interval through 10000 ms. If retry is omitted it defaults to at least the selected interval. The old item option has been removed. Restart Minecraft to upgrade an attached agent; use the GUI to change settings live.

## Source and verification

Generated source and game mocks are in source/. Build on Windows with source\build.cmd or Linux with source/build.sh. Run source/test.sh on Linux with Java 21 to verify actual JVM attachment against mocked game classes. No external build dependencies are needed. The optional `planter.WindowFixture` desktop smoke test requires a display or Xvfb and verifies an actual Apply click, hide, reopen and disposal.

Tests cover live Apply changes to all three settings, scheduler delay changes, cursor retention and range reset, invalid input, defaults, CMD GUI reopening, stopped-agent rejection, and a 70-request burst, 100 ms continuation batches, full area coverage before wraparound, rejected requests without starvation, retained progress across pauses/refills/fractional movement, resets on block movement, arbitrary blocks/seeds/items, empty hand, occupied spaces, non-colliding supports, the single feet-Y layer, standing on farmland/slabs, fractional and integer negative Y, moving between layers, exact 8-block range boundaries and targets beyond the old 5-block limit, packet hand/face/top heights/sequences, mock server inventory consumption and placement updates, refill, rebuilding, menus, focus, world changes, task backlog, P/End and CMD stop.

Packet, hit-result, entity feet position, block-state collision shape, voxel shape and inventory symbols were checked against FabricMC Yarn 1.21.8 mappings. Mock attachment tests do not establish live server acceptance or actual placement behavior. The uploaded Erazion JAR is not included or required by the module itself; Erazion items still require Erazion in your game.
