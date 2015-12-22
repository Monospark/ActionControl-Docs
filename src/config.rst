=============
Configuration
=============

The config file of ActionControl allows you to .
It can be found at ``config/actioncontrol/config.json``.
If the config file is not present, an example config will be generated.

Structure
=========

The general structure of the config can be seen here::

    {
        "categories": {
            "<category1>": {
                "<actionRule1>": {
                    ...
                },
                "<actionRule2>": {
                    ...
                },
                "<actionRule3>": {
                    ...
                }
            },
            "<category2>": {
                "<actionRule4>": {
                    ...
                },
                "<actionRule5>": {
                    ...
                }
            }
        }
    }
  
A category is a collection of action rules.
An action rule specifies how to react on an action performed by a player.
For example, the action rule ``placeBlock`` can deny certain players to place certain blocks.
The categories that contain action rules can then be assigned to specific players or groups using a Permissions plugin by using the permission ``actioncontrol.category.<categoryName>``.

Action Rules
============

As mentioned previously, action rules specify how to react on an action.
Action rules can be divided into two types, **simple action rules** and **complex action rules**.
Currently, only simple action rules exist, because ActionControl is still in an early development stage.

Simple Action Rules
-------------------

Simple action rules have the following structure::

    "<actionRuleName>": {
        "response": "allow|deny",
        "filter": [{
            ...
        },
        {
            ...
        }]
    }

The ``response`` property defines how to react when the action matches the ``filter`` property.
The ``filter`` property is different for every action rule, since actions are different.
A good example for this are the ``breakBlock`` and ``placeBlock`` action rules::

    "breakBlock": {
        "response": "allow|deny",
        "filter": [{
            "blockIds": [<blockId1>, <blockId2>, ...],
            "toolIds": [<itemId1>, <itemId2>, ...]
        },
        {
            "blockIds": [<blockId3>, <blockId4>, ...],
            "toolIds": [<itemId3>, <itemId4>, ...]
        }]
    },
    "placeBlock": {
        "response": "allow|deny",
        "filter": {
            "blockIds": [<blockId1>, <blockId2>, ...],
        }
    }
    
As you can see, the filters differ since the actions differ as well.
Action rules can also have multiple filters, which you can see at the ``breakBlock`` filter property above.

List of Action Rules
--------------------

Minecraft IDs
=============

As seen previously, you have to specify some kind of IDs for every action rule.
These IDs can be block or item IDs, but also entity IDs and even enchantment IDs.

Blocks and Items
----------------

To specify a block or item, you need the ID of it.
These can be found at the `Minecraft ID list <http://minecraft-ids.grahamedgecombe.com/>`__.
For example, the ID of dirt is ``minecraft:dirt``.
When specifying a vanilla block, you can also omit the ``minecraft`` prefix, so the ID ``dirt`` is valid as well.

However, there are some blocks and items that have variants like ``minecraft:planks`` or ``minecraft:coal``.
This means that if you write ``minecraft:planks`` you implicitly include ``minecraft:planks:1`` (spruce planks), ``minecraft:planks:2`` (birch planks), etc.
You can avoid this by adding a data value to the ID, for example ``minecraft:coal:0`` for coal and ``minecraft:coal:1`` for charcoal.

Furthermore, you can also use data values ranges like ``minecraft:planks:1-3`` to include planks with the data values 1,2 and 3.

Entities
--------

To specify an entity, you need the ID of the entity.
These can be found at the `Minecraft entity ID list <http://minecraft-ids.grahamedgecombe.com/entities>`__.
Since there are no additional data values for entities, specifying entities is pretty straightforward compared to blocks and items.

Enchantments
------------

Already implemented, but currently not used.



Putting It All Together
=======================

Here is an example that uses all of the previously covered features to realize a small RPG system in which a player can accept one of four possible jobs:

* The farmer who can plant or harvest crops
* The miner who can use a pickaxe
* The hunter who can attack entities using a sword and a bow
* The woodcutter who can use an axe

Furthermore, there should be some things that nobody should be able to do like destroying cacti, because why not.
Using ActionControl, it's possible to realize this jobs system pretty easily.
Here is how to do it::

    {
        "categories": {
            "all": {
                "breakBlock": {
                    "response": "deny",
                    "filter": {
                        "blockIds": ["cactus"]
                    }
                }
            },
            "noFarmer": {
                "rightClickBlock": {
                    "response": "deny",
                    "filter": {
                        "blockIds": ["farmland"],
                        "itemIds": ["wheat_seeds", "pumpkin_seeds", "melon_seeds"]
                    }
                },
                "breakBlock": {
                    "response": "deny",
                    "filter": {
                        "blockIds": ["wheat", "melon_block", "melon_stem", "pumpkin", "pumpkin_stem"]
                    }
                },
            },
            "noMiner": {
                    "response": "deny",
                    "filter": {
                        "blockIds": "*",
                        "toolIds": ["wooden_pickaxe", "diamond_pickaxe", "golden_pickaxe", "iron_pickaxe", "stone_pickaxe"]
                    }
            },
            "noHunter": {
                "useItem": {
                    "response": "deny",
                    "filter": {
                        "itemIds": ["bow", "wooden_sword", "stone_sword", "iron_sword", "golden_sword", "diamond_sword"]
                    }
                },
                "leftClickEntity": {
                    "response": "deny",
                    "filter": {
                        "itemIds": ["bow", "wooden_sword", "stone_sword", "iron_sword", "golden_sword", "diamond_sword"],
                        "entityIds": "*"
                    }
                },
            },
            "noWoodcutter": {
                    "response": "deny",
                    "filter": {
                        "blockIds": "*",
                        "toolIds": ["wooden_axe", "diamond_axe", "golden_axe", "iron_axe", "stone_axe"]
                    }
            }
        }
    }
    
Now you just have to assign the permissions to each group.

The ``actioncontrol.category.all`` permission must be assigned to the parent group of all groups to apply it to every group.

The other categories have to be assigned to their respective groups as well.
For example, the farmer group gets the ``actioncontrol.category.noMiner``, ``actioncontrol.category.noHunter`` and ``actioncontrol.category.noWoodcutter`` permission.

There you have it, a fully working RPG system implemented by just using a single plugin.