# Blender MCP Grand Challenge - Project Status & Next Steps

This document outlines the current status and planned next steps for the "grand challenge" scenario, which involves using the Blender MCP server to build and render a complex 3D scene based on a game design brief.

## 1. Current Status (As of Pause)

We are currently **paused in Phase B: Asset kit-bashing**.

*   **Objective of Phase B:** Build a "vertical slice" of an alpine village at dusk.
*   **Progress:**
    *   Phase A (Brief Ingestion & Task Graph Generation) was completed successfully. A JSON task graph was generated based on the provided game design brief.
    *   Core terrain and lighting setup (Phase B, Step 1) was partially completed:
        *   A snow texture (`snow_02`) was identified and its material created.
        *   An HDRI skybox (`belfast_sunset_puresky`) was imported and set as the world environment.
        *   A sun light was added to the scene.
    *   **Blocking Issue:** We encountered persistent "No data received" communication errors when attempting to download model assets (specifically pine trees: `fir_tree_01`, `pine_tree_01`) from Poly Haven using the `download_polyhaven_asset` tool.
*   **Reason for Pause:** To allow the user to arrange an alternative asset source or access method (potentially via "Superhive").

## 2. User's Plan for Phase B (JSON)

The following JSON structure was provided by the user to guide the execution of Phase B. We will resume following this plan, adapting asset sourcing as needed.

```json
[
  /* --------------------------------------------------------------------- */
  /* 1. CORE TERRAIN & LIGHTING                                           */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"Ardennes winter terrain heightmap", "tags":["terrain","snow","pbr"], "limit":1},
      "assign":"terrain"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$terrain.id", "location":[0,0,0], "scale":[1,1,1]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"HDRI dusk winter sky", "tags":["hdri","skybox","dusk"], "limit":1},
      "assign":"skybox"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$skybox.id", "location":[0,0,0]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"run_op",
      "args":{"op":"bpy.ops.object.light_add",
              "kwargs":{"type":"SUN","location":[0,0,30],"rotation":[0.7,0,0.8]} }
  }},

  /* --------------------------------------------------------------------- */
  /* 2. NATURAL ELEMENTS (8 assets)                                        */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"snow pine tree", "tags":["tree","conifer","winter"], "limit":2},
      "assign":"pines"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"bare deciduous tree winter", "tags":["tree","deciduous","winter"], "limit":1},
      "assign":"deciduous"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"snow undergrowth bush", "tags":["foliage","undergrowth"], "limit":1},
      "assign":"undergrowth"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$pines[0].id", "location":[-10,12,0], "scale":[1.4,1.4,1.4]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$pines[1].id", "location":[8,-15,0], "scale":[1.2,1.2,1.2]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$deciduous[0].id", "location":[-6,-8,0], "scale":[1.5,1.5,1.5]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$undergrowth[0].id", "location":[-4,3,0], "scale":[0.8,0.8,0.8]}
  }},

  /* --------------------------------------------------------------------- */
  /* 3. PROCEDURAL PLACEHOLDERS (2 assets via Geometry Nodes)              */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"py_eval",
      "args":{"code":"
import bpy, random
bpy.ops.mesh.primitive_cube_add(size=2, location=(4,4,0))
house = bpy.context.active_object
house.name = 'GN_Lowpoly_House_A'
# Simple GN modifier: inset → extrude → random roof pitch
gn = house.modifiers.new('GN_House','NODES')
nt = bpy.data.node_groups.new('House_GN','GeometryNodeTree')
gn.node_group = nt
# (node-creation code omitted for brevity)
"} }},

  { "role":"assistant", "name":"blender", "content":{
      "cmd":"py_eval",
      "args":{"code":"
import bpy, random
bpy.ops.mesh.primitive_cube_add(size=2.5, location=(-12,-3,0))
barn = bpy.context.active_object
barn.name = 'GN_Lowpoly_Barn'
gn = barn.modifiers.new('GN_Barn','NODES')
nt = bpy.data.node_groups.new('Barn_GN','GeometryNodeTree')
# (node-creation code omitted for brevity)
"} }},

  /* --------------------------------------------------------------------- */
  /* 4. DESTRUCTIBLE STRUCTURES & RUBBLE (3 assets)                        */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"WW2 european cottage destructible", "tags":["building","modular","destructible"], "limit":1},
      "assign":"cottage"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$cottage[0].id", "location":[2,-10,0], "scale":[1,1,1]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"rubble pile snow", "tags":["debris","rubble"], "limit":1},
      "assign":"rubble"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$rubble[0].id", "location":[3,-11,0]}
  }},

  /* --------------------------------------------------------------------- */
  /* 5. KEY PROP & VEHICLE (2 assets)                                      */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"Panzer IV tank winter camo", "tags":["tank","ww2","vehicle"], "limit":1},
      "assign":"panzer"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$panzer[0].id", "location":[-20,-5,0], "rotation":[0,0,1.57], "scale":[0.6,0.6,0.6]}
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"asset_search",
      "args":{"query":"WW2 intel cache radio satchel", "tags":["prop","objective"], "limit":1},
      "assign":"intel"
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"import_asset",
      "args":{"asset_id":"$intel[0].id", "location":[1.2,-9.5,0.4], "scale":[0.4,0.4,0.4]}
  }},

  /* --------------------------------------------------------------------- */
  /* 6. VOLUMETRIC SNOW PARTICLES & WIND                                   */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"run_op",
      "args":{"op":"bpy.ops.object.effector_add",
              "kwargs":{"type":"WIND","location":[0,0,10]} }
  }},
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"py_eval",
      "args":{"code":"
import bpy
# Add an emitter plane 50×50 m above the scene
bpy.ops.mesh.primitive_plane_add(size=50, location=(0,0,30))
emitter = bpy.context.active_object
ps = emitter.modifiers.new('Snow','PARTICLE_SYSTEM').particle_system
ps.settings.count = 120000
ps.settings.lifetime = 600
ps.settings.render_type = 'OBJECT'
# Re-use a small icosphere as the snowflake object
bpy.ops.mesh.primitive_ico_sphere_add(radius=0.04)
flake = bpy.context.active_object
ps.settings.instance_object = flake
"} }},

  /* --------------------------------------------------------------------- */
  /* 7. HOUSEKEEPING                                                       */
  /* --------------------------------------------------------------------- */
  { "role":"assistant", "name":"blender", "content":{
      "cmd":"get_state",
      "args":{"selector":"scene"},               /* sanity snapshot */
      "assign":"scene_state"
  }}
]
```

