Build #114 [0.8.1]:
- A completely new rewrite.  Fully compatible with x.7.x version plugins _if_ they do not dip into wrapper's internal methods and stick strictly to the previously documented API:
http://wrapper.benbaptist.com/docs/api.html
- Methods in the client/server (like sending packets) are different.  Plugins doing this will need to be modified. Using the wrapper permissions or other wrapper components directly (self.wrapper.permissions, etc) by plugins will be broken with this version.

API changes Summary:

[Minecraft]

- lookupName (uuid) Get name of player with UUID "uuid"
- lookupUUID (name) Get UUID of player "name" - both will poll Mojang or the wrapper cache and return _online_ versions
- ban code methods added (bannUUID, banName, banIp, pardonName, pardonUUID, pardonIp, isUUIDBanned, isIpBanned)

[Player]

- ".name" and ".uuid" are deprecated properties that reference ".username" and ".mojangUuid" respectively.
- added player .offlineUuid (offline server UUID) and .ipaddress (the actual IP from proxy) self variables.
- quicker isOp_fast() - similar to .isOP(), but does not read ops.json file each time.  For use initerative loops where re-reading a file is a performance issue.  Refreshes at each player login.
- refreshOps() - refresh isOp_fast dictionary.
- sendCommand(self, command, args) -Sends a command to the wrapper interface as the player.  Similar to execute, but will execute wrapper commands.  Example: `player.sendCommand("perms", ("users", "SurestTexas00", "info"))`
- self.clientboundPackets / serverboundPackets - contain the packet class constants being used by wrapper.  Usage example: `player.getClient().packet.send(player.clientboundPackets.PLAYER_ABILITIES,
"byte|float|float", (bitfield, self.fly_speed, self.field_of_view))` renaming these in the plugin code is fine:
```
pktcb = player.clientboundPackets
player.getClient().packet.send(pktcb.PLAYER_ABILITIES ...)
```
- setPlayerAbilities(self, fly) - pass True to set player in flying mode.
- added event "player.Spawn" - occurs after "player.login"; when the player actually spawns (i.e., when the player actaully sees something other than the "login, building terrain..").  This is a good place for plugins to start gathering information like player.position and such because it gives the proxy time to bather those items.  The "player.login" happens too soon for proxy to gather information on many player variables...

Broad-spectrum summary of changes:

[Re-vamped console interface]

- added ban commands (ban-ip, ban, pardon, pardon-ip)
- colorized the menus/helps (does not use log statements for display)
- minimal console player class added to wrapper.  Console has a fake "player" class that mimicks player methods and variables.  Presently, this allows the in-game wrapper menus and the console commands to share common methods, so that separate code is not needed to do things like... 
- Add /perms to console commands!  Add /playerstats also.. (the player.message() components of the in-game commands are passed to the console as colored print statements).

[wrapper structure in general]

- proxy split into submodules
 -- base.py - General functions, including various ban methods.
 -- mcpacket.py - contains general packet abstraction classes.  Defines critical-change version-to-protocol number references, start and stop points to define a given packet set, etc. Contains packet sets for all wrapper-supported minecraft versions.
 -- packet.py - old packet class; send/receive packets.
 -- clientconnection.py - old Client class.
 -- serverconnection.py - old proxy Server class (not the console server.py!)
- Utilities (/utils) submodule package set added as a place for generic high use or univeral code.
- Creation of /core group where most core wrapper items reside.  Base wrapper source folder just has `__init__.py` and `__main__.py`.
- log.py removed in favor of wrapper built-in loggin module
- trace level logging of packet parsing is available

features/other changes:

- wrapper handles all it's own ban code for the server.
- temporary bans are available (uses the "expires" field in the ban files!)
- logging defaults and configuration moved out of wrapper.properties to it's own loggin.json file.
- wrapper fully handles keepalives between client and server. technically, we are moving towards the separation of client and server that could allow future functions like server restarts that don't kick players, cleaner transfers to other server instances, and a "pre-login" lobby where player can interact with wrapper before wrapper logs the player onto the server...

