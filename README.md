# Fabric 1.21.8 Java nuker agent

[Download nuker-package.zip](https://github.com/Calculus1212/Codex/raw/refs/heads/main/nuker-package.zip)

Extract the ZIP on Windows. It contains `nuker-agent.jar`, `attach.cmd`, detailed instructions, and Java source.

Requires Minecraft **1.21.8**, Fabric Loader, and **Java 21** with `jdk.attach`. Fabric API is optional. Instant block breaking requires **creative mode**; the server decides which breaking requests succeed. This cannot force instant survival mining.

Open CMD in the extracted folder. List Minecraft's process ID:

```cmd
java --add-modules jdk.attach -jar nuker-agent.jar --list
```

Replace `12345` with Minecraft's PID and attach:

```cmd
java --add-modules jdk.attach -jar nuker-agent.jar 12345 nuker-agent.jar
```

Press **N** to toggle the nuker; **End** stops it. It starts disabled. See the packaged README for configuration and troubleshooting.

Attachment and behavior were tested against mock game classes with and without Fabric API. Live Minecraft compatibility remains untested.

Package SHA-256: `24156bb22aa43f79df8d84d013400c2fe206a468cbc3525d72eaffa428dd42ae`.
