Enums
=====

QuestLine.Event
---------------

`string:"event"` `readonly`

Objective triggered by a Roblox event.

|Parameter|Type             |Default     |Description
|--------:|:---------------:|:----------:|:----------
|  *event*|`RBXScriptSignal`|*[required]*| A roblox signal.
|  *count*|`number`         |1           | Expected trigger count.
|*compare*|`function`       |*see below* | A custom compare function.

``` lua
-- Knock 3 times
local doorClick = workspace.Door.ClickDetector
myQuest:AddObjective(QuestLine.Event, doorClick.MouseClicked, 3)
```

#### The comparision function

The optional *compare* function receives the player as the first argument, followed by the results of the triggered event, and should return a boolean.

``` lua
-- Default comparision function
function compare(plr, arg) return plr == arg end
```

This is appropiate for events including:

* The OnServerEvent of a RemoteEvent.
* The Triggerd event of a ProximityPrompt.
* The MouseClick event of a ClickDetector.

QuestLine.Score
---------------

`string:"score"` `readonly`

Objective triggered by a leaderstat value.  Similar to QuestLine.Value, except this waits for the leaderstat to be available before connecting to the *Changed* event.

> Note: This does not construct leaderstats automatically.

By default, the leaderstat value must be `>=` to the given value.

|Parameter |Type    |Default     |Description
|---------:|:------:|:----------:|:----------
|*statName*|`string`|*[required]*| The name of the leaderstat to track.
|*amount*  |`number`|*[required]*| Value to consider complete.
|*operator*|`string`|">="        | The comparison operator to use.

``` lua
-- Score 10 points on leaderstats
myQuest:AddObjective(QuestLine.Score, "Points", 10)
```

QuestLine.Timer
---------------

`string:"timer"` `readonly`

A time-based objective.  The delay is measured in seconds.

The optional *step* parameter refers to the number of steps the objective is broken into.  A value of 

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*count*  |`number`|*[required]*| Number of seconds to wait.
|*steps*  |`number`|1           | Steps of progress counted.

``` lua
-- Wait 5 seconds, counting progress each second
myQuest:AddObjective(QuestLine.Timer, 5, 5)
```

QuestLine.Touch
---------------

`string:"touch"` `readonly`

An objective to touch a part.  Uses the *Touched* event of a basepart.

This is suitable for an objective to enter a certain area.

|Parameter  |Type      |Default     |Description
|----------:|:--------:|:----------:|:----------
|*touchPart*|`BasePart`|*[required]*| A touchable part within the workspace.

``` lua
-- Return to dropoff
myQuest:AddObjective(QuestLine.Touch, workspace.DropOff)
```

QuestLine.Value
---------------

`string:"value"` `readonly`

Objective based on an IntValue.  Uses the *Changed* event of the instance given.

By default, IntValue.Value must be `>=` to the given value.

| Parameter|Type      |Default     |Description
|---------:|:--------:|:----------:|:----------
|  *intVal*|`IntValue`|*[required]*| A reference to an IntValue.
|   *count*|`number`  |*[required]*| Value to consider complete.
|*operator*|`string`  |">="        | The comparison operator to use.

``` lua
-- Track kills
myQuest:AddObjective(QuestLine.Value, player.EnemiesKilled, 5)
```

Static Members
==============

QuestLine.interval
------------------

`number = 1.0`

Transition time between one objective and the next.  Measured in seconds.

Because an objective fires *OnProgress* for both zero and 100%, this provides a chance to update the player's gui before assigned the next objective.

QuestLine.new()
---------------

`QuestLine.new(questId:string, self:{any}?):QuestLine`

Creates a new questline.  This returns the parameter *self* (if supplied) with it's it's metatable set to *QuestLine*.

The *self* parameter is useful for storing properties about the questline.  This can later be accessed through one of the event handlers.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*questId*|`string`|*[required]*| A unique identifier for the quest.
|   *self*|`{any}` |{}          | A table of properties associated with the quest.

