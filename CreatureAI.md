# Creature AI Behaviour in Ragnarok Online

Creatures in RO exhibit a simplistic behaviour that is derived from predefined configuration settings ("presets"). These categorize monsters roughly by their intended approach to  interacting with other actors in the game world, and are then enriched with specific ability usage patterns for each individual creature type to provide additional depth.

## Prerequisites

If you aren't already familiar with the following topics, you might want to read up on them:

* [Finite State Machines](https://en.wikipedia.org/wiki/Finite-state_machine): Simple AI is frequently implemented with an automaton that switches between states based on inputs from the simulation, as is the case in RO

## Definitions

Lengthy explanations in the tables below aren't ideal, so let's clarify some terminology:

* Individual actors in the game world I call **units**, with *creatures* (neutral or hostiles,  "monsters") and *NPCs* being special cases with differing behaviour
* A unit is said to **touch** another when its position is identical, or directly adjacent to another's position (within one world unit in either direction)
* A unit's **area of interest** consists of a roughly circular area around the unit representing its *field of view* (2D "line of sight"), in which relevant information is provided to it by the server (with a radius of 14 world units by default, not counting the unit's position itself)
* An attack is considered a *short-range* or **melee** attack when the distance between attacker and target is less than or equal to five world units, and *long-range* or simply **ranged** otherwise (only the maximum range counts; spells are therefore always ranged)
* Creatures that are spawned by another creature I'll call **minions** or *followers*, and the connection between both a **link** or *tether*. Linked creatures are tethered to a *parent* unit and will usually follow anywhere it goes. If linked they will die when it does, as well
* Creatures are said to be **allied** if they are of the same class, and are set to *assist* each other in combat when attacked by players in the assisting creature's area of interest

These are mostly arbitrary terms I assign to the concepts in a way that I think most accurately describes them, based on my own understanding. Some may be known under a different name in the RO community (such as master/slave), but there is likely some overlap.

## High-Level Overview

AI behaviours can be sorted into a few basic categories:

* Movement: Random walks, following parents, and chasing targets
* Combat: Reacting to attacks/spellcasts, target selection (including AOI checks)
* Looting: Picking up items from the floor

There's two special categories worth mentioning specifically:

* Skills: Abilities that are defined for each creature type, and specific to only it
* NPC Events: Unrelated to creatures, they facilitate regular dialogs and interactions

The first type of behaviour is implemented in the basic AI presets mentioned in the introduction, and assigned to creatures. It can also be changed on the fly to enable more complex reactions to standard gameplay situations.

Each unit class (monster type) also comes with a set of preconfigured skill usage patterns, based on the AI FSM's current state and probabilities for each event (skill use/emote/chat).

*NPC events* are a special case and only apply to regular (non-creature) NPCs, which are technically using the same framework but with fewer, specialized states and no transitions, as it seems that the state of NPC actors never really changes (i.e., they are always "Idle").

## AI States

The behaviour of each unit is identified by a (finite) state machine; at any given point in time the FSM's state defines what the unit is doing in the game world (by setting the *AI state*).

