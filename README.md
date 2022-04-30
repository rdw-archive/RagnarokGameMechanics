**This repository has migrated! It is now archived and will no longer see updates.** My work continues here: [Ragnarok Research Lab](https://github.com/RagnarokResearchLab/)

To read the latest results of my research into Ragnarok Online's game mechanics, visit [https://ragnarokresearchlab.github.io/game-mechanics/](https://ragnarokresearchlab.github.io/game-mechanics/).

---

# RagnarokGameMechanics

This repository includes the findings of my research and inquiries into the game mechanics of Gravity Co's MMORPG Ragnarok Online.

Game mechanics in this context refers to only the gameplay logic that has been implemented in their server software; if you're interested in learning more about the client software you might want to check out the [RagnarokFileFormats](https://github.com/Duckwhale/RagnarokFileFormats) repository instead.

## Goals

While much knowledge of the various game mechanics is available, certain implementation details appear to be unclear or are only described as approximatations. This isn't very useful (to me), or at least not as useful as it could be, and so I decided to extend my documentation efforts.

The main goals then are:

* Documenting the exact behaviour of the game servers, as specifically as possible
* Providing data or other evidence that can serve to verify this behaviour where needed
* Create a single source of truth that can provide reliable information in a way that's easily understandable

Please note that all of this is written from a developer's perspective. If you're just a player interested in the high-level concepts, this might be far too much detail for your needs.

## Contributing

Please open an issue or contact me on Discord (RDW#9823) if you can contribute in any way!

## Game Mechanics Overview

This section serves mostly as a guideline for high-level topics that I want to eventually have included:

* [Server Update Loop](ServerUpdateLoop.md): Details on how the simulation of the world is updated
* [Movement](Movement.md): How actors move in the game world
* [Creature AI](CreatureAI.md): Describes the rules governing the behaviour of creatures (and NPCs)
* Physical Combat
* Statistics
* Casting
* Skills
* Experience and Leveling System
* Items and Loot
* Boss Encounters and Mini Bosses
* Rest TBD
