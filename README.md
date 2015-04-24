A bashing script Achaea
=======================

What's this?
------------

While bashing (or hunting NPCs) in Achaea is not overly complicated, it is sometimes useful to handle targets in a certain
order instead of going through the seemingly random list from top to bottom. Different types of targets in an area make
bashing even more cumbersome. To avoid having to target specific NPCs by number (which is error prone and slow), this project
was born.

Requirements
------------

- Mudlet
- gmcp enabled
- svo, WunderSys or a queuing system with a [custom plugin](#support-for-other-systems)

Downloads and Releases
----------------------

The bashing script can be downloaded on the [github release page of the project](https://github.com/keneanung/Bashing/releases).

Stable releases are versioned with the versioning scheme `vXX.XX` where each XX stands for a number. The latest official
stable release has a green tag on the left side.

Releases with an orange tag are pre-releases for testing. *Those are automatically generated and not guaranteed to work.*
Additional to a numeric version number, the versioning scheme contains a short commit hash and a branch name. If you notice a
problem with a pre-release, please include this information in your bug report.

Quickstart
----------

1. Download the Bashing.mpackage
2. Import the package into Mudlet
3. If using svo: Deactivate or delete the keybinding of F2 that comes with svo
4. If using a custom system: [setup the system table](#support-for-other-systems) and configure `kconfig basing system <name>`
5. Use the alias `kconfig bashing toggle` to enable the script
6. Start killing things. Acceptable targets must be killed at least once in an area to register them with the bashing script.
   The basher will use the "target" variable or the in game target as a fallback, if there is no item in the prio list.
7. Keep bashing away using the F2 keybinding to work yourself down the list.

Priority management
-------------------

New acceptable targets are always added to the end of a priority list of an area. To change this, you will need to change the
order manually.

Use `kconfig bashing prios` to bring up the list of areas with priority lists. You can then click on an area name to bring
the priority list for that area. To filter the list of areas, you can use `kconfig bashing prios <partial area name>`. If
only one match is found, the list for that area is shown instead.

By using `kconfig bashing prios <area name>` or clicking on an area name in the area list, the script will bring up the
priority list for the area. If you are in that area and have one of these denizen in your room, the script will order them
from top to bottom. That means the topmost NPC type has the highest priority and the one at the lower end of the list the
lowest priority.

To change the position of an item in the list, click the `(vv)` and `(^^)` arrows to lower or raise the priority
respectively. You can also click on the `(DD)` to delete that NPC as an acceptable target.

Setting custom attacks
----------------------

The system supports the setting of custom attack command. To do so, use the alias `kconfig bashing attackcommand <your command>`. You can use the slash (`/`) to separate multiple commands that should be used at once. For example 2 handed knights might want to use `kconfig bashing attackcommand battlefury focus speed/kill`.

Shield handling
---------------

For classes with a quick and easy way to handle shielding NPCs, the basher has a very simple way to use it.

First turn the auto raze option on, using the alias `kconfig bashing raze`. This will enable switching the command used when
the denizen you attack shields. Additionally you need to configure the command used for auto razing. That can be done with
`kconfig bashing razecommand <command>`. You may use `/` as a separator for multiple commands. Use `&tar` as the place holder
for the target, if it needs to be somewhere within the command.

Congratulations, you will now raze the shield of NPCs whenever needed.

Fleeing
-------

The basher supports 2 ways to flee from dire situations, be it a stronger NPC entering your room, a tell you need to
concentrate to answer or luring NPCs into another room. The basher will always try to recognize the direction you entered a
room from and flee into that direction. *If that exit is not available, it will use the direction it recorded before.* It
will issue a warning in that case. You can use the alias `flee <command>` to set that manually.

The first way is the manual way. Simple press the keybinding `F3` to stop attacking and move ASAP.

The second way is automated and may be activated, if the **average** damage you have taken in this room between 2 attacks is
higher than you have at the next attack with an configurable threshold. The basher will also issue a warning, if average the
damage between 2 attacks is higher than your current health and another threshold. You can enable and disable that with the
alias `kconfig bashing autoflee` and the configure thresholds with `kconfig bashing fleeat <health amount>` and
`kconfig bashing warnat <health amount>`.

You may specify values as a flat amount, a percentage of the maximum health (ending the config value with a `%`) or as a
multiple of the damage taken in the room (ending the config value with a `d`) as the security threshold (amount of health
left after subtracting the current damage from the current health).

Configuration
-------------

The current configuration can be shown with the alias `kconfig bashing`. All items in red are clickable and will either
toggle the item or set the alias to the command line, so you only need the add the value you want to set.

Scripting
---------

Some example scripts can be found in the [plugin repository](https://github.com/keneanung/BashingPlugins).

### Events ###
The script fires two events that can be used to extend the bashing script.

The `keneanung.bashing.targetList.changed` event gets raised whenever anything in the list changes. That may be a new target
gets added, a target removed or the list got reordered. To access the current target list, use
`keneanung.bashing.targetList`.

The `keneanung.bashing.targetList.firstChanged` event is raised whenever the first item of the target list changes. That
means a new taget is used. This event also gives the new target as argument to the event handlers. To access the first item
of the target list directly, you can access `keneanung.bashing.targetList[1]`.

### Plugins ###
Additions to the script can now be loaded in two ways:
- register to the `keneanung.bashing.loaded` event and run your integration logic there. Use this approach if you need to
   load a package/xml due to triggers/aliases/other scripts
- the user can specify lua files that should be run after the bashing script is loaded. These scripts can be anywhere mudlet
   can reach them. The additional files are run using the lua `dofile`.

### Support for other systems ###
A special case of plugins is the support of custom systems.

While two often used systems are supported by the bashing script, there are a multitude of other systems (private and public)
out there. To allow integration of the bashing script into these systems, an interface was designed.

To interface the system, create a table with the following structure:

~~~~~~~
{
	startAttack = function()
		-- code that should be run, whenever the user wants to start attacking.
	end,

	stopAttack = function()
		-- code that should be run, whenever the user (or system) wants to stop attacking
	end,

	flee = function()
		-- code that is run when fleeing is needed
	end,

	warnFlee = function(avg)
		-- code that prints a warning at the "warn" threshold. The variable "avg" contains the
		-- average damage taken between attacks
	end,

	notifyFlee = function(avg)
		-- code that prints a warning at the "flee" threshold. The variable "avg" contains the
		-- average damage taken between attacks
	end,

	handleShield = function()
		-- code that is called, whenever the current target uses the shield tattoo
	end,

	setup = function()
		-- code that is run whenever the user connects to Achaea or the user configures the basher
		-- to use this system
	end,

	teardown = function()
		-- code that is run whenever the user switches from this system to another. Should be used
		-- to undo actions of setup()
	end,
}
~~~~~~~

This table can be added to the `keneanung.bashing.systems` table, using your system name as a key. The user must then
configure the basher to use the system by choosing the name *exactly* like the key you used to add the interface table.

Some further hints:
- If your queueing supports running functions instead of sending things directly to the mud, queue the
   `keneanung.bashing.nextAttack()` function and keep similar the svo implementation
- If your queueing uses the Achaean server side queue, keep close to the WunderSys implementation
- If your queueing is neither of those two possibilities, think about a way you can register attacks. If you have a way, you
   can check the WunderSys implementation for things needed to flee.


Acknowledgements
================

Tool creators
-------------

- GitHub user @bradrhodes for his [GithubDocSync project](http://bradrhodes.github.io/GithubDocSync/)
- Webtoolkit for their [Base64 Javascript implementation](http://www.webtoolkit.info/javascript-base64.html) (used in
  GithubDocSync)
- Github user @chjj for the [marked Project](https://github.com/chjj/marked) (used in GithubDocSync)