There's a relatively small number of these states and each unit's AI FSM will constantly switch between them, based on inputs provided by ingame events (see [below](#input-and-output-events)):

| State ID | Interpretation | Description |
| :---: | :---: | :---: |
| ``ABNORMAL_ST`` | Status | Impacted by loss of control effects that prevent it from acting normally |
| ``ANGRY_ST`` | ? (TBD) | Aggressively attacking or chasing target, using an alternate behaviour set (switches after being attacked) |
| ``BERSERK_ST`` | Attacking | In range and actively attacking their target |
| ``DEAD_ST`` | Dead | Currently removed from the map (inactive until respawn)
| ``FOLLOW_SEARCH_ST`` | Scanning (Follow) | Seeking new targets in the area of interest, while following a parent unit (?) |
| ``FOLLOW_ST`` | Following | Moving to a new location, while following a parent unit (?) |
| ``IDLE_ST`` | Idle | No action is currently in progress (default) |
| ``MOVEITEM_ST`` | Looting | Moving towards an item, with the intent to loot it |
| ``RMOVE_ST`` | Walking | Moving (random walk, when not in combat) |
| ``RUSH_ST`` | Pursuing | Chasing down a target (player), with the intent to attack it |
| ``SEARCH_ST`` | Scanning | Monitoring the area of interest to find new targets |

## Input and Output Events

AI state changes are triggered by certain game events, which serve as *input* to the underlying FSM. Similarly, state transitions emit *output* events which can then influence the game world.

The exact order this happens isn't entirely clear, but there's some evidence the output events are emitted some time *before* the resulting state transition is executed:

> The term spawn is misleading as monsters don’t technically die, rather monsters are temporarily removed from the map.  This is important because when a monster is returned to the map the monster AI will remain in the previous state, e.g. if a monster is aggressive to a player, and were to respawn near that player, it will remain aggressive towards that player.  While this is rarely encountered due to a combination of horizontal/vertical position variance and non-zero regeneration timers it does explain otherwise inexplicable behavior, e.g. an ordinarily passive monster respawning with aggressive behavior.

Source: [Kokotewa's Blog](https://blog.kokotewa.com/monster-spawns/)

This would necessitate events unfolding in the following order:

1. Creature is "killed" (removed from map) and set to a "Dead" AI state
2. The respawn timer expires, which triggers the regeneration logic
3. The creature is revived in its original AI state (output event triggers here)
4. **It "spawns" next to a player, still in its original AI state**
5. The player's proximity triggers the appropriate input event
6. The creature immediately attacks the player, switching AI states
7. The "Idle" output state is never reached, because another transition overrode it

If there was no player to attack when the creature respawns, it should follow that instead of 5 to 7, the output state will then be set (after the "scanning for targets" step). This is mere conjecture, but I don't see any other explanation for the aforementioned phenomenon.

### Input Events

These are the ingame events that can trigger FSM state transitions:

| ID | Description | Scope |
| :---: | :---: | :---: |
``ARENASTART_IN`` | Some "arena" event was started (?) | NPC |
``ARRIVEDAT_ITEM_IN`` | The creature arrived at the location of the item it is currently trying to loot | MOB |
``ATTACKED_IN`` | The creature was attacked (by a player?) | MOB |
``CHANGE_NORMALST_IN`` | All loss of control effects were removed from the creature | MOB |
``CHARACTER_ATTACKSIGHT_IN`` | A player entered the creature's attack range (on their own?) | MOB |
``CHARACTER_INATTACKSIGHT_IN`` | The creature reached the targeted unit and is now within attack range | MOB |
``CHARACTER_INSIGHT_IN`` | A player entered the creature's area of interest | MOB |
``CLICK_IN`` | The NPC was clicked (player started interacting with it) | NPC |
``DEADSTATE_TIMEOUT_IN`` | The respawn timer for the creature has elapsed | MOB |
``DESTINATION_ARRIVED_IN`` | The creature has arrived at the target location of its most recent movement action | MOB |
``ENEMY_INATTACKSIGHT_IN`` | The targeted player is within the creature's attack range | MOB |
``ENEMY_OUTATTACKSIGHT_IN`` | The targeted player is outside the creature's attack range (but within its area of interest) | MOB |
``ENEMY_OUTSIGHT_IN`` | The targeted player has left the creature's area of interest | MOB |
``ENERGY_RECHARGED_IN`` | The creature's power has replenished (?) | MOB |
``FRIEND_ATTACKED_IN`` | An allied creature was attacked in the creature's area of interest | MOB |
``G_CHARACTER_ATTACKSIGHT_IN`` | A player entered the WOE guardian's attack range (?) | MOB |
``G_CHARACTER_INSIGHT_IN`` | A player entered the WOE guardian's area of interest (?) | MOB |
``ITEM_INSIGHT_IN`` | A lootable item was detected in the creature's area of interest | MOB |
``LOWERLEVEL_INSIGHT_IN`` | A viable target (low level player?) has been detected in the creature's area of interest | MOB |
``MAGIC_LOCKON_IN`` | A spellcast targeting the creature was detected | MOB |
``MILLI_ATTACKED_IN`` | The creature was attacked (by a player?) in melee range | MOB |
``MOVE_END_POS_IN`` | The creature has arrived (after moving to a new location) | MOB |
``MOVE_RANDOM_END_IN`` | The random movement delay timer is about to be reset | MOB |
``MOVE_START_IN`` | The creature is about to start moving to a new location | MOB |
``MYOWNER_ATTACKED_IN`` | The unit's parent unit was attacked | MOB |
``MYOWNER_OUTSIGHT_IN`` | The unit's parent has left the area of interest | MOB |
``NEAREST_CHARACTER_IN`` | The nearest player (or other unit?) has been updated for this creature | MOB |
``TARGET_ITEM_DISAPPEAR_IN`` | The item that the creature was trying to loot has disappeared from the floor | MOB |
``TOUCHED_IN`` | A player has moved on or directly adjacent to the NPC  | NPC |
``TOUCHED2_IN`` | A player or a party member has moved on or directly adjacent to the NPC (?) | NPC |
``TOUCHEDNPC_IN`` | Another NPC has moved on or directly adjacent to the NPC (?) | NPC |
``WAIT_END_IN`` | The random movement delay timer has elapsed | MOB |

Some of them are only used for "friendly" units, like vendors and quest NPCs. This is usually evident from the event description, but I've added a  "scope" field to make it even clearer.

This distinction does not exist in the game, as all NPC actors can theoretically be assigned arbitrary events and states (though the resulting behavior may be undefined?).

### Output Events

These are ingame events emitted by the FSM when a state transitions finishes.

| ID | Description | Scope |
| :---: | :---: | :---: |
``CALL_ARENASTART_OUT`` | Starts an "arena" event (?) | NPC |
``CALL_CLICKEVENT_OUT`` | Triggers the NPC's ``OnClick`` handler | NPC |
``CALL_TOUCHEVENT_OUT`` | Triggers the NPC's ``OnTouch`` handler (NPC touched by player) | NPC |
``CALL_TOUCHEVENT2_OUT`` | Triggers the NPC's ``OnTouch2`` handler (NPC touched by player or party member?) | NPC |
``CALL_TOUCHNPCEVENT_OUT`` | Triggers the NPC's ``OnTouchNPC`` handler (NPC touched by other NPC?) | NPC |
``CHANGE_ENEMY_OUT`` | Attempts to change targets (start scanning?) | MOB |
``CHANGE_NORMALST_OUT`` | Restores the default ("Idle") state | MOB |
``EXPEL_OUT`` | Starts pursuing a target, outside of the creature's attack range | MOB |
``MOVE_RANDOM_START_OUT`` | Starts a new random walk | MOB |
``MOVE_START_OUT`` | Starts a (random?) walk, for NPCs only? | NPC |
``MOVETO_ITEM_OUT`` | Starts moving to the location of an item on the floor | MOB |
``MOVETO_MYOWNER_OUT`` | Starts moving to the parent unit (reduce distance?) | MOB |
``NONE_OUT`` | Does nothing (dummy event?) | MOB |
``PICKUP_ITEM_OUT`` | Makes the creature loot an item | MOB |
``REVENGE_ENEMY_OUT`` | Starts using attacks (and/or abilities?) on a hostile unit in attack range? | MOB |
``REVENGE_RANDOM_OUT`` | Starts using attacks (and/or abilities?) on a random hostile unit in attack range? | MOB |
``SEARCH_OUT`` | Starts scanning for targets in the area of interest | MOB |
``STOP_AND_CLICKEVENT_OUT`` | Stops NPC movement, then triggers its ``OnClick`` handler? | NPC |
``STOP_MOVE_OUT`` | Stops the current movement, discarding the path | MOB |
``TRADE_START_OUT`` | Opens the vendor's trading window? | NPC |
``TRY_REVIVAL_OUT`` | Attempt to regenerate the creature (can it fail?) | MOB |
``WAIT_START_OUT`` | Starts the random walk delay timer | MOB |

## AI Presets

There are at least 27 known behaviour archetypes that form the basis of the AI system, though some appear to be unused/discarded experiments:

**TODO: Add table, identifity behavioural archetypes**

Please note that these are not fixed for any given monster, but rather assigned. The implication is that creatures are free to change between archetypes if their custom logic so dictates.

While this doesn't affect their skill usage, it can make them prioritize targets differently or move in new ways.

## Skill Usage

Skills are used according to a predefined behaviour table for each creature type, based on the creature's current AI state and random chance, as well as certain conditions. Emotes and chat can also be triggered in a similar fashion.

Example:

> Angeling has a 100% chance to summon new followers, when it's attacking or idle and the number of existing minions is three or less. This cast takes 2 seconds, is not interruptible, and can only be used once per minute. While doing so it will emote ``/heh``.

**TODO: Add more details**

## See also

* [Hercules monster modes reference](https://github.com/HerculesWS/Hercules/blob/stable/doc/mob_db_mode_list.md)