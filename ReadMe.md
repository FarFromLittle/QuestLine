Getting Started
=========

QuestLine is a server-sided module script that aids in the creation, assignment, and tracking of a series of objectives.

The module itself does not include data storage or visual elements.

Instead, it offers a framework to create customized quest systems that are event-driven and easily maintained.

Creating QuestLines
-------------------

QuestLines are created with a call to *new*.  This requires a unique string used to store the questline within the system.

``` lua
local myQuest = QuestLine.new("myQuestId", { Title = "My First Quest" })
```

The optional second parameter allows you to specify a table for *self*.

After a questline is created, it can later be retrieved with a call to [`QuestLine.getQuestById()`](api.html#static-members-questlinegetquestbyid).

``` lua
local myQuest = QuestLine.getQuestById("myQuestId")
```

Adding Objectives
-----------------

A questline consists of one or more objectives.  Progress moves from one objective to the next until it's complete.

``` lua
myQuest:AddObjective(objType, ...any)
```

The first parameter refers to one of the objective types.  The rest are dependant on the type of objective being added.

There are a total five objective types.

| Type | Description
|-----:|:-----------
| [Event](api.html#enums-questlineevent) | A generic, event-based objective.
| [Score](api.html#enums-questlinescore) | An objective based on the value of a leaderstat.
| [Timer](api.html#enums-questlinetimer) | A time-based objective.
| [Touch](api.html#enums-questlinetouch) | A touch-based objective.
| [Value](api.html#enums-questlinevalue) | An objective based on the value of a given *IntValue*.

``` lua
-- Wait for player to activate
local prompt = workspace.Activate.ProximityPrompt
myQuest:AddObjective(QuestLine.Event, prompt.Triggered)

-- reach level 10 on leaderstats
myQuest:AddObjective(QuestLine.Score, "Level", 10)

-- wait 360 seconds (one hour), count whole minutes
myQuest:AddObjective(QuestLine.Timer, 360, 60)

-- Wait for player to touch a given part
myQuest:AddObjective(QuestLine.Touch, workspace.DropOff)

-- Continue after reaching 10 kills
myQuest:AddObjective(QuestLine.Value, player.EnemiesKilled, 10)
```

Each objective has it's own set of parameters.  Refer to the [api](api.html#enums) for a detailed explanation of each objective type.

Registering Players
-------------------

Players must first register with the system before questlines can be assigned.  A call to [register()](api.html#static-members-questlineregister) requires the *player* and a table containing their progress.

``` lua
QuestLine.registerPlayer(player, playerData)
```

Player progression is stored in a table under the key supplied by *questId*.  The value is stored as a integer.

Additionally, this table can be pre-populated with starter quests by assigning zero to an entry.

``` lua
local playerData = {
	myQuestId = 0 -- assign zero to auto-accept
}
```

Assigning QuestLines
--------------------

Once a player is registered, they are ready to be assigned quests.

``` lua
myQuest:Assign(player)
```

If no entry is found in the player's progress table, the [OnAccept()](api.html#events-questlineonaccept) callback will fire and `myQuestId = 0` will be added.

When a player is initially registered, all questlines not found to be complete, or canceled, will automatically be assigned.  This triggers the [OnAssign()](api.html#events-questlineonassign) callback.

Handling Progression
--------------------

Events are triggered using callbacks related to the various stages of progression.

Events are fired in the following order:

`OnAccept(player:Player)`
* Fires when a player is assigned a previously unknown questline.
  
``` lua
function QuestLine:OnAccept(player)
    print(player, "accepted", self)
end
```

`OnAssign(player:Player)`
* Fires each time a player is assigned the questline.
* This includes when a player resumes progress from a previous session.
  
``` lua
function QuestLine:OnAssign(player)
    print(player, "assigned", self)
end
```

`OnProgress(player:Player, progress:number, objIndex:number)`
* Triggers at each step of progression.
* The first event fires with `progress = 0`
* Lastly with `progress = myQuest:GetObjectiveValue(objIndex)`.
  
``` lua
function QuestLine:OnProgress(player, progress, index)
    print(player, "has", progress, "out of", self:GetObjectiveValue(index))
end
```

`OnComplete(player:Player)`
* Fires when a player has completed the questline.

``` lua
function QuestLine:OnComplete(player)
    print(player, "completed", self)
end
```

`OnCancel(player:Player)`
* Only triggered by a call to `myQuest:Cancel(player)`.
* Can be used to fail a questline.
* A canceled questline can be re-accepted.

``` lua
function QuestLine:OnCancel(player)
    print(player, "canceled", self)
end
```

A typical questline is managed by a global callback function.  Local callbacks can be defined on individual questlines, but you may need to make a call to the global one as well.

``` lua
function myQuest:OnComplete(player)
    QuestLine.OnComplete(self, player)
    -- Run myQuest code
end
```

The global callback will not run when a local one has been assigned.  This makes it necessary to do it manually.

Be aware that you can only set a callback once per context (global or local).
Setting it again will overwrite the previous behavior.

> Take note that both examples use a colon `:` when defining the method, which means *self* is implied.  However, when calling the global callback, a period `.` is used and *self* is passed along with the player.

Saving Player Data
------------------

When a player leaves the game, they need to be unregistered from the system.

``` lua
local playerData = QuestLine.unregisterPlayer(player)
```

This returns a simple table containing the player's progress to be saved in a datastore.

