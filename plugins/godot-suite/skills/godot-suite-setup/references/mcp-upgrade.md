# MCP upgrade: runtime capabilities

The suite defaults to `@coding-solo/godot-mcp` (stable). To get runtime
inspection, in-game screenshots, and input simulation, switch to a
runtime-capable server:

- mkdevkit/godot-mcp — https://github.com/mkdevkit/godot-mcp
- DaRealDaHoodie/Claude-GoDot-MCP — https://github.com/DaRealDaHoodie/Claude-GoDot-MCP

To switch: edit `plugins/godot-suite/.mcp.json` (or your project `.mcp.json`) to
point `mcpServers.godot` at the chosen server per its README, then restart Claude
Code and verify with `/mcp` (green dot, tools > 0).
