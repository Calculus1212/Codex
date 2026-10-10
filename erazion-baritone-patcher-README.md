# Erazion + Baritone offline patcher

This tool creates erazion-with-baritone.jar from the exact original erazion.jar and baritone-api-fabric-1.15.0.jar supplied for this patch. It embeds Baritone in META-INF/jars and adds its entry to Erazion's root fabric.mod.json. Existing Erazion classes and bundled libraries, including the Fabric crash-report module, are preserved. The Baritone archive is embedded byte-for-byte, including its nether-pathfinder dependency.

## Windows

1. Extract this ZIP into a new folder.
2. Put your original erazion.jar and baritone-api-fabric-1.15.0.jar in the same folder.
3. Run patch.cmd, or open CMD in that folder and run `java -jar erazion-baritone-patcher.jar`.
4. Close Minecraft. Back up the installed erazion.jar, then replace it with erazion-with-baritone.jar renamed to erazion.jar.
5. Restart Erazion. Avoid installing a second copy of Baritone alongside the embedded copy.

Requires Java 21, already required by Minecraft 1.21.8. This edits archives offline; it does not launch mods, change your installed game automatically, or use the network. It creates a separate output and refuses to overwrite existing files. Hash checks prevent accidentally applying this patch to a different build. A launcher that restores its managed erazion.jar may replace the modified copy.

Expected inputs:

- Erazion: 28,097,175 bytes; SHA-256 bc68ffa29c5e4c098064a1fd0ff339786ff9a351a670cd870c3bc3a85eb8f336.
- Baritone: 1,580,784 bytes; SHA-256 c58ef35a133b6ffce96a74682138ac2ee818cbc063b7c62671db9f9d7d783ebb.

Baritone's supplied metadata explicitly supports Minecraft 1.21.6, 1.21.7 and 1.21.8. Archive structure, nested-mod declarations and unchanged contents were verified. Live Erazion/Baritone startup has not been tested.

No Erazion or Baritone binaries are redistributed in this download. Java source is included in source/. Optional CLI paths: `java -jar erazion-baritone-patcher.jar INPUT_ERAZION INPUT_BARITONE OUTPUT`.

Rebuild with a Java 21 JDK: `javac --release 21 -d build source/EmbedBaritone.java`, then `jar --create --file erazion-baritone-patcher.jar --main-class EmbedBaritone -C build .`.
