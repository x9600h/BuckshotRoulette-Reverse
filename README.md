# BuckshotRoulette scripts reverse

## 📜 Overview
Reverse engineering buckshot roulette and script analysis using gdre tools

![изображение](https://github.com/user-attachments/assets/d692d34e-b785-4631-83c1-4054dd344980)

### 🛠️ DeathManager.gd

As we can see the realisation of death is very primitive and it is not worth explaining. (Developer is good in naming variables) 

```gd
var shitIsFuckedUp = false
func Kill(who : String, trueDeath : bool, returningShotgun : bool):
  ...
  if (shotgunShooting.roundManager.health_player == 1): shitIsFuckedUp = true
  ...
```
You may also notice the steam achievement unlocks when you reach 1000000 score

![изображение](https://github.com/user-attachments/assets/7db28f89-b0de-423b-b678-790b1d3cd891)

### 🛠️ InteractionManager.gd

In the InteractionManager script we can see objects such as bathroom door, backroom door, latches left\right, which I did not personally see during the game, most likely they are hidden.

```gd
func InteractWith(alias : String):
    ...
		"bathroom door":
			intro.Interaction_BathroomDoor()
		"backroom door":
			intro.Interaction_BackroomDoor()
		"heaven door":
			heaven.Fly()
		"latch left":
			brief.OpenLatch("L")
		"latch right":
			brief.OpenLatch("R")
    ...
```

### 🛠️ IntroManager.gd

We'll get an achievement if we wait in the bathroom longer than 60 seconds
```gd
var counting = false
var count_current = 0
var count_max = 60
var fs1 = false
func _process(delta):
	if (counting): CountTimer()
	
func CountTimer():
	if (counting): count_current += get_process_delta_time()
	if (count_current > count_max && !fs1):
		ach.UnlockAchievement("ach14")
		fs1 = true

func Interaction_BathroomDoor():
	...
	counting = true
	...
```

### 🛠️ RoundManager.gd

From this code it is clear that the number of live rounds = 1 to half of all rounds. So the number of blank rounds will be either greater than or equal to the number of live rounds.

Number of items from 1 to 5. 

Health points from 2 to 4
```gd
var curhealth = 0
func GenerateRandomBatches():
	for b in batchArray:
		for i in range(b.roundArray.size()):
			b.roundArray[i].startingHealth = randi_range(2, 4)
			curhealth = b.roundArray[i].startingHealth
			
			var total_shells = randi_range(2, 8)
			var amount_live = max(1, total_shells / 2)
			var amount_blank = total_shells - amount_live
			b.roundArray[i].amountBlank = amount_blank
			b.roundArray[i].amountLive = amount_live
			
			b.roundArray[i].numberOfItemsToGrab = randi_range(2, 5)
			b.roundArray[i].usingItems = true
			var flip = randi_range(0, 1)
			if flip == 1: b.roundArray[i].shufflingArray = true

```


### 🛠️ ShellLoader.gd

Maximum trivial ShellLoader script. Number of rounds = number of blanks + number of live rounds

```gd

func LoadShells():
...
	var numberOfShells = roundManager.roundArray[roundManager.currentRound].amountBlank + roundManager.roundArray[roundManager.currentRound].amountLive
	for i in range(numberOfShells):
		speaker_loadShell.play()
		animator_dealerHandRight.play("load single shell")
		if(roundManager.shellLoadingSpedUp): await get_tree().create_timer(.17, false).timeout
		else: await get_tree().create_timer(.32, false).timeout
		pass
...
```
### 🛠️ ShellSpawner.gd

Another maximally primitive implementation of shell mixing. The shuffle function is used, which looks like this

![изображение](https://github.com/user-attachments/assets/b49dfa32-ef61-4ae7-89bb-71813429907b)

```gd
func SpawnShells(numberOfShells : int, numberOfLives : int, numberOfBlanks : int, shufflingArray : bool):
	#DELETE PREVIOUS SHELLS
	for i in range(spawnedShellObjectArray.size()):
		spawnedShellObjectArray[i].queue_free()
	spawnedShellObjectArray = []
	
	#SETUP SHELL ARRAY
	sequenceArray = []
	tempSequence = []
	ai.sequenceArray_knownShell = []
	for i in range(numberOfLives):
		tempSequence.append("live")
	for i in range(numberOfBlanks):
		tempSequence.append("blank")
	if (shufflingArray):
		tempSequence.shuffle()
	for i in range(tempSequence.size()):
		sequenceArray.append(tempSequence[i])
		ai.sequenceArray_knownShell.append(false)
		pass
```

### 🛠️ StatueManager.gd

If we type in the name **vellon**, we can see some kind of statue

```gd
func CheckStatus():
	if (rm.playerData.playername == " vellon"):
		cup.visible = false
		statue.visible = true
```

> [!CAUTION]
> - **I have only described how some of the scripts work, for a complete study you can explore the scripts that I attached to this post**
> 
> - **Game version:**
> 08.11.2024

