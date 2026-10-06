# Fabric 1.21.8 Java nuker agent — version 3

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract the ZIP into a new folder. **Restart Minecraft before attaching this update.** The archive contains `nuker-agent.jar`, `attach.cmd`, instructions, and Java source.

Version 3 sets the default scheduling interval to **100 ms** and removes the creative-only requirement. Requested radius remains **6 blocks**, capped by the player's actual interaction range, and the budget remains **128 attempts per cycle**.

In creative mode, the agent processes block breaks in batches. In survival and other noncreative modes, it retains a target and advances normal mining progress across cycles. Tools, hardness, cooldowns, game permissions, and server rules still apply. Hard blocks take multiple cycles; the 100 ms setting can mine them more slowly than manual mining every game tick. This does not grant instant survival breaking.

Requires Minecraft **1.21.8**, Fabric Loader, and **Java 21** with `jdk.attach`. Fabric API is optional.

Open CMD in the extracted folder and list Java processes:

```cmd
attach.cmd
```

Replace `12345` with Minecraft's PID:

```cmd
attach.cmd 12345
```

Press **N** to toggle the nuker; **End** stops it. It starts disabled. The launcher uses `radius=6,budget=128,interval=100`. Switching game mode keeps it enabled. Partial survival mining is cancelled when paused or stopped. See the packaged README for details and configuration.

Attachment and behavior passed mock runtime tests with and without Fabric API, including survival completion, pause/resume/stop handling, the 100 ms interval, creative batches, player reach, and avoiding a task backlog. Live Minecraft performance remains untested.

Package SHA-256: `8a28860117897a96713f2408c5abe8e9a1615987df6e328a98b1fdb3252aeec0`.
