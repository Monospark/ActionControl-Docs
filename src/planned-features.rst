================
Planned Features
================

HOCON support
=============

The config files are currently JSON files.
Since JSON is a very rigid format that doesn't allow comments, HOCON would be a better option.

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
