# FusionMCP

Control Autodesk Fusion 360 with natural language through Claude. FusionMCP is an [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) server that gives Claude direct access to Fusion 360's CAD engine -- sketching, modeling, assemblies, export, and more.

## How It Works

```
Claude Desktop  ──stdio──▶  MCP Server (Python)  ──HTTP──▶  Fusion 360 Add-in
                             fusion_server.py                 FusionMCP.py
                             (your machine)                   (inside Fusion 360)
```

1. **MCP Server** (`mcp_server/fusion_server.py`) — Runs locally and exposes tools to Claude via the Model Context Protocol over stdio.
2. **Fusion 360 Add-in** (`FusionMCP.py`) — Runs inside Fusion 360, listens on `http://127.0.0.1:7432`, receives commands from the MCP server, and executes them on Fusion's main thread using the Fusion 360 Python API.

## Features

### Sketching
- Primitives: rectangle, center rectangle, circle, line, arc, polygon, ellipse, spline, slot, text
- Operations: fillet, offset, mirror, rectangular pattern
- Constraints: coincident, tangent, perpendicular, parallel, horizontal, vertical, concentric, equal, midpoint, fix, collinear, smooth
- Dimensions: distance, angle, diameter, radius

### 3D Modeling
- Extrude, revolve, loft, sweep, helix, pipe
- Shell, fillet, chamfer, draft, thread
- Press/pull faces, thicken sketches
- Boolean operations (join, cut, intersect)
- Move, rotate, scale (uniform and non-uniform), copy, mirror
- Rectangular and circular body patterns
- Hole features (simple, counterbore, countersink)

### Assembly
- Create components, move bodies between components
- Joints: rigid, revolute, slider, cylindrical, pin-slot, planar, ball
- As-built joints

### Inspection
- Design info (components, sketches, bodies, parameters, joints)
- Face/edge/sketch info for targeted operations
- Body measurement (bounding box, volume, face/edge/vertex counts)
- Distance measurement between entities
- Feature timeline

### Parameters & Appearance
- Create and update user parameters
- Apply material appearances from Fusion's library
- Set custom RGB colors on bodies

### Export & Capture
- STL, STEP, 3MF, F3D (Fusion archive)
- Viewport screenshot capture

### Other
- Execute arbitrary Python scripts inside Fusion 360
- Create/clear documents
- Undo/redo
- Save/save as

## Prerequisites

- [Autodesk Fusion 360](https://www.autodesk.com/products/fusion-360) (any license tier)
- [Python 3.10+](https://www.python.org/downloads/)
- [Claude Desktop](https://claude.ai/download)

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/Anonimus124/fusion-mcp.git
cd fusion-mcp
```

### 2. Install Python Dependencies

```bash
pip install -r mcp_server/requirements.txt
```

This installs:
- `mcp` — Model Context Protocol SDK
- `requests` — HTTP client for communicating with the Fusion add-in

### 3. Install the Fusion 360 Add-in

Create a folder named `FusionMCP` inside Fusion 360's add-ins directory, then copy **both** `FusionMCP.py` and `FusionMCP.manifest` into it.

**Windows:**
```
%APPDATA%\Autodesk\Autodesk Fusion 360\API\AddIns\FusionMCP\
```

**macOS:**
```
~/Library/Application Support/Autodesk/Autodesk Fusion 360/API/AddIns/FusionMCP/
```

The folder must be named exactly `FusionMCP` and contain both files — Fusion 360 will not recognize the add-in without the manifest.

#### Start the Add-in in Fusion 360

1. Open Fusion 360
2. Go to the **Utilities** tab (or **Tools** depending on your version)
3. Click **Add-Ins** (or press `Shift+S`)
4. In the Add-Ins dialog, go to the **Add-Ins** tab
5. Click the green **+** icon next to "My Add-Ins" and browse to the `FusionMCP` folder
6. Select **FusionMCP** and click **Run**
7. A dialog will confirm the bridge is running on port 7432

To auto-start the add-in with Fusion 360, check **Run on Startup** in the Add-Ins dialog.

### 4. Configure Claude Desktop

Open your Claude Desktop configuration file:

**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

Add the FusionMCP server:

```json
{
  "mcpServers": {
    "fusion360": {
      "command": "python",
      "args": ["C:/path/to/fusion-mcp/mcp_server/fusion_server.py"]
    }
  }
}
```

Replace the path with the actual absolute path to `fusion_server.py` on your machine. On Windows, use forward slashes or escaped backslashes in the path.

Restart Claude Desktop after saving the config.

## Usage

Once everything is set up:

1. Open Fusion 360 and make sure the FusionMCP add-in is running (check for the startup dialog)
2. Open Claude Desktop — you should see a hammer icon indicating MCP tools are available
3. Start asking Claude to create things in Fusion 360

### Example Prompts

```
Create a box that's 10cm x 5cm x 3cm
```

```
Draw a circle with radius 2cm on the XY plane and extrude it 5cm
```

```
Add 2mm fillets to all edges of the first body
```

```
Create a hollow cylinder: outer radius 3cm, wall thickness 0.5cm, height 8cm
```

```
Export the current design as STL to my desktop
```

```
Create a gear-like shape: draw a 12-sided polygon and extrude it, then add
a center hole with diameter 1cm
```

### Tips

- All dimensions in FusionMCP use **centimeters** (Fusion 360's internal unit)
- Angles are in **degrees** in tool parameters (converted to radians internally)
- Use `get_design_info` to see what's currently in your design
- Use `get_face_info` or `get_edge_info` before operations that target specific faces/edges
- The `execute_script` tool lets Claude run arbitrary Fusion 360 Python API code for anything not covered by the built-in tools

## Troubleshooting

**"Cannot reach Fusion 360"**
- Make sure Fusion 360 is open
- Check that the FusionMCP add-in is running (Utilities > Add-Ins)
- Verify nothing else is using port 7432

**Claude Desktop doesn't show MCP tools**
- Verify your `claude_desktop_config.json` syntax is valid JSON
- Make sure the path to `fusion_server.py` is correct and absolute
- Restart Claude Desktop after editing the config
- Check that `python` is in your PATH (try `python --version` in a terminal)

**Add-in fails to start in Fusion 360**
- Check the Fusion 360 text commands window for error details (View > Text Commands)
- Make sure the file is named `FusionMCP.py` inside a folder named `FusionMCP`

## License

MIT
