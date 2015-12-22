================
Planned Features
================

Control Over More Actions
=========================

It is planned to be able to control many more actions, but since the Sponge API is still in heavy development, it is not possible yet to implement them.
As soon as Sponge is ready, these features will get added.
These planned features include controlling the following actions:

* Crafting items
* Smelting items
* Brewing potions
* Enchanting items
* Repairing items
* Renaming items
* Dropping items from destroyed blocks
* Dropping items from killed entities
* Using the fishing rod

Player Matchers
===============

Right now, it is not possible to use a player as a valid entity when filtering events.
For example, it is possible to deny attacking any mob using a sword, but it is not possible to deny attacking players since they're not a valid Minecraft entity.
Player matchers will solve this problem and even add some more possibilities. 
It will be possible to select not only players, but specific players e.g. players with a specific permission.

Config Validation
=================

The config files of ActionControl can get very complex.
Because it is easy to make a small mistake when changing the config file, it is required that the plugin can tell the user about errors in the config file.
Right now, the config is validated very poorly, which means that the plugin will throw unexpected exceptions while running which is bad.

Modularize Action Controllers
=============================

To modularize action controllers means that every action controller will be exported into a separate jar file and can then be copied into a special directory for action controllers.
This allows server admins to completely customize ActionControl by only adding the action controllers that they really need.
Furthermore, this allows ActionControl to work with any kind of mod, since action controllers can be created for a specific mod.
A server admin can then easily install these custom action controllers.
A good example for this would be a custom action controller for the Mining Laser from the IndustrialCraft mod.

Customizable Messages
=====================

When a player can't perform an action because ActionControl denies it, there is currently no indication for the player why he isn't allowed to perform that action.
It is planned to have a separate config file in which server admins can customize the chat messages that are beint sent to the player.
Within this config file, it should also be possible to access some important attributes like the group of the player, to include them in the message.
