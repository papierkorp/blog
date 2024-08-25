---
layout: post
title: "Different ways to use Pathfinding for 2d Projects in Godot 4"
date: 2023-12-08
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

# NavigationAgent

## with TileMaps

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

# AStar2D

- create a new Scene with a `Node2D` (map)
  - add a `TileMapLayer` (floor) and then a new TileSet
    - RightClick - Access as UniqueName
  - add a `TileMapLayer` (walls) and then a new TileSet
    - RightClick - Access as UniqueName
  - create a small example level..
- create a new Scene with a `CharacterBody2D` (follower)
  - add a Sprite2D
    - add a new AtlasTexture
      - Edit Region - Select the Red Ball
  - add a CollisionShape2D
    - use a small circle shape (if its exactly the size of the sprite/rectangle it can be stuck on corners)
  - add a Script

# AstarGrid2D
