=============
Configuration
=============

As opposed to other plugins, ActionControl can have multiple config files.
Every file in the *config/org.monospark.actioncontrol* directory is interpreted as a config file in the HOCON format.
Config names can be used to indicate the goal of the config, for example *disable-lighers.hocon*.
If no config is present on startup, an example config will be generated.

Reloading
=========

Players with the ``actioncontrol.reload`` permission are can execute the command ``/actioncontrol reload`` to reload all configuration files.
This is used to adopt the changes made to configuration files while the server is running.

SpongeMatchers
==============

The ActionControl configuration files make heavy use of `SpongeMatchers <https://forums.spongepowered.org/t/spongematchers-parsable-predicates-for-all-sponge-objects-v1-1/11927>`_.
SpongeMatchers is a library for Sponge plugins that allows you to create powerful matchers for all kinds of objects like blocks, item stacks, players and more.
For more information about SpongeMatchers and matcher creation, visit the `SpongeMatchers documentation <https://docs.monospark.org/spongematchers/>`_.

Structure
=========

The general structure of the config can be seen here::

    player-filter = "<player matcher>"
    action-rules {
        <rule name> {
            filter {
                ...
            },
            response {
                ...
            }
        },
        <rule name> {
            filter {
                ...
            },
            response {
                ...
            }
        },
        ...
    }
  
Player filters are used to specify which players are affected by the following action rules and accept a `player matcher <https://docs.monospark.org/spongematchers/types.html#players>`_.
Action rules specify how to react on an action performed by a player.
For example, the action rule *placeBlock* can be used to deny the players matched by the player filter to place certain blocks.

Action Rules
============

As mentioned previously, action rules specify how to react on an action.
They consist out of a *filter* and a *response* element.
The *filter* element lets you specify on which actions you want to react on. Since every type of action differs, the filters for them also differ.
The *response* element lets you specify what to do if the action has passed the filter.

List of Action Rules
--------------------

The following blocks show all currently implemented action rules and their filters

Block destruction
^^^^^^^^^^^^^^^^^

::

    break-block {
        filter = [{
            block = "<block matcher>",
            tool = "<item stack matcher>"
        },
        {
            block = "<block matcher>",
            tool = "<item stack matcher>"
        }],
        response {
            ...
        }
    }

Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Block placement
^^^^^^^^^^^^^^^
    
::

    place-block {
        filter {
            block = "<block matcher>",
        },
        response {
            ...
        }
    }
    
Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_.
 
Block interaction
^^^^^^^^^^^^^^^^^
    
::

    interact-with-block {
        filter = [{
            block = "<block matcher>",
            item = "<item stack matcher>"
        },
        {
            block = "<block matcher>",
            item = "<item stack matcher>"
        }],
        response {
            ...
        }
    }

Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
   
Item usage
^^^^^^^^^^
    
::

    use-item {
        filter {
            item = "<item stack matcher>"
        },
        response {
            ...
        }
    }

Used: `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Item crafting
^^^^^^^^^^^^^
    
::

    craft {
        filter {
            result = "<item stack matcher>"
        },
        response {
            ...
        }
    }

Used: `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Entity damaging
^^^^^^^^^^^^^^^
    
::

    damage-entity {
        filter = [{
            entity = "<entity matcher>",
            item = "<item stack matcher>"
        },
        {
            entity = "<entity matcher>",
            item = "<item stack matcher>"
        }],
        response {
            ...
        }
    }
    
Used: `entity matchers <https://docs.monospark.org/spongematchers/types.html#entities>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.

Entity interaction
^^^^^^^^^^^^^^^^^^
    
::

    interact-with-entity {
        filter = [{
            entity = "<entity matcher>",
            item = "<item stack matcher>"
        },
        {
            entity = "<entity matcher>",
            item = "<item stack matcher>"
        }],
        response {
            ...
        }
    }
    
Used: `entity matchers <https://docs.monospark.org/spongematchers/types.html#entities>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.

Action Responses
----------------

With action responses you can control what happens after the action is applied to the filter.
There are two elements, *match* and *noMatch*, which are executed when the action matches the filter or doesn't match the filter.
The structure of a response element looks like this::

    <rule name> {
        filter {
            ...
        },
        response {
            match = "<response>",
            noMatch = "<response>"
        }
    }
    
The *match* element or *noMatch* element can be omitted if you don't want to respond to the action in any way.
There are currently four response types available:

``deny``
  Cancels the action.
  
``command(<cmd>)``
  Executes the command *<cmd>* as the console. To refer to the player that performed the action, use ``<player>`` in your command.
  
``player-command(<cmd>)``
  Executes the command *<cmd>* as the player that performed the action.
  
``log(<message>)``
  Prints *<message>* in the console. To refer to the player that performed the action, use ``<player>`` in your message.
  
It is also possible to execute multiple action responses at once by declaring them in an array.

Example configurations
======================

