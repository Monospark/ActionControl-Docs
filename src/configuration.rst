=============
Configuration
=============

As opposed to other plugins, ActionControl can have multiple config files.
Every file in the *config/actioncontrol* directory is interpreted as a config.
Config names can be used to indicate the goal of the config, for example *disable-lighers.json*.
If no config is present on startup, an example config will be generated.

Reloading
=========

Players with the ``actioncontrol.reload`` permission are can execute the command ``/actioncontrol reload`` to reload all configuration files.
This is used to adopt the changes made to configuration files while the server is running.

SpongeMatchers
==============

The ActionControl configuration files make heavy use of `SpongeMatchers <https://github.com/monospark/spongematchers>`_.
SpongeMatchers is a library for Sponge plugins that allows you to create powerful matchers for all kinds of objects like blocks, item stacks, players and more.
For more information about SpongeMatchers and matcher creation, visit the `SpongeMatchers documentation <https://docs.monospark.org/spongematchers/>`_.

Structure
=========

The general structure of the config can be seen here::

    {
        "playerFilter": "<player matcher>",
        "actionRules": {
            "<rule name>": {
                "filter": {
                    ...
                },
                "response": {
                    ...
                }
            },
            "<rule name>": {
                "filter": {
                    ...
                },
                "response": {
                    ...
                }
            },
            ...
        }
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

    "breakBlock": {
        "filter": [{
            "block": "<block matcher>",
            "tool": "<item stack matcher>"
        },
        {
            "block": "<block matcher>",
            "tool": "<item stack matcher>"
        }],
        "response": ...
    }

Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Block placement
^^^^^^^^^^^^^^^
    
::

    "placeBlock": {
        "filter": {
            "block": "<block matcher>",
        },
        "response": ...
    }
    
Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_.

Block damaging
^^^^^^^^^^^^^^

::

    "leftClickBlock": {
        "filter": [{
            "block": "<block matcher>",
            "item": "<item stack matcher>"
        },
        {
            "block": "<block matcher>",
            "item": "<item stack matcher>"
        }],
        "response": ...
    }

Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Block interaction
^^^^^^^^^^^^^^^^^
    
::

    "rightClickBlock": {
        "filter": [{
            "block": "<block matcher>",
            "item": "<item stack matcher>"
        },
        {
            "block": "<block matcher>",
            "item": "<item stack matcher>"
        }],
        "response": ...
    }

Used: `block matchers <https://docs.monospark.org/spongematchers/types.html#blocks>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
   
Item usage
^^^^^^^^^^
    
::

    "useItem": {
        "filter": {
            "item": "<item stack matcher>"
        },
        "response": ...
    }

Used: `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.
    
Entity damaging
^^^^^^^^^^^^^^^
    
::

    "leftClickEntity": {
        "filter": [{
            "entity": "<entity matcher>",
            "item": "<item stack matcher>"
        },
        {
            "entity": "<entity matcher>",
            "item": "<item stack matcher>"
        }],
        "response": ...
    }
    
Used: `entity matchers <https://docs.monospark.org/spongematchers/types.html#entities>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.

Entity interaction
^^^^^^^^^^^^^^^^^^
    
::

    "rightClickEntity": {
        "filter": [{
            "entity": "<entity matcher>",
            "item": "<item stack matcher>"
        },
        {
            "entity": "<entity matcher>",
            "item": "<item stack matcher>"
        }],
        "response": ...
    }
    
Used: `entity matchers <https://docs.monospark.org/spongematchers/types.html#entities>`_, `item stack matchers <https://docs.monospark.org/spongematchers/types.html#item-stacks>`_.

Action Responses
----------------

With action responses you can control what happens after the action is applied to the filter.
There are two elements, *match* and *noMatch*, which are executed when the action matches the filter or doesn't match the filter.
The structure of a response element looks like this::

    "<rule name>": {
        "filter": ... ,
        "response": {
            "match": <response> ,
            "noMatch": <response>
        }
    }
    
The *match* element or *noMatch* element can be omitted if you don't want to respond to the action in any way.
There are currently four response types available:

``deny``
  Cancels the action.
  
``command(<cmd>)``
  Executes the command *<cmd>* as the console. To refer to the player that performed the action, use ``<player>`` in your command.
  
``playerCommand(<cmd>)``
  Executes the command *<cmd>* as the player that performed the action.
  
``log(<message>)``
  Prints *<message>* in the console. To refer to the player that performed the action, use ``<player>`` in your message.
  
It is also possible to execute multiple action responses at once by declaring them in an array.

Example configuration
=====================

Here is an example that uses all of the previously covered features to realize a small RPG system in which a player can have one out of four possible jobs:

* The farmer who can plant or harvest crops
* The miner who can use a pickaxe
* The hunter who can attack entities using a sword and a bow
* The woodcutter who can use an axe

Furthermore, nobody should be able to activate a nether portal in the overworld.
Using ActionControl, it's possible to realize this jobs system pretty easily.
It's always recommended to create multiple config files that are responsible for controlling only one action instead of one big and cluttered file.

*disable-portals.json*::

    {
        # Only match players in the overworld
        "playerFilter": "{'location': {'world': {'dimension': {'name': 'overworld'}}}}",
        "actionRules": {
            "rightClickBlock": {
                "filter": {
                    "block": "{'state': {'type': 'minecraft:obsidian'}}",
                    "item": "{'type': 'minecraft:flint_and_steel'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            }
        }
    }
    
*farmer.json*::

    {
        # Only match players with the specified permission.
        # Note that you can use different permission names, these are just examples.
        "playerFilter": "{'permissions': {'actioncontrol.group.farmer': true}}",
        "actionRules": {
            "rightClickBlock": {
                "filter": {
                    "block": "{'state': {'type': 'minecraft:farmland'}}",
                    "item": "{'type': 'minecraft:wheat_seeds'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            },
            "breakBlock": {
                "filter": {
                    "block": "{'state': {'type': 'minecraft:wheat'}}"
                },
                "response": {
                    "noMatch": "deny"
                }
            }
        }
    }
    
*miner.json*::

    {
        "playerFilter": "{'permissions': {'actioncontrol.group.miner': true}}",
        "actionRules": {
            "leftClickBlock": {
                "filter": {
                    # We're using regular expressions here to make the matcher shorter. 
                    "item": "{'type': r'minecraft:.+?_pickaxe'}"
                    # If you don't know regular expressions, you can create the same effect using a different approach:
                    # "item": "{'type': 'minecraft:wooden_pickaxe' | 'minecraft:stone_pickaxe' | 'minecraft:iron_pickaxe' | 'minecraft:golden_pickaxe' | 'minecraft:diamond_pickaxe'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            }
        }
    }
    
*hunter.json*::

    {
        "playerFilter": "{'permissions': {'actioncontrol.group.hunter': true}}",
        "actionRules": {
            "rightClickEntity": {
                "filter": {
                    # We're using regular expressions again.
                    "item": "{'type': r'minecraft:.+?_sword'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            },
            "useItem": {
                "filter": {
                    "item": "{'type': 'minecraft:bow'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            }
        }
    }
    
*woodcutter.json*::

    {
        "playerFilter": "{'permissions': {'actioncontrol.group.woodcutter': true}}",
        "actionRules": {
            "leftClickBlock": {
                "filter": {
                    # We're using regular expressions again.
                    "item": "{'type': r'minecraft:.+?_axe'}"
                },
                "response": {
                    "noMatch": "deny"
                }
            }
        }
    }

.. note:: These examples files contain comments. Since comments are not supported in json, these config files won't work with the comments in them. But HOCON support is coming soon(tm)!
    
Now you just have to assign the permissions to each group.
And here you have it, a fully working RPG system implemented by just using a single plugin.
Of course this is a fairly basic RPG system but it can be extended in any way to fit your needs.
And if you want to do something entirely different, you can do that too!