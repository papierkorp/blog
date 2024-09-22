---
layout: post
title: "Different ways to use Pathfinding for 2d Projects in Godot 4"
date: 2024-09-22
tags: gamedev godot howto
subtitle: "Using the NavigationAgent and AStar in Godot 4.3"
display_toc: false
comments_id: 3
---

# What

For my current Project I needed Pathfinding for the enemies and tried a bunch of things. My goal was to get an enemie which followed a specific Path and to give a few RallyPoints which had to be reached in order. But the Player can block the Path by placing Walls between the RallyTargets.

Now I just want to summarize my findings for future reference.

Be aware that this will only be the basic Usage and nothing advanced.

There are quite a few ways for Pathfinding in Godot 4:

- [NavigationAgent](https://docs.godotengine.org/en/stable/tutorials/navigation/navigation_using_navigationagents.html) (Godot way of using AStar)
  - with TileMaps
  - per Mesh
- [Astar2D](https://docs.godotengine.org/en/stable/classes/class_astar2d.html) (A\* the hard way)
- [AstarGrid2D](https://docs.godotengine.org/en/stable/classes/class_astargrid2d.html) (A\* optimized for 2D Grids)
- [PathFollow2D](https://docs.godotengine.org/de/4.x/classes/class_pathfollow2d.html) (simple way of path following, without pathfinding just straight from waypoint to waypoint)


# NavigationAgent

## with TileMaps

![Example Overview](/blog/assets/gamedev/godot/navigation_agent_example.gif)


- create a new Scene with a `TileMapLayer` (map) and add a new TileSet
  - make necessary Settings, e.g. TileSize
  - Physics Layers - Add Element
  - Navigation Layers - Add Element
  - add the Example Tilemap to the TileSet
    - go to Paint - Physics Layer - and paint your walls (in the example the black block)
    - go to Paint - Navigation Layer - and paint your floor (in the example the white block)
  - create a small example level..
- create a new Scene with a `CharacterBody2D` (follower)
  - make necessary settings (so it isnt stuck on corners)
    - Motion Mode: Floating
    - Wall Min Slide Angle: 0
    - remove collision between enemies (not necessary..)
      - Collision Layer: 3
      - Mask: nothing
  - add a Sprite2D
    - add a new AtlasTexture
      - Edit Region - Select the Red Ball
  - add a CollisionShape2D
    - use a small circle shape (if its exactly the size of the sprite/rectangle it can be stuck on corners)
  - add a NavigationAgent2D
  - add a Script
- create a new Scene with a `Area2D` (rally point)
  - now to create a target for the characterbody2d to charge at we could use our player or we create another target (which i needed for my project ..)
  - repeat the same as with the follower (sprite and collision shape)
  - go to Node - Groups - Add a new group and call it `RallyPoints`
  - Collision Settings
    - Collision: Nothing
    - Mask: 3

```python
extends CharacterBody2D

const speed = 200

@onready var nav_agent: NavigationAgent2D = $NavigationAgent2D

var rally_points: Array[Area2D] = []
var current_target_index: int = 0

func _ready():
    get_rally_targets()
    calculate_path()

func _physics_process(delta: float) -> void:
    move()

func get_rally_targets():
    var rally_nodes = get_tree().get_nodes_in_group("RallyPoints")
    for node in rally_nodes:
        if node is Area2D:
            rally_points.append(node)

    rally_points.sort()

func calculate_path() -> void:
    if current_target_index < rally_points.size():
        nav_agent.target_position = rally_points[current_target_index].global_position
    else:
        print("FINAL TARGET REACHED")
        self.queue_free()

func is_rally_point_reached() -> bool:
    if current_target_index < rally_points.size():
        var current_rally_point = rally_points[current_target_index]
        return current_rally_point.overlaps_body(self)
    return false

func move():
    if is_rally_point_reached():
        print(self, " RALLY POINT REACHED")
        current_target_index += 1
        calculate_path()
    else:
        var dir = to_local(nav_agent.get_next_path_position()).normalized()
        velocity = dir * speed
        move_and_slide()


```

- now add the follower and the rallytarget to the tilemap Scene

## per Mesh

soon to be coming (surely...) ...

... but pretty identical to the Tilemap variant

# AstarGrid2D

![Example Overview](/blog/assets/gamedev/godot/astargrid2d_example.gif)



- create a new Scene with a `Node2D` (map)
  - add a `TileMapLayer` (floor) and then a new TileSet
    - RightClick - Access as UniqueName
  - add a `TileMapLayer` (walls) and then a new TileSet
    - RightClick - Access as UniqueName
  - create a small example level..
    - start at the coordinate (0,0), otherwise
- create a new Scene with a `CharacterBody2D` (follower)
  - make necessary settings (so it isnt stuck on corners)
    - Motion Mode: Floating
    - Wall Min Slide Angle: 0
    - remove collision between enemies (not necessary..)
      - Collision Layer: 3
      - Mask: nothing
  - add a Sprite2D
    - add a new AtlasTexture
      - Edit Region - Select the Red Ball
  - add a CollisionShape2D
    - use a small circle shape (if its exactly the size of the sprite/rectangle it can be stuck on corners)
  - add a Script

**Script**

1. initate grid
2. get target coords
3. send self to target (+ draw line if necessary)
4. check if target reached / last target reached => repeat from 2

```python
extends CharacterBody2D

@onready var TileMapLayerFloor: TileMapLayer = %floor
@onready var TileMapLayerWalls: TileMapLayer = %walls
@onready var map: Node2D = get_parent()

const speed = 500

var astargrid = AStarGrid2D.new()
var rally_points: Array[Area2D] = []
var current_path: PackedVector2Array = []
var target_point_index = 0
var current_path_line: Line2D = null

func _ready():
  initialize_astargrid2d()
  get_rally_targets()
  set_next_target()

func _process(_delta):
  if Input.is_mouse_button_pressed(MOUSE_BUTTON_MIDDLE):
    print_positions()
  if !current_path.is_empty():
    var next_point = current_path[0]
    var direction = (next_point - global_position).normalized()
    
    velocity = direction * speed
    move_and_slide()

    if global_position.distance_to(next_point) < 5:
      current_path.remove_at(0)
      if current_path.is_empty():
        set_next_target()


func initialize_astargrid2d():
  astargrid.region = TileMapLayerFloor.get_used_rect()
  astargrid.cell_size = TileMapLayerFloor.tile_set.tile_size
  astargrid.jumping_enabled = true
  astargrid.diagonal_mode = AStarGrid2D.DIAGONAL_MODE_ONLY_IF_NO_OBSTACLES
  astargrid.update()

  for tile in TileMapLayerWalls.get_used_cells():
    astargrid.set_point_solid(tile)

  astargrid.update()

func get_rally_targets():
  var rally_nodes = get_tree().get_nodes_in_group("RallyPoints")
  for node in rally_nodes:
    if node is Area2D:
      rally_points.append(node)
  
  rally_points.sort()

func set_next_target():
  if target_point_index < rally_points.size():
    var target = rally_points[target_point_index]
    var start_point = TileMapLayerFloor.local_to_map(global_position)
    var end_point = TileMapLayerFloor.local_to_map(target.global_position)
    var path = astargrid.get_point_path(start_point, end_point)

    if path.is_empty():
      print("No path found to target ", target_point_index + 1)
      target_point_index += 1
      set_next_target()
    else:
      current_path.clear()
      current_path = path
      target_point_index += 1
      print_path(path)
  else:
    print("All targets reached")
    queue_free()


func print_path(path: PackedVector2Array) -> void:
  if current_path_line:
    current_path_line.queue_free()
  
  var world_path: PackedVector2Array = []
  for point:Vector2i in path:
    world_path.append(point + Vector2i(astargrid.cell_size) / 2)
  
  current_path_line = Line2D.new()
  map.add_child(current_path_line)
  current_path_line.width = 2
  current_path_line.default_color = Color.LIGHT_GREEN
  current_path_line.begin_cap_mode = Line2D.LINE_CAP_ROUND
  current_path_line.end_cap_mode = Line2D.LINE_CAP_ROUND
  current_path_line.joint_mode = Line2D.LINE_JOINT_ROUND
  current_path_line.points = world_path

#-------------DEBUGGING-------------

func print_positions() -> void:
  var global_pos = get_global_mouse_position()
  var tilemap_pos_floor = TileMapLayerFloor.local_to_map(global_pos)
  var in_bounds = astargrid.is_in_bounds(tilemap_pos_floor.x, tilemap_pos_floor.y)
  var is_solid = astargrid.is_point_solid(tilemap_pos_floor)
  print("get_tree:")
  get_tree().root.print_tree()
  print("Global Position: ", global_pos)
  print("Map Position Floor: ", tilemap_pos_floor)
  print("in_bounds: ", in_bounds)
  print("is_solid: " , is_solid)

```


# AStar2D

soon to be coming (surely...) ...

... but pretty identical to the Tilemap variant

# PathFollow2D

soon to be coming (surely...) ...

# Afterwords

Here the zip File for the project:

[Example Godot Project](/blog/assets/gamedev/godot/pathfinding_example.zip)

Have Fun!