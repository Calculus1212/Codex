# Replanter for Fabric 1.21.8, version 8

An attachable Java 21 agent with a modeless JOptionPane settings dialog. Configure batch delay, maximum block-use requests per batch, range, and a 7×7 checkbox planting pattern while the game is running. Fabric Loader in the intermediary namespace is required; Fabric API is optional. The package keeps the planter-agent.jar name.

## Windows CMD

Restart Minecraft before upgrading an attached agent. Extract planter-package.zip into its own folder, enter your world, hold the seeds or item you want to use, and open CMD in that folder.

```cmd
attach.cmd
```

Find Minecraft's current PID and replace 12345:

```cmd
attach.cmd 12345 gui
```

Both `attach.cmd 12345` and `attach.cmd 12345 gui` install the module and open settings on first attachment. O in Minecraft or the gui command reopens the current dialog. P toggles interactions. The module starts OFF. End stops it until Minecraft restarts. Use Alt+Tab to move between the separate desktop dialog and Minecraft. Closing or hiding settings leaves the module attached. Settings change live; upgrading the JAR requires a game restart and a new PID.

```cmd
attach.cmd 12345 stop
```

Use Java 21 with jdk.attach under the same OS user as Minecraft. If attachment is disabled, add -XX:+EnableDynamicAgentLoading to Minecraft's JVM arguments, remove -XX:+DisableAttachMechanism if present, and restart. Java 21 normally prints a dynamic-agent warning. Initialization failures show the underlying cause in CMD when available; the complete Attach Listener exception is in the game log or launcher console. An older loaded planter agent must be replaced by restarting Minecraft.

## Settings and the 7×7 pattern

Defaults are 100 ms between completed batches, a maximum of 70 requests per batch, range 8, and pattern mode enabled with all 49 cells checked.

- **Delay between batches:** 20–1000 ms.
- **Maximum blocks per batch:** 1–70 placement attempts.
- **Target range:** greater than 0 and at most 8 blocks, measured from the eyes to the support's top-center.
- **Use checked cells:** enables the 7×7 pattern. Uncheck it to restore the full-radius scan; the edited pattern is retained.

Each checkbox is one support block. The highlighted center is your current player block. Grid rows run north (−Z) to south (+Z); columns run west (−X) to east (+X). Top-left is offset X=−3, Z=−3, and bottom-right is X=+3, Z=+3. The center cell is independently selectable. The pattern moves with the player and keeps its compass orientation when you turn. It covers offsets −3 through +3 from floor(playerX), floor(playerZ); fractional movement does not shift the grid until a player block boundary is crossed.

Click cells to design the planting shape, or use **Select all** and **Clear all**. Click **Apply settings** to commit both the pattern and numeric settings. Editing checkboxes or loading defaults alone does not change the running module. Clearing every cell and applying sends no requests while pattern mode is enabled. Reopening shows the applied settings and pattern. Values last for the current Minecraft session and are not saved to disk.

Changing the pattern, toggling pattern mode or changing range restarts the nearest-first scan on the next active batch. Changing only delay or the request limit preserves scan progress. Each batch continues after the last examined target, including skipped cells, until the whole selected pattern is examined. It restarts nearest-first on a later batch, never halfway through the last batch of a pass. Cells outside the configured range are skipped even if checked. With all 49 cells selected, a complete pattern pass can contain at most 49 distinct requests, and fewer when some cells are ineligible; a request limit of 70 does not repeat cells to fill the batch.

A running batch keeps its original settings. A new delay affects the next scheduled wait; an already pending wait finishes first. The wait follows the completed client-thread task, so this is not an exact wall-clock rate. GUI Apply sets the retry cooldown to the selected delay. Only one task can remain queued, preventing a backlog. **Defaults** loads the original settings and all selected cells; click Apply to commit. The dialog can scroll on smaller displays.

## Placement behavior

The module sends MAIN_HAND, UP-face PlayerInteractBlockC2SPacket requests using any nonempty selected main-hand item, including XP Seeds, other seeds, building blocks or tools. It does not switch hotbar slots or refill inventory. Empty hands, menus and lost Minecraft focus pause requests and preserve scan progress. Editing the desktop dialog pauses requests while Minecraft is unfocused. World changes turn the module OFF. Crossing an X/Z block boundary, changing support Y, or toggling P resets the sweep.

Only the support layer immediately below the player's actual feet is considered: floor(nextDown(playerY)). Feet at Y=64 use support Y=63. Standing on farmland at 63.9375 or a slab at 63.5 also selects Y=63; negative coordinates work the same way. Hit height uses the support's collision-shape top. No other support layers are targeted.

Eligible supports must be non-air, have a nonempty collision shape with its top within its block cell, and have air directly above. Water, non-colliding supports and occupied placement spaces are skipped. The selected hotbar slot is synchronized before each batch. Requests use Minecraft's private sequenced sender and run on the client thread. The entire eligible batch is sent without per-cell delay or waiting for placement acknowledgements. No digging packets, local placement or inventory changes are fabricated.

These are interaction requests, not guaranteed successful placements. The server controls reach, held-item behavior, valid soil, collisions and inventory consumption. The agent does not till soil or automatically harvest existing crops. Prepare compatible soil and hold XP Seeds to plant them; cells already occupied by crops are skipped. An 8-block configured range does not extend the server's permitted reach. An arbitrary held item can use a block or open its interface rather than place. Stopping prevents new requests but cannot recall interactions already sent.

## Command-line configuration

```cmd
java --add-modules jdk.attach -jar planter-agent.jar 12345 planter-agent.jar "radius=8,budget=70,interval=100,retry=100,pattern=on"
```

Pattern defaults to on with all cells selected. `pattern=off` restores the previous circular range scan. Custom checkbox shapes are configured through the dialog. Budget 0 also means 70; larger values are rejected. Retry accepts values from interval through 10000 ms; if omitted it defaults to at least interval. The old item option is unsupported.

## Source and verification

Source, build scripts and game mocks are included in source/. Rebuild with a Java 21 JDK using source\build.cmd on Windows or source/build.sh on Linux. No external build dependencies are required.

Run source/test.sh on Linux with Java 21. It verifies actual JVM attachment against mocked Minecraft classes, GUI-first installation, live settings changes, scheduler timing, full-radius sweep continuation, upgrade/error diagnostics, and packet-level 7×7 pattern behavior. Pattern checks cover all 49 cells, asymmetric selections, corners and center, budget continuation, pattern resets, changed delay/limit retaining progress, fractional movement, negative coordinates, empty selections, empty hands, occupancy and GUI restoration. The optional planter.WindowFixture desktop test verifies actual checkbox and Apply clicks, reopening and disposal; it requires a display or Xvfb.

Minecraft intermediary symbols were checked against FabricMC Yarn 1.21.8. Mock tests and desktop smoke tests do not establish live server acceptance. Live Minecraft/server compatibility remains untested. Uploaded Erazion JARs are not included in this package; their custom items still require the appropriate game mod.
