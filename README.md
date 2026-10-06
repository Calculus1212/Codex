# Fabric 1.21.8 Java nuker agent — version 4

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract the ZIP into a new folder. **Restart Minecraft before attaching this update.** It contains `nuker-agent.jar`, `attach.cmd`, instructions, and Java source.

Version 4 sends a **START/STOP break-request pair for every non-air block in a radius-6 sphere**, then immediately moves to the next block. It does not wait for normal mining progress or for a block to disappear. Remaining blocks are attempted again in the next batch. The delay after each completed batch is **100 ms**.

The default `budget=0` means all surrounding non-air blocks are attempted. Client-side game-mode, interaction-reach, hardness, and per-cycle block filters have been removed. The configured radius defines the area. Servers still decide whether requests succeed; early or out-of-reach requests can be rejected, and some survival blocks may remain unbroken. The agent does not force local block removal.

Requests use Minecraft's sequenced-packet sender and synchronize the selected tool slot first. One client-thread batch may be pending at a time. Large batches increase client, network, and server load.

Requires Minecraft **1.21.8**, Fabric Loader, and **Java 21** with `jdk.attach`. Fabric API is optional.

Open CMD in the extracted folder:

```cmd
attach.cmd
```

Find Minecraft's process ID and replace `12345`:

```cmd
attach.cmd 12345
```

**N** toggles; **End** stops. It starts disabled and pauses in menus or when unfocused. World changes turn it off. Defaults: `radius=6,budget=0,interval=100`. A positive budget optionally limits requests per batch. See the packaged README for details.

Real JVM attachment passed mock tests with and without Fabric API. The mock deliberately leaves all blocks unbroken, verifying full-area requests and retries without waiting, START/STOP ordering, packet sequences, optional limits, selected-tool synchronization, 100 ms scheduling, pause/stop behavior, and avoiding queued bursts. Live Minecraft/server compatibility remains untested.

Package SHA-256: `509fba07061700835d4521f7e5ca643b85bd1ecbe2ae77697453a1557646642d`.
