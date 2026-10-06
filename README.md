# Fabric 1.21.8 block agent — version 6

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract into a new folder and **restart Minecraft before attaching this update**. The ZIP includes `nuker-agent.jar`, `attach.cmd`, instructions, Java source, and mock runtime tests.

This version sends **START_DESTROY_BLOCK only**, with **1000 ms between batches**. It sends no STOP_DESTROY_BLOCK or ABORT_DESTROY_BLOCK packets. Each batch attempts every non-air block in the radius-6 sphere around the player's eyes, without waiting for progress or block removal.

**START is a block attack request and can still break instantly mineable blocks or creative-mode blocks. This does not guarantee that surrounding blocks stay intact.** Server behavior determines what happens. The agent does not force local block removal.

Defaults: `radius=6,budget=0,interval=1000`. Budget 0 means all targets. Client-side game-mode, reach, and hardness filters are absent; the server still applies its rules. The selected tool slot is synchronized, and requests use Minecraft's sequenced-packet sender.

Requires **Java 21** with `jdk.attach`, Minecraft **1.21.8**, and Fabric Loader. Fabric API is optional.

Open CMD in the extracted folder:

```cmd
attach.cmd
```

Find Minecraft's PID, then replace `12345`:

```cmd
attach.cmd 12345
```

**N** toggles; **End** stops. It starts OFF, pauses in menus or when unfocused, and turns OFF on world changes. Stopping prevents new requests but sends no cancellation packet. See the packaged README for configuration and details.

Attachment tests passed with and without mock Fabric API: START-only actions, no STOP or ABORT, one request per target, packet sequences, full-area coverage, optional limits, retries, 1000 ms scheduling, and pause/stop behavior. Live Minecraft/server behavior remains untested; these mocks cannot verify that blocks will stay intact.

Package SHA-256: `6aa1fed483097ff122d1b6de6ff430359c1d865ec7e13e8439e8832104fe41d2`.