ISSUES ADDRESSED:

Progresses towards:
- [Refactor Wrapper Version and Release Pipeline #299](https://github.com/benbaptist/minecraft-wrapper/issues/299)

Addresses in part:
- [Performance Updates #295](https://github.com/benbaptist/minecraft-wrapper/issues/295)
- [Python 3 #301](https://github.com/benbaptist/minecraft-wrapper/issues/301)
- [Python CPU usage creeps up over time #276](https://github.com/benbaptist/minecraft-wrapper/issues/276)

Fixes:
- [Fresh install of Wrapper.py in certain conditions does this thing #262](https://github.com/benbaptist/minecraft-wrapper/issues/262)
- [Proxy.py Rewrite #244](https://github.com/benbaptist/minecraft-wrapper/issues/244)
- [UUID set to False #227](https://github.com/benbaptist/minecraft-wrapper/issues/227)
- [Strange client.properties issue that disconnected all people #190](https://github.com/benbaptist/minecraft-wrapper/issues/190)
- [Web.py Client.runaction Action=Admin_stats dictionary problem #188](https://github.com/benbaptist/minecraft-wrapper/issues/188)
- [UUID-related error in console. #133](https://github.com/benbaptist/minecraft-wrapper/issues/133)

 
Builds - 112 - 113:
- experimental pre-releases of build #114

Build #111:
- Added dashboard.py - beginning work on Dashboard rewrite using Flask + SocketIO

Build #110:
- [pull request #269] Bug fixes by sasszem
- based on build 109 with all current pull requests as of 1/30/2016
- updated md5 to hashlib in a few places (md5 is deprecated)
- Updated the Player.chatbox event to allow the chat to modified.  Also upgraded it to handle UTF-8 and special
		characters in chat.
- added to api.minecraft: getTimeofDay(self, format=0)
		# 0 = ticks, 1 = Military, else = civilian AM/PM, return -1 if no one on or server not started
		returns actual world time of day  (changes in server.py and API/minecraft.py)
- Added Chat.py plugin example...
- ..that utilizes the new abilty of "player.chatbox" event to accept plugin changes to chat.
- Added clock.py example for getTimeofDay()
- Lots of changes to proxy.py... Added mcpkt.py file for packet number references.  Packets are now referenced (in the
		play mode sections) by names, not hardcoded packet numbers.
- Added file mcpkt.py, where packet definitions are stored for reference by proxy.py.
- Beefed up player.interact event some to allow blocking of "interaction events" like lava and water placement, shooting, etc

Build #109:
- [pull request #247] Bookmarks plugin by Cougar
- [pull request #248] Storage.py fixes, NBT slot reading/sending

Build #108:
- Disabled slot parsing for 0x30 until fixes can be made to NBT things

Build #107:
 Screw the release canidates. Just another regular build.
- [pull request #218/issue #210] Another, hopefully final fix for Wind0ze log issue
- [pull request #222] Add player.createsign event (signedit 0x12 packet)
- [pull request #219] Allow disabling plugins via plugin metadata
- [issue #221] api.minecraft getAllplayers filelock issue on Wind0ze
- Fixed spectator teleportation while using proxy mode
- Added support for Minecraft protocol 54/15w32c
- Fixed offline mode being broken

Build #106 [0.7.7 RC3]:
- [issue #210] More possible fixes for log-rotation issue on Windows
- [issue #214] Fixed slot packet not being parsed properly and causing random disconnectionss
- [issue #80] server-icon.png is now read with rb, to fix Windows compatibility
- [pull request #209/issue #131] - Make groups inherit other groups

- Plugin changes:
	- WorldEdit:
		- Fixed wand not functioning properly

Build #105 [0.7.7 RC2]:
I added a new ROADMAP.md file for keeping a nice, organized file on the future of Wrapper.py updates.

- [issue #80] server-icon.png is now loaded once prior to starting the server, to prevent file conflicts on Windows
- [issue #210] Experimental fix for log-rotation issue on Windows
- Disabled editing server.properties until fixed
- Fixed indentation inconsistencies in proxy.py

- Plugin changes:
	- WorldEdit:
		- Re-fixed double-click issue thingy or something [not entirely sure what the issue was]
		- No longer using player.execute - doesn't require op AND permission node

Build #104 [0.7.7 RC1]:
- Fixed Control+D crash
- Removed most debug printing stuff as it's mostly ready for primetime
- Fixed issue with new permissions methods not working at all

- Plugin changes:
	- WorldEdit:
		- Added //replacenear command
		- Added //extinguish command
		- Added /help entry for all WorldEdit commands

Build #103:
- SERIOUS bug fix: Wrapper.py spams "day changed, rotating logs..." and creates new log file every second
- Traversing cross-server is now far more efficient (uses respawn to different dimension, then switches to actual dimension)

Build #102:
- Added log rotation, and logs are now stored in logs/wrapper directory
- Default server jar is now set to 1.8.7 for wrapper.properties
- Fixed support for Minecraft 1.7.10 in proxy mode
- Fixed `/wrapper halt` command in-game
- API changes:
	- [issue #199] Added new methods for modifying player permissions [untested]:
		- player.setGroup(group)
		- player.setPermission(node, value=True) (value argument is optional, default is True)
		- player.removePermission(node)
		- player.removeGroup(group)
	- [issue #164] Implemented timer.tick event (finally!)

Build #101:
- Fixed crashes relating to packets 0x1a (again!) and 0x1e
- player.getPosition() now returns the following tuple format: (x, y, z, yaw, pitch) (removed onGround)

Build #100:
- Proxy mode improvements:
	- Fixed 1.7.10 servers not working due to changes in #98 (packet 0x2b being sent on a non-1.8 server)
	- Fixed skin settings not persisting when changing servers
	- Fixed status effects not disappearing when connected to a secondary server
	- Fixed client disconnecting from 0x1a packet
	- [issue #200] Fixed crash/chunks not loading in Nether and End
- API changes:
 - [*pull request #193/#194] player.getPosition() now returns the following tuple format: (x, y, z, onGround, yaw, pitch) [MAY BREAK EXISTING PLUGINS]

*Pull request was modified from original to better fit the API.  

Build #99:
- Added /lobby command for cross-servers
- [pull request #178] Fix player.setResourcePack
- Removed trace amounts of try:except statements in proxy code in an effort to reduce random issues that aren't being detected
	- I've also done some little fixes, and I think the proxy mode is a bit more stable now.
	- Translating more packets that involve entity IDs when cross-server to fix weird issues
		- Clothes can be changed, beds work properly, animations might work better, etc.

Build #98:
- Added new example plugins `teleport` and `home` by Cougar [pull requests #155 and #154 respectively]
- Removed Top 10 Players until lag can be fixed
- [issue #160] Fixed new players without a skin breaking parts of the Wrapper
- Cross-server improvements:
	- Fixed skins and duplicating players on tab list when traversing between Wrapper.py servers
	- Weather is now accurate
- [pull request #176] Parse PID 0x30 (Window Items)
- [pull request #174/#173] Fix issue #172

- Plugin changes:
	- Essentials:
		- [pull request #162] Assign default MOTD during login if it does not exist like /motd does

Build #97:
- Split plugin, command, and event code into separate respectively-named files/classes for cleaner code
- Fixed nobody being able to connect via proxy with oddly-formatted whitelist.json
- `/permissions groups [group] info` now shows usernames alongside UUID [surresttexas00/#145]
- `/permissions users [username] remove` sub-command added [surresttexas00/#148]
- Fixed `player.hasGroup` always returning None [issue #144]
- Fixed `/help` command showing two pages when only four items are listed [issue #141]
- Passing a list or tuple as the command name to api.registerCommand for easily registering multiple aliases for a command [issue #135]
- Commands are no longer case-sensitive [issue #135]
- Removed bolding on `/help` command groups [issue #121]
- Proxy config option `convert-player-files` is now False by default until fixed