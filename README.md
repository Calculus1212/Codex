# Fabric 1.21.8 redstone ore agent — version 7

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract into a new folder and **restart Minecraft before attaching this update**. Includes `nuker-agent.jar`, `attach.cmd`, instructions, Java source, and mock runtime tests.

Sends **START_DESTROY_BLOCK only** for normal and deepslate redstone ore, including lit states, inside a **15-block sphere around the player's eyes**. Other blocks are skipped. The delay between completed batches remains **1000 ms**. No STOP_DESTROY_BLOCK or ABORT_DESTROY_BLOCK packets are sent.

Only ore in the client's loaded world can be detected. The server can reject targets outside its permitted reach; this does not grant a 15-block interaction reach. START can still break instantly mineable or creative-mode ore, so this is not guaranteed nonbreaking interaction. The agent does not force local block removal.

Defaults: `radius=15,budget=0,interval=1000`. Budget 0 covers every matching ore. The selected tool slot is synchronized; requests use Minecraft's sequenced-packet sender. There is no mining-progress wait, and ore still present is retried each batch.

Requires **Java 21** with `jdk.attach`, Minecraft **1.21.8**, and Fabric Loader. Fabric API is optional.

Open CMD in the extracted folder:

```cmd
attach.cmd
```

Find Minecraft's PID and replace `12345`:

```cmd
attach.cmd 12345
```

**N** toggles; **End** stops. Starts OFF, pauses in menus or when unfocused, and turns OFF on world changes. Stopping prevents new requests but sends no cancellation packet. See the packaged README for configuration and details.

Mock JVM attachment tests passed with and without Fabric API. Checks include redstone/deepslate filtering, lit states, other ore exclusion, the 15-block boundary, negative coordinates, START-only actions, packet sequences, optional limits, retries, 1000 ms scheduling, pause/stop behavior, and block-type changes between batches. Live Minecraft/server behavior remains untested.

Package SHA-256: `4fc8242836bc5a4fa7d7ed74745480ede6381e3af70db2d9545d29f4cacfad39`.
