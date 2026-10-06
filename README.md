# Fabric 1.21.8 Java nuker agent — version 2

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract the ZIP into a new folder. **Restart Minecraft before attaching this update.** The archive contains `nuker-agent.jar`, `attach.cmd`, detailed instructions, and Java source.

Version 2 increases the requested radius from **3 to 6 blocks**, the breaking budget from **8 to 128 attempts per cycle**, and reduces the scheduling interval from approximately **50 ms to 10 ms**. The radius is capped by the player's actual interaction range; client-thread availability and server processing determine actual speed.

Requires Minecraft **1.21.8**, Fabric Loader, and **Java 21** with `jdk.attach`. Fabric API is optional. Instant block breaking requires **creative mode**. The server decides which breaking requests succeed; this cannot force instant survival mining.

Open CMD in the extracted folder and list Java processes:

```cmd
attach.cmd
```

Replace `12345` with Minecraft's PID:

```cmd
attach.cmd 12345
```

Press **N** to toggle the nuker; **End** stops it. It starts disabled. The launcher uses `radius=6,budget=128,interval=10`. See the packaged README for configuration and troubleshooting.

Attachment and behavior passed tests against mock game classes, with and without Fabric API, including 128 removals per cycle, the larger radius, cooldown restoration, and avoiding a task backlog. Live Minecraft or server performance remains untested.

Package SHA-256: `1734f950354ea876e7583b01198d1da31b15ef2d400ff75027a6d7b14c34dd72`.