|Return     |Description
|:----------|:----------
|*QuestLine*| A new QuestLine.

``` lua
local myQuest = QuestLine.new("myQuestId", {...})
```

QuestLine.getQuestById()
------------------------

`QuestLine.getQuestById(questId:string):QuestLine`

Returns a quest created with the given *questId*.

Useful for accessing questlines between scripts.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*questId*|`string`|*[required]*| A unique identifier for the quest.

|Return     |Description
|----------:|:----------
|`QuestLine`| The quest identified by *questId*.

``` lua
local myQuest = QuestLine.getQuestById("myQuestId")
```

QuestLine.registerPlayer()
--------------------

`QuestLine.registerPlayer(player:Player, playerData:{})`

Registers a player with the quest system and loads the player's progress.

The data table is used to store the player's progress of each questline.  The key is a reference to a questline's *questId*.  The value stored is an integer representing progress.

|Parameter   |Type               |Default     |Description
|-----------:|:-----------------:|:----------:|:----------
|*player*    |`Player`           |*[required]*| The player to register.
|*playerData*|`{[string]:number}`|*[required]*| The player's progression table.

``` lua
-- start with new data or load from datastore
local playerData = {
    myQuestId = 0 -- Assign zero to auto-accept
}

QuestLine.registerPlayer(player, playerData)
```

QuestLine.unregisterPlayer()
----------------------

`QuestLine.unregisterPlayer(player:Player):{}`

Unregisters the player from the quest system and returns the player's progress.

The table returned should be saved to a datastore and passed along to QuestLine.registerPlayer when the player rejoins.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to add to the quest system.

|Return             |Description
|:------------------|:----------
|`{[string]:number}`| The player's progression table.

``` lua
-- save the returned data to datastore
local playerData = QuestLine.unregisterPlayer(player)
```

Public Methods
==============

AddObjective()
--------------

`self:AddObjective(objType:string, ...any):number`

Adds a new objective according to the given objective type.  Additional parameters are determined by the type of objective you wish to add.