## 3. Detailed Next Steps (Upon Resuming Phase B)

1.  **Integrate New Asset Source (e.g., Superhive):**
    *   If the new asset source provides an MCP-compatible interface (e.g., new tools like `search_superhive_assets`, `download_superhive_asset`), I will use these tools. Please provide the new tool names and their expected arguments if this is the case.
    *   If assets from the new source need to be manually downloaded or placed by you, I will require the asset IDs (if applicable for an import tool) or file paths (if I need to script their import using `bpy.ops.import_scene` or similar via `execute_blender_code`).

2.  **Acquire Natural Element Assets (Phase B, Step 2 - Resuming):**
    We were attempting to acquire the following assets. We will need to source these (or suitable alternatives) from the new asset provider:
    *   **Two Pine/Fir Trees:**
        *   Previously identified from Poly Haven (failed to download): `fir_tree_01`, `pine_tree_01`.
        *   User plan variable: `$pines[0].id`, `$pines[1].id`.
    *   **One Bare Deciduous Tree (Winter Look):**
        *   Previously identified from Poly Haven: `dead_tree_trunk_02`.
        *   User plan variable: `$deciduous[0].id`.
    *   **One Snow Undergrowth Bush/Element:**
        *   Previously identified from Poly Haven: `dry_branches_medium_01`.
        *   User plan variable: `$undergrowth[0].id`.

3.  **Import and Place Natural Elements:**
    Once the new asset IDs/paths are known, I will proceed with the import commands from your JSON plan, adapting them as necessary for the new asset source. For example, for the first pine tree:
    *   User's command: `{ "cmd":"import_asset", "args":{"asset_id":"$pines[0].id", "location":[-10,12,0], "scale":[1.4,1.4,1.4]} }`
    *   I will then use `execute_blender_code` to set the location and scale for each imported asset, e.g.:
        ```python
        import bpy
        # Assuming 'imported_asset_name' is known after import
        obj = bpy.data.objects.get('imported_asset_name')
        if obj:
            obj.location = (-10, 12, 0)
            obj.scale = (1.4, 1.4, 1.4)
        ```
    This process will be repeated for all four natural element assets.

4.  **Procedural Placeholders (Phase B, Step 3):**
    Execute the `py_eval` commands from your plan to create `GN_Lowpoly_House_A` and `GN_Lowpoly_Barn` using Geometry Nodes (via `execute_blender_code`).

5.  **Destructible Structures & Rubble (Phase B, Step 4):**
    Search for and import assets for a "WW2 european cottage destructible" and "rubble pile snow" using the new asset source, then place them.

6.  **Key Prop & Vehicle (Phase B, Step 5):**
    Search for and import assets for a "Panzer IV tank winter camo" and "WW2 intel cache radio satchel," then place them.

7.  **Volumetric Snow Particles & Wind (Phase B, Step 6):**
    Execute the `run_op` and `py_eval` commands to set up the wind effector and snow particle system.

8.  **Housekeeping (Phase B, Step 7):**
    Execute the `get_state` command (likely via `get_scene_info` or a more detailed `execute_blender_code` script) to get a snapshot of the scene.

## 4. Subsequent Phases (Future Work)

Once Phase B is successfully completed, we will proceed with the remaining phases of the grand challenge:

*   **C. Physics & animation pass:** Add wind-affected cloth banners and a flock of birds.
*   **D. Cinematic camera + render:** Produce a 10-second dolly shot.
*   **E. Iterative critique loop:** Accept human text feedback and apply deltas.
*   **F. Export & hand-off:** Package the scene to FBX and generate a manifest.

## 5. Tooling Context

All Blender operations will continue to be performed using the `github.com/ahujasid/blender-mcp` server.
*   Commands like `run_op` and `py_eval` from your plan will be executed using the `execute_blender_code` tool.
*   Asset searching and importing (e.g., `asset_search`, `import_asset`) will use the Poly Haven tools (`search_polyhaven_assets`, `download_polyhaven_asset`) or their equivalents for the new asset source (e.g., Superhive), once specified.

---

Please let me know when you are ready to resume, and provide details on how to access assets from the new source.