Below are some real life examples that illustrate the capabilities of ActionControl and help you understand config creation even more.

Small RPG system
----------------

This example covers the creation of a small RPG system in which a player can have one out of four possible jobs:

* The farmer who can plant or harvest crops
* The miner who can use a pickaxe
* The hunter who can attack entities using a sword and a bow
* The woodcutter who can craft planks and use an axe

Using ActionControl, it's possible to realize this jobs system pretty easily.
It's always recommended to create multiple config files that are responsible for controlling only one action instead of one big and cluttered file.
    
*farmer.json*::

    # Match players without the actioncontrol.group.farmer permission.
    # Note that you can use different permission names, these are just examples.
    player-filter = "{'permissions': !{'actioncontrol.group.farmer': true}}"
    action-rules {
        interact-with-block {
            filter {
                block = "{'state': {'type': 'minecraft:farmland'}}",
                item = "{'type': 'minecraft:wheat_seeds'}"
            },
            response {
                match = "deny"
            }
        },
        break-block {
            filter {
                block = "{'state': {'type': 'minecraft:wheat'}}"
            },
            response {
                match = "deny"
            }
        }
    }
    
*miner.json*::

    # Match players without the actioncontrol.groupminer permission.
    player-filter = "{'permissions': !{'actioncontrol.group.miner': true}}"
    action-rules {
        break-block {
            filter {
                # We're using regular expressions here to match any pickaxe. 
                tool = "{'type': r'minecraft:.+?_pickaxe'}"
                # If you don't know regular expressions, you can create the same effect using a different approach:
                # tool = "{'type': 'minecraft:wooden_pickaxe' | 'minecraft:stone_pickaxe' | 'minecraft:iron_pickaxe' | 'minecraft:golden_pickaxe' | 'minecraft:diamond_pickaxe'}"
            },
            response {
                match = "deny"
            }
        }
    }
    
*hunter.json*::

    # Match players without the actioncontrol.group.hunter permission.
    player-filter = "{'permissions': !{'actioncontrol.group.hunter': true}}"
    action-rules {
        attack-entity {
            filter: {
                # We're using regular expressions again to match any sword.
                item = "{'type': r'minecraft:.+?_sword'}"
            },
            response: {
                match = "deny"
            }
        },
        use-item {
            filter {
                item = "{'type': 'minecraft:bow'}"
            },
            response {
                match = "deny"
            }
        }
    }
    
*woodcutter.json*::

    # Match players without the actioncontrol.group.woodcutter permission.
    player-filter = "{'permissions': !{'actioncontrol.group.woodcutter': true}}"
    action-rules {
        break-block {
            filter {
                # We're using regular expressions again to match any axe.
                tool = "{'type': r'minecraft:.+?_axe'}"
            },
            response {
                match = "deny"
            }
        },
        craft {
            # We're denying all non-woodcutters to craft planks.
            # Note that we need to specify the quantity of the crafting result.
            result = "{'type': 'minecraft:planks', 'quantity': 4}"
        }
    }
    
Now you just have to assign the permissions to each group.
And here you have it, a fully working RPG system implemented by just using a single plugin.
Of course this is a fairly basic RPG system but it can be extended in any way to fit your needs.

Global blacklist
----------------

This examples shows you how to disallow certain action for all players except admins.

The first config denies players that don't have an admin permission to activate portals in the overworld.

*disable-portals.json*::

    player-filter = "{'permissions': !{'admin.permission': true}, 'location': {'world': {'dimension': {'name': 'overworld'}}}}"
    action-rules {
        interact-with-block {
            filter: {
                block = "{'state': {'type': 'minecraft:obsidian'}}",
                item = "{'type': 'minecraft:flint_and_steel'}"
            },
            response {
                match = "deny"
            }
        }
    }
    
The next config denies players that don't have an admin permission to place certain blocks.
    
*banned-blocks.json*::

    player-filter = "{'permissions': !{'admin.permission': true}}"
    action-rules {
        place-block {
            filter {
                block = "{'state': {'type': 'minecraft:beacon' | 'minecraft:tnt'}}"
            },
            response {
                match = [
                    "deny",
                    "player-command(say Hey everyone, I didn't read the rules!)"
                ]
            }
        }
    }

Command buttons
---------------

This examples shows you how to execute multiple commands after a player pressed a certain button.

*command-buttons.json*::

    player-filter = "*"
    action-rules {
        interact-with-block {
            filter {
                block = "{'state': {'type': {'id': 'minecraft:stone_button'}}, 'location': {'x': 1, 'y': 70, 'z': 23, 'world': {'name': 'world'}}}"
            },
            response {
                # We use the <player> placeholder that will be replaced with the players name on execution
                match = [
                    "command(give <player> diamond_pickaxe)",
                    "player-command(say I just clicked the button!)"
                ]
            }
        }
    }

----

Remember that these are just basic examples and if you want to do something entirely different, you can do that too!