---
layout: post
title: "How to create a Turn-Based Game in Godot 4"
date: 2023-09-18
tags: gamedev godot howto
subtitle: "How to create a flexible Turn-Queue in Godot 4.2"
comments_id: 3
---
# Create a Turn Based Game in Godot 4.2

![Example Overview](/blog/assets/gamedev/godot/turn_based_example.png)

## Introduction

This is my first Project in godot and im completly new to Gamedevelopment, so there will propbly a lot of possible improvements. Nevertheless I want to share my Turn-Queue approach (also for self reference :D). It`s basically the same as [GDQeusts - How to Code a Turn-Based Game: Godot Turn Queue Tutorial](https://www.youtube.com/watch?v=FV4JkwI4OF4) with a few minor changes.

This is based on Godot_v4.1.2-stable_win64.

## What you get

A Turn Queue Node where you can add an unlimited amount of base_character Nodes. New Nodes will be implemented into the Turn Queue. The sequence is based on a `speed` Parameter of the base_character. Each Nodes turn will last until `_turn_end()` is called.

I also implemented an simple stamina/Actionpoint System where as a Character can only executed a set Amount of Actions.



# Create the Character Scene

First we will start the journey with a Character, for this we will need a new CharacterBody2D scene and name it `character`. (doesnt have to be a CharacterBody2D)

Add this Nodes:

* Sprite2D (drag and drop the godot Icon)
* CollisionShape2D (create a simple one just for the error, isnt needed)
* Label - `show_data_label` and place it above the Character Sprite (just to display some informations)
* AnimationPlayer
    - New Animation - active
        + select Sprite2D - Visibility - Modulate and make it a loop blinking (just as an example to see the current selected Character)
* CanvasLayer
    - HBoxContainer - `combat_menu` (set Anchor to Bottom wide)
        + BoxContainer - `possible_actions`
            * Button - MoveButton
                - Text: Move
            * Button - TurnEndButton
                - Text: End Turn
                - Signals - pressed() - connect
        + GridContainer - `possible_attacks`
            * Button - attack_1
                - Text: Attack 1
                - Signals - pressed() - connect
            * Button - attack_2
                - Text: Attack 2
                - Signals - pressed() - connect

make sure to hide the combat_menu after creating it. I would also recommend to make own Scene for the combat UI.

After all this add a Script:


```python
extends CharacterBody2D

class_name character

@onready var turn_queue: turn_queue
@onready var animation_player = $AnimationPlayer
@onready var combat_menu = $CanvasLayer/combat_menu
@onready var show_data_label = $show_data_label

enum CHAR_STATE {
    WAIT,
    IN_MOVE,
    IN_ACTION,
    IN_ATTACK
}

const _state_strings = {
    CHAR_STATE.WAIT: "WAIT",
    CHAR_STATE.IN_MOVE: "IN_MOVE",
    CHAR_STATE.IN_ACTION: "IN_ACTION",
    CHAR_STATE.IN_ATTACK: "IN_ATTACK"
}

@export var stamina: int
@export var speed: int

var _state: CHAR_STATE = CHAR_STATE.WAIT
var cur_stamina: int

func _ready():
    initialize()

func _physics_process(delta):
    if turn_queue.active_char == self:
        animation_player.play("active")
    else:
        animation_player.stop()
    var s = "%s - %s - %s speed - %s stamina left" % [self.name, _state_strings[_state], self.speed, self.cur_stamina]
    show_data_label.text = s

#--------------Turn Queue + Actions

func initialize():
    turn_queue = get_parent()


func play_turn():
    if _state == CHAR_STATE.WAIT and turn_queue.active_char == self:
        set_stamina(self.stamina)
        set_state(CHAR_STATE.IN_ACTION)
    elif _state == CHAR_STATE.WAIT:
        combat_menu.hide()
    match _state:
        CHAR_STATE.IN_ACTION:
            show_possible_actions()
        CHAR_STATE.IN_ATTACK:
            #do something
            #set_state(CHAR_STATE.IN_ACTION)
            pass

func show_possible_actions():
    combat_menu.show()


func set_state(new_state: CHAR_STATE) -> void:
    if new_state == _state:
        return
    _state = new_state


func combat_actions(action:String) -> void:
    #if _state == CHAR_STATE.WAIT:
#       combat_menu.hide()
    match action:
        "attack_1":
            if check_stamina(-4):
                set_state(CHAR_STATE.IN_ATTACK)
                print("attack1 with %s stamina" % stamina)
                change_stamina(-4)
                check_turn_end()
            else:
                check_turn_end()
        "attack_2":
            if check_stamina(-1):
                set_state(CHAR_STATE.IN_ATTACK)
                print("attack2 with %s stamina" % stamina)
                change_stamina(-1)
                check_turn_end()
            else:
                check_turn_end()
        "turn_end":
            _turn_end()

#--------------Stamina Helper Functions

func check_stamina(stamina: int) -> bool:
    var new_cur_stamina = (cur_stamina + stamina)
    if new_cur_stamina < 0:
        print("Not enough stamina left, you have: %s, you need: %s" % [cur_stamina, -stamina])
        return false
    else:
        return true

func set_stamina(stamina: int) -> void:
    cur_stamina = stamina

func change_stamina(stamina: int) -> void:
    var new_cur_stamina = (cur_stamina + stamina)
    cur_stamina = new_cur_stamina


#--------------Turn End

func check_turn_end() -> void:
    if cur_stamina == 0 and _state != CHAR_STATE.WAIT:
        _turn_end()
    else:
        play_turn()

func _turn_end() -> void:
        set_state(CHAR_STATE.WAIT)
        set_stamina(0)
        turn_queue.play_turn()


#--------------Buttons

func _on_turn_end_button_pressed():
    combat_actions("turn_end")


func _on_attack_1_pressed():
    combat_actions("attack_1")


func _on_attack_2_pressed():
    combat_actions("attack_2")

```

What are my thoughts behind this system and how do I want to extend it?

Basically `play_turn()` is called from the turn_queue node, so this is the start and the return after each action (if there is still stamina left).

Each Action is hidden behind a Button. Currently the Buttons are all created manually. In the future I intend to create a `base_button` with which I want to dynamically create this Buttons based on a json File.

The buttons uniformly call the `combat_actions()` Function which will handle and distribute the possible Actions. To end the cycle this function will always call `check_turn_end()` which in turn will either restart by calling `self.play_turn()` or allow the next Character to perform the actions by calling `turn_queue.play_turn()`.



# Create the Turn-Queue Scene

Now the important part the Turn Queue Node, for this we will add a Node2D - `turn_queue` Scene.
Currently all the children will be added manually, in the future they will be dynamically added per Script:

* Instatiate Child Scene - character
    - set speed
    - set stamina
* Instatiate Child Scene - character2
    - set speed
    - set stamina
* Instatiate Child Scene - character3
    - set speed
    - set stamina
* Instatiate Child Scene - character4
    - set speed
    - set stamina

Now add a Script:


```python
extends Node2D

class_name turn_queue

var all_chars: Array
var active_char: character
var next_active_char: character
var counter: int = 0

func initalize():
    get_chars()
    play_turn()

func play_turn():
    if (next_active_char):
        active_char = next_active_char
    counter += 1 # Turn Counter
    active_char.play_turn()
    var new_index: int = (active_char.get_index() + 1) %  get_child_count() # sets the index to the next child or to 0 if there are no more childs
    next_active_char = get_child(new_index)

func get_chars() -> void:
    all_chars = get_children(false)
    all_chars.sort_custom(sort_chars) #sort all childs based on e.g. a speed parameter
    for char in all_chars:
        char.move_to_front() # puts this node to the first position of its parent
    active_char = get_child(0)

func sort_chars(a : character,b: character) -> bool:
    #add a custom speed calculation
    #eg a.speed * a.stamina > b.speed * b.stamina
    return a.speed > b.speed

func add_new_char(char: character) -> void:
    #currently not used, but you propably get the gist :D
    var new_char = char.new()
    add_char_to_queue(new_char)

func add_char_to_queue(char: character) -> void:
    #currently not used, but you propably get the gist :D
    add_child(char)

func remove_char_from_queue(char: character) -> void:
    #currently not used, but you propably get the gist :D
    char.queue_free()

```


# Add a level Scene

As for the level we only need a Node2D - `level` Scene, then Instatiate Child Scene - turn_queue and a small Script to initalize the turn_queue:

```python
func _ready():
    var turn_queue: turn_queue = get_node("TurnQueue")
    turn_queue.initalize()
```