This method returns the index of the objective within the questline.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*objType*|`string`|*[required]*| The desired objective type to construct.
|*...*    |`...any`|*[required]*| See the [objectives](#enums) for details.

|Return  |Description
|:-------|:----------
|`number`| The index of the created objective within the *Questline*.

``` lua
local index = myQuest:AddObjective(QuestLine.Touch, workspace.TouchPart)
```

Assign()
--------

`self:Assign(player:Player)`

Assigns a *player* to a questline.

Triggers *OnAccept* if the quest was previously unknown followed by *OnAssign*.

Finally, *OnProgress* is triggered.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to assign.

``` lua
myQuest:Assign(player)
```

Cancel()
--------

`self:Cancel(player:Player)`

Causes the *player* to cancel/fail the current questline.  Triggers the *OnCancel* event listener.

A quest can be re-assigned after being canceled, which will trigger the *OnAccept* event once more.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to cancel the quest on.

``` lua
myQuest:Cancel(player)
```

GetCurrentProgress()
--------------------

`self:GetCurrentProgress(player:Player):(number, number)`

Retrieves an objective's progress for a player.  This method returns two numbers.

The first number returned represents the player's progress according to the current objective.

The second number represents the index of the current objective within the questline.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to query.

|Return  |Description
|:-------|:----------
|`number`| The current progress of the objective.
|`number`| The index of the objective within the questline.

``` lua
local currentProgress, index = myQuest:GetCurrentProgress(player)
```

GetObjectiveValue()
-------------------

`self:GetObjectiveValue(index:number):number`

Retrieves an objective's total value.  This depends on the objective being indexed.

This can be used within an event handler to get a percentage of progress.

``` lua
function QuestLiine:OnProgress(player, progress, index)
    local percent = progress / self:GetObjectiveValue(index)
    print(player, "progressed to", ("%d%%"):format(percent * 100))
end
```

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*index*  |`number`|*[required]*| The index of the objective within the quest to query.

|Return  |Description
|:-------|:----------
|`number`| The objective's maximum progression.

``` lua
local value = myQuest:GetObjectiveValue(index)
```

GetProgress()
-------------

`self:GetProgress(player:Player):number`

Retrieves a player's progress for the questline.  This correlates to the value stored in the player's progress table.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to query.

|Return  |Description
|:-------|:----------
|`number`| The player's progress within the questline.

``` lua
local progress = myQuest:GetProgress(player)
```

IsAccepted()
------------

`self:IsAccepted(player:Player):boolean`

Checks if the quest is accepted by the player.  A quest is only accepted after being assigned, or after it has been canceled.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to query.

|Return   |Description
|:--------|:----------
|`boolean`| Determines if the quest is accepted.

``` lua
if myQuest:IsAccepted(player) then
    -- This is no surprise
end
```

IsCanceled()
------------

`self:IsCanceled(player:Player):boolean`

Checks if the quest is canceled for the player.

A questline that has been cancelled can be re-assigned.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to query.

|Return   |Description
|:--------|:----------
|`boolean`| Determines if the quest is canceled.

``` lua
if myQuest:IsCanceled(player) then
    -- Where did I go wrong?
end
```

IsComplete()
------------

`self:IsComplete(player:Player):boolean`

Checks if the *player* has completed the quest.

|Parameter|Type    |Default     |Description
|--------:|:------:|:----------:|:----------
|*player* |`Player`|*[required]*| The player to query.

|Return   |Description
|:--------|:----------
|`boolean`| Determines if the quest is complete.

``` lua
if myQuest:IsCompete(player) then
    -- Yeah, I did that!
end
```

Events
======

QuestLine.OnAccept()
--------------------

`QuestLine:OnAccept(player:Player)`

Called at the beginning of a quest and only when it's first initialized.  This can be used to give a player starter items specific to the quest.

|Parameter |Type    |Description
|---------:|:------:|:----------
|*player*  |`Player`| A reference to the player.

``` lua
function QuestLine:OnAccept(player)
    -- Give player a quest item
end
```

QuestLine.OnAssign()
--------------------

`QuestLine:OnAssign(player:Player)`

Called each time the player is assigned the questline.  This runs after it is accepted, or when a player reloads from a previous session.

Useful for creating gui elements needed to display a questlog.

|Parameter |Type    |Description
|---------:|:------:|:----------
|*player*  |`Player`| A reference to the player.

``` lua
function QuestLine:OnAssign(player)
    -- Run code upon assignment
end
```

QuestLine.OnCancel()
--------------------

`QuestLine:OnCancel(player:Player)`

Called only when a call to *Cancel()* has been made.  This can be used to fail a quest.

A failed questline is recorded in the player's progress table as a negetive number.

|Parameter |Type    |Description
|---------:|:------:|:----------
|*player*  |`Player`| A reference to the player.

``` lua
function QuestLine:OnCancel(player)
    -- Run code upon cancelation
end
```

QuestLine.OnComplete()
----------------------

`QuestLine:OnComplete(player:Player)`

Called when a player has completed a questline.

The value recorded in the player's progress table will be `>=` the questline's total value.

|Parameter |Type    |Description
|---------:|:------:|:----------
|*player*  |`Player`| A reference to the player.

```lua
function QuestLine:OnComplete(player)
    -- Run code upon completion
end
```

QuestLine.OnProgress()
----------------------

`QuestLine:OnProgress(player:Player, progress:number, index:number)`

Called when a player has made progress.  Based on the current objective.

For the first time, progress will be zero, and lastly, the progress will be equal to `myQuest:GetObjectiveValue(index)`.

|Parameter |Type    |Description
|---------:|:------:|:----------
|*player*  |`Player`| A reference to the player.
|*progress*|`number`| The objective's progress for the player.
|*index*   |`number`| The objective's index within the quest.

```lua
function QuestLine:OnProgress(player, progress, index)
    print(player.Name, "has progressed to", progress, "for objective", index)
end
```
