
Finally got around to writing this after getting the idea two years ago lol, nearly dying from an extremely rare autoimmune disease has a way of messing with your hobbies. So this code is relatively old, and though I've tested it recently, there may be problems I've missed. Feel free to DM me in the Pret Discord or just post the issues to the Main Pokecrystal channel! 

This code should be compatible with most other features. Edits will be needed to work with Pokecrystal16. Will add those to this tutorial when I can.

This tutorial will allow use of HMs and other Overworld Moves (Rock Smash, Headbutt, Sweet Scent, Dig, Teleport) without pokemon in the party needing to have learned the move, as long as they are capable of learning the move. By default, I also have code that checks for Badges required, and the prescense of the HM/TM of the corresponding move in the bag. These code checks are optional, so if you want an open world you can have it. 

For the TM overworld moves, my assumption is that you've already implemented [Infinitely reusable TMs](Infinitely-reusable-TMs) so that once you aquire the TMs, they are permanent in the bag. If you do not wish to have [Infinitely reusable TMs](Infinitely-reusable-TMs) or to tie the use of the TM Overworld moves to needing to have a copy of the TM in the bag, you can link them to badges if you'd like, or have no other requirements except having a pokemon capable of learning the move.

A major feature of this tutorial is adding location checks when populating the Pokemon Submenu, so that the submenu does not get cluttered with Overworld moves in situations where you can't even use them anyways.

Please Note: There is a graphical glitch if you try to clear a Whirlpool without the required badge, but the glitch seems to be harmless. Just keep this in mind, and let me know if there are any issues that come up! I'm very active on the Pret Discord.

## Contents

1. [Adding the New CanPartyLearnMove Function we'll be using](#1-adding-the-new-canpartylearnmove-function-well-be-using)
2. [TryCutOW](#2-trycutow)
3. [TrySurfOW](#3-trysurfow)
4. [TryWaterfallOW](#4-trywaterfallow)
5. [TryStrengthOW](#5-trystrengthow)
6. [TryWhirlpoolOW](#6-trywhirlpoolow)
7. [TryHeadbuttOW](#7-tryheadbuttow)
8. [HasRockSmash](#8-hasrocksmash)
9. [Editing the Pokemon Submenu](#9-editing-the-pokemon-submenu)
10. [New Submenu Functions](#10-new-submenu-functions)
11. [Flash](#11-flash)
12. [Fly](#12-fly)
13. [Sweet Scent](#13-sweet-scent)
14. [Dig](#14-dig)
15. [Teleport](#15-teleport)
16. [Softboiled and Milk Drink](#16-softboiled-and-milk-drink)
17. [OPTIONAL TM Flag Array or Polished Crystal Compatibility](#17-optional-tm-flag-array-or-polished-crystal-compatibility)
18. [OPTIONAL New Pokecrystal16 Compatibility](#18-optional-new-pokecrystal16-compatibility)

## 1. Adding the New CanPartyLearnMove Function we'll be using

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Starting off, this function mirrors a lot of other functions in the codebase that cycle through each Pokemon in the party. 
NOTE: It's important to remeber that the Move in question must be loaded into register ```d``` before calling this function!!!!
We get the species and skip the mon if the species is 0 (empty party slot/end of party) or if it's an egg. Once we have the species in ```wCurPartySpecies``` via ```ld [wCurPartySpecies], a```  we ```farcall CanLearnTMHMMove``` which is the function that's used when you try to teach a Pokemon a TM or HM, so it's perfect for our uses here. The result is given in register ```c``` which we check. 

If the mon we're currently checking can learn the Move in ```d```, we return with the index of the party mon in ```wCurPartyMon```, zero out register ```a``` and don't bother checking the rest of the party.

If the TM/HM/Move tutor check fails, we then check the Mon's Level-up Moveset. This code is actually adapted from ax6 and Vulcandth's Pokecrystal16, and we also use the same code in the mon submenu!

All we do is check to see if the pokemon is capabale of evolving, and if so, skip over the evolution data so we start at the Level up moves. If we find the move, we exit early. If we read a 0 for the Lvl-learned part of the entry, we know we've reached the end of the learnset, and exit because we obviously didn't find the move we're looking for.

If we fail to find a mon in the party that can learn the Move, the Carry Flag is set via ```scf``` before we return.

```diff
CheckPartyMove:
; Check if a monster in your party has move d.
...
.no
	scf
	ret

+CheckPartyCanLearnMove:
+; CHECK IF MONSTER IN PARTY CAN LEARN MOVE D
+	ld e, 0
+	xor a
+	ld [wCurPartyMon], a
+.loop
+	ld c, e
+	ld b, 0
+	ld hl, wPartySpecies
+	add hl, bc
+	ld a, [hl]
+	and a
+	jr z, .no
+	cp -1
+	jr z, .no
+	cp EGG
+	jr z, .next
+
+	ld [wCurPartySpecies], a
+	ld a, d
+; Check the TM/HM/Move Tutor list
+	ld [wPutativeTMHMMove], a
+	push de
+	farcall CanLearnTMHMMove
+	pop de
+.check
+	ld a, c
+	and a
+	jr nz, .yes
+; Check the Pokemon's Level-Up Learnset
+	ld b,b
+	ld a, d
+	push de
+	call OW_CheckLvlUpMoves
+	pop de
+	jr nc, .yes
+; done checking
+
+.next
+	inc e
+	jr .loop
+
+.yes
+	ld a, e
+	; which mon can learn the move
+	ld [wCurPartyMon], a
+	xor a
+	ret
+.no
+	scf
+	ret
+
+OW_CheckLvlUpMoves:
+; move looking for in a
+	ld d, a
+	ld a, [wCurPartySpecies]
+	dec a
+	ld b, 0
+	ld c, a
+	ld hl, EvosAttacksPointers
+	add hl, bc
+	add hl, bc
+	ld a, BANK(EvosAttacksPointers)
+	ld b, a
+	call GetFarWord
+	ld a, b
+	call GetFarByte
+	inc hl
+	cp 0
+	jr z, .find_move ; no evolutions
+	dec hl ; does have evolution(s)
+	call OW_SkipEvolutions
+.find_move
+	call OW_GetNextEvoAttackByte
+	and a
+	jr z, .notfound ; end of mon's lvl up learnset
+	call OW_GetNextEvoAttackByte
+	cp d
+	jr z, .found
+	jr .find_move
+.found
+	xor a
+	ret ; move is in lvl up learnset
+.notfound
+	scf ; move isnt in lvl up learnset
+	ret
+
+OW_SkipEvolutions:
+; Receives a pointer to the evos and attacks, and skips to the attacks.
+	ld a, b
+	call GetFarByte
+	inc hl
+	and a
+	ret z
+	cp EVOLVE_STAT
+	jr nz, .no_extra_skip
+	inc hl
+.no_extra_skip
+	inc hl
+	inc hl
+	jr OW_SkipEvolutions
+
+OW_GetNextEvoAttackByte:
+	ld a, BANK(EvosAttacksPointers)
+	call GetFarByte
+	inc hl
+	ret
+
FieldMoveFailed:
```

## 2. TryCutOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Most of the functions we're editing in ```/engine/events/overworld.asm``` will mirror each other. 

Not much is needed to implement our new ```CheckPartyCanLearnMove``` function. 

First, we need to move the ```CheckPartyMove``` function to the end of the sequence, since it simply checks in any mons in the Party have the Move loaded into ```d``` as one of their 4 learned moves. Instead of checking this first thing, we move it to after ```CheckPartyCanLearnMove```. So now the function flows like this whenever the player interacts with a Cut Tree:

Step 1) Check if we have Hivebadge. If not, don't bother checking anything else, just exit with failure.

Step 2) Check if we have the CUT HM in the bag. If not, don't bother checking anything else, just exit with failure.

Step 3) Check if a Pokemon in the Party can learn CUT. If yes, we are done and jump to ```.yes``` and the Cut Tree will be cut. If not, we fallthrough to Step 4.

Step 4) Check if a Pokemon in the Party knows CUT. If not, we fail, and the Cut Tree cannot be Cut. If yes, we fallthrough to ```.yes``` and the Cut Tree will be cut.

```diff
TryCutOW::
-	ld d, CUT
-	call CheckPartyMove
-	jr c, .cant_cut
-
+ ; Step 1
	ld de, ENGINE_HIVEBADGE
	call CheckEngineFlag
	jr c, .cant_cut
+ ; end of Step 1
+
+ ; Step 2
+	ld a, HM_CUT
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .cant_cut
+ ; end of Step 2
+
+ ; Step 3
+	ld d, CUT
+	call CheckPartyCanLearnMove
+	jr z, .yes
+ end of Step 3
+

+	ld d, CUT
+	call CheckPartyMove
+	jr c, .cant_cut
+.yes
	ld a, BANK(AskCutScript)
	ld hl, AskCutScript
	call CallScript
	scf
	ret

.cant_cut
	ld a, BANK(CantCutScript)
	ld hl, CantCutScript
	call CallScript
	scf
	ret
```

Technically, you COULD assume that no Pokemon could have an Overworld move in their moveset without being able to learn it, so Step 4 could be safely deleted if you're sure that it could never happen. I am leaving it in the interest of compatibility and testing reasons. But if you are confident you don't need this check, you could modify this and the rest of the functions to look like: 

```diff
TryCutOW::
...
; Step 3
	ld d, CUT
	call CheckPartyCanLearnMove
-	jr z, .yes
-
- ; Step 4
-	ld d, CUT
-	call CheckPartyMove
	jr c, .cant_cut
+ ; end of Step 3
- ; end of Step 4
-.yes
	ld a, BANK(AskCutScript)
	ld hl, AskCutScript
	call CallScript
	scf
	ret
```

We don't need the ```.yes``` label anymore so we also get rid of it.

Similarly, if you don't want to make the use of HM Overworld moves dependant on having a Badge or having the HM/TM in the bag, just delete those steps entirely. 

## 3. TrySurfOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Nothing too different from ```TryCutOW```, Surf just has additional checks at the beginning that we don't need to mess with. After those, we still apply the same 4-Step checks.


```diff
TrySurfOW::
; Checking a tile in the overworld.
; Return carry if fail is allowed.

; Don't ask to surf if already fail.
	ld a, [wPlayerState]
	cp PLAYER_SURF_PIKA
	jr z, .quit
	cp PLAYER_SURF
	jr z, .quit

; Must be facing water.
	ld a, [wFacingTileID]
	call GetTileCollision
	cp WATER_TILE
	jr nz, .quit

; Check tile permissions.
	call CheckDirection
	jr c, .quit

+ ; Step 1
	ld de, ENGINE_FOGBADGE
	call CheckEngineFlag
	jr c, .quit

+ ; Step 2
+	ld a, HM_SURF
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .quit
+
+ ; Step 3
+	ld d, SURF
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+
+ ; Step 4
	ld d, SURF
	call CheckPartyMove
	jr c, .quit
+.yes
	ld hl, wBikeFlags
	bit BIKEFLAGS_ALWAYS_ON_BIKE_F, [hl]
	jr nz, .quit
```
## 4. TryWaterfallOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Nothing special to note here. Same 4-Steps, moving the ```CheckPartyMove``` code to be the last check instead of the first.

```diff
TryWaterfallOW::
-	ld d, WATERFALL
-	call CheckPartyMove
-	jr c, .failed
+ ; Step 1
	ld de, ENGINE_RISINGBADGE
	call CheckEngineFlag
	jr c, .failed
+
+ ; Step 2
+	ld a, HM_WATERFALL
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .failed
+
+ ; Step 3
+	ld d, WATERFALL
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+
+ ; Step 4
+	ld d, WATERFALL
+	call CheckPartyMove
+	jr c, .failed
+.yes
	call CheckMapCanWaterfall
	jr c, .failed
```

## 5. TryStrengthOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Same thing for Strength, we move ```CheckPartyMove``` code to be Step 4 instead of the first thing checked.

```diff
TryStrengthOW:
-	ld d, STRENGTH
-	call CheckPartyMove
-	jr c, .nope
+; Step 1	
	ld de, ENGINE_PLAINBADGE
	call CheckEngineFlag
	jr c, .nope

+; Step 2
+	ld a, HM_STRENGTH
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .nope
+
+; Step 3
+	ld d, STRENGTH
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+
+; Step 4
+	ld d, STRENGTH
+	call CheckPartyMove
+	jr c, .nope
+
+.yes
	ld hl, wBikeFlags
	bit BIKEFLAGS_STRENGTH_ACTIVE_F, [hl]
	jr z, .already_using
```

## 6. TryWhirlpoolOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Whirlpool has more complicated functions surrounding ```TryWhirlpoolOW``` but according to my testing, they don't actually do much if anything. However, there is a problem SOMEWHERE, considering there's a small graphical bug with my code here. If you examine a Whirlpool without having the badge or the TM (either or both) the text box goes yellow then blue the next time you examine it. As far as I can tell, it doesn't do anything else besides this color thing. But if anyone finds the source of this bug, a solution or work around, please let me know or directly edit this tutorial if you're sure!

```diff
TryWhirlpoolOW::
-	ld d, WHIRLPOOL
-	call CheckPartyMove
-	jr c, .failed
+; Step 1
	ld de, ENGINE_GLACIERBADGE
	call CheckEngineFlag
	jr c, .failed
+
+; Step 2
+	ld a, HM_WHIRLPOOL
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .failed
+
+; Step 3
+	ld d, WHIRLPOOL
+	call CheckPartyCanLearnMove
+	jr z, .yes
+
+; Step 4
+	ld d, WHIRLPOOL
+	call CheckPartyMove
+	jr c, .failed
+
+.yes
	call TryWhirlpoolMenu
	jr c, .failed

	ld a, BANK(Script_AskWhirlpoolOW)
	ld hl, Script_AskWhirlpoolOW
	call CallScript
	scf
	ret

.failed
	ld a, BANK(Script_MightyWhirlpool)
	ld hl, Script_MightyWhirlpool
	call CallScript
	scf
	ret
```

## 7. TryHeadbuttOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Now we're done with the HMs and onto the TM Overworld moves! Nothing too different, except no more badge checks (unless you want to add a badge check, feel free!) 

```diff
TryHeadbuttOW::
+; Step 1
+	ld a, TM_HEADBUTT
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .no
+
+; Step 2
+	ld d, HEADBUTT
+	call CheckPartyCanLearnMove
+	jr z, .can_use ; cannot learn headbutt
+
+; Step 3
	ld d, HEADBUTT
	call CheckPartyMove
	jr c, .no
+.can_use
	ld a, BANK(AskHeadbuttScript)
	ld hl, AskHeadbuttScript
	call CallScript
	scf
	ret

.no
	xor a
	ret
```

## 8. HasRockSmash

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

Pretty Self explanitory at this point, you're an expert!

```diff
HasRockSmash:
-	ld d, ROCK_SMASH
-	call CheckPartyMove
-	jr nc, .yes
-; no
+; Step 1
+	ld a, TM_ROCK_SMASH
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .no
+
+; Step 2
+	ld d, ROCK_SMASH
+	call CheckPartyCanLearnMove
+	jr z, .yes
+
+ Step 3
+	ld d, ROCK_SMASH
+	call CheckPartyMove
+	jr nc, .yes
+.no
	ld a, 1
	jr .done
.yes
	xor a
	jr .done
.done
	ld [wScriptVar], a
	ret
```

## 9. Editing the Pokemon Submenu

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):

Don't be intimiated lol

```diff
GetMonSubmenuItems:
	call ResetMonSubmenu
	ld a, [wCurPartySpecies]
	cp EGG
	jr z, .egg
	ld a, [wLinkMode]
	and a
	jr nz, .skip_moves
-
+	call CanUseFlash
+	call CanUseFly
+	call CanUseDig
+	call Can_Use_Sweet_Scent
+	call CanUseTeleport
+	call CanUseSoftboiled
+	call CanUseMilkdrink
-
.skip_moves
	ld a, MONMENUITEM_STATS
	call AddMonMenuItem
	ld a, MONMENUITEM_SWITCH
	call AddMonMenuItem
	ld a, MONMENUITEM_MOVE
	call AddMonMenuItem
```

## 10. New Submenu Functions

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CheckMonCanLearn_TM_HM:
+; Check if wCurPartySpecies can learn move in 'a'
+	ld [wPutativeTMHMMove], a
+	ld a, [wCurPartySpecies]
+	farcall CanLearnTMHMMove
+.check
+	ld a, c
+	and a
+	ret z
+; yes
+	scf
+	ret
+
+CheckMonKnowsMove:
+	ld b, a
+	ld a, MON_MOVES
+	call GetPartyParamLocation
+	ld d, h
+	ld e, l
+	ld c, NUM_MOVES
+.loop
+	ld a, [de]
+	and a
+	jr z, .next
+	cp b
+	jr z, .found ; knows move
+.next
+	inc de
+	dec c
+	jr nz, .loop
+	ld a, -1
+	scf ; mon doesnt know move
+	ret
+.found
+	xor a
+	ret z
+
+CheckLvlUpMoves:
+; move looking for in a
+	ld d, a
+	ld a, [wCurPartySpecies]
+	dec a
+	ld b, 0
+	ld c, a
+	ld hl, EvosAttacksPointers
+	add hl, bc
+	add hl, bc
+	ld a, BANK(EvosAttacksPointers)
+	ld b, a
+	call GetFarWord
+	ld a, b
+	call GetFarByte
+	inc hl
+	cp 0
+	jr z, .find_move
+	dec hl
+	call MonSubMenu_SkipEvolutions
+.find_move
+	call MonSubMenu_GetNextEvoAttackByte
+	and a
+	jr z, .notfound ; end of mon's lvl up learnset
+	call MonSubMenu_GetNextEvoAttackByte
+	cp d ;MAKE SURE NOT CLOBBERED
+	jr z, .found
+	jr .find_move
+.found
+	xor a
+	ret z ; move is in lvl up learnset
+.notfound
+	scf ; move isnt in lvl up learnset
+	ret
+
+MonSubMenu_SkipEvolutions:
+; Receives a pointer to the evos and attacks for a mon in b:hl, and skips to the attacks.
+	ld a, b
+	call GetFarByte
+	inc hl
+	and a
+	ret z
+	cp EVOLVE_STAT
+	jr nz, .no_extra_skip
+	inc hl
+.no_extra_skip
+	inc hl
+	inc hl
+	jr MonSubMenu_SkipEvolutions
+
+MonSubMenu_GetNextEvoAttackByte:
+	ld a, BANK(EvosAttacksPointers)
+	call GetFarByte
+	inc hl
+	ret
```

## 11. Flash

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CanUseFlash:
+	ld de, ENGINE_ZEPHYRBADGE
+	ld b, CHECK_FLAG
+	farcall EngineFlagAction
+	ld a, c
+	and a
+	ret z ; .fail, dont have needed badge
+; Flash
+	farcall SpecialAerodactylChamber
+	jr c, .valid_location ; can use flash
+	ld a, [wTimeOfDayPalset]
+	cp DARKNESS_PALSET
+	ret nz ; .fail ; not a darkcave
+
+.valid_location
+	ld a, FLASH
+	call CheckMonKnowsMove
+	and a
+	jr z, .yes
+
+	ld a, HM_FLASH
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	ret nc ; hm isnt in bag
+
+	ld a, FLASH
+	call CheckMonCanLearn_TM_HM
+	jr c, .yes
+
+	ld a, FLASH
+	call CheckLvlUpMoves
+	ret c ; fail
+
+.yes
+	ld a, MONMENUITEM_FLASH
+	call AddMonMenuItem
+	ret
```

## 12. Fly

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CanUseFly:
+	ld de, ENGINE_STORMBADGE
+	ld b, CHECK_FLAG
+	farcall EngineFlagAction
+	ld a, c
+	and a
+	ret z ; .fail, dont have needed badge
+
+	call GetMapEnvironment
+	call CheckOutdoorMap
+	ret nz ; not outdoors, cant fly
+
+	ld a, FLY
+	call CheckMonKnowsMove
+	and a
+	jr z, .yes
+
+	ld a, HM_FLY
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	ret nc ; .fail, hm isnt in bag
+
+	ld a, FLY
+	call CheckMonCanLearn_TM_HM
+	jr c, .yes
+
+	ld a, FLY
+	call CheckLvlUpMoves
+	ret c ; fail
+.yes
+	ld a, MONMENUITEM_FLY
+	call AddMonMenuItem
+	ret
```

## 13. Sweet Scent

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+Can_Use_Sweet_Scent:
+	farcall CanUseSweetScent
+	ret nc ; .no_battle
+	farcall GetMapEncounterRate
+	ld a, b
+	and a
+	ret z ; .no_battle
+
+.valid_location
+	ld a, SWEET_SCENT
+	call CheckMonKnowsMove
+	and a
+	jr z, .yes
+
+	ld a, TM_SWEET_SCENT
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	ret nc ; .fail, tm not in bag
+
+	ld a, SWEET_SCENT
+	call CheckMonCanLearn_TM_HM
+	jr c, .yes
+
+	ld a, SWEET_SCENT
+	call CheckLvlUpMoves
+	ret c ; fail
+.yes
+	ld a, MONMENUITEM_SWEETSCENT
+	call AddMonMenuItem
+	ret
```

## 14. Dig

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CanUseDig:
+	call GetMapEnvironment
+	cp CAVE
+	jr z, .valid_location
+	cp DUNGEON
+	ret nz ; fail, not inside cave or dungeon
+
+.valid_location
+	ld a, DIG
+	call CheckMonKnowsMove
+	and a
+	jr z, .yes
+
+	ld a, TM_DIG
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	ret nc ; .fail ; TM not in bag
+
+	ld a, DIG
+	call CheckMonCanLearn_TM_HM
+	jr c, .yes
+
+	ld a, DIG
+	call CheckLvlUpMoves
+	ret c ; fail
+.yes
+	ld a, MONMENUITEM_DIG
+	call AddMonMenuItem
+	ret
```

## 15. Teleport

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CanUseTeleport:
+	call GetMapEnvironment
+	call CheckOutdoorMap
+	ret nz ; .fail
+	
+	ld a, TELEPORT
+	call CheckMonKnowsMove
+	and a
+	jr z, .yes
+
+	ld a, TELEPORT
+	call CheckLvlUpMoves
+	ret c ; fail
+.yes
+	ld a, MONMENUITEM_TELEPORT
+	call AddMonMenuItem	
+	ret
```

## 16. Softboiled and Milk Drink

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


```diff
+CanUseSoftboiled:
+	ld a, SOFTBOILED
+	call CheckMonKnowsMove
+	and a
+	ret nz
+	ld a, MONMENUITEM_SOFTBOILED
+	call AddMonMenuItem
+	ret
```


```diff
+CanUseMilkdrink:
+	ld a, MILK_DRINK
+	call CheckMonKnowsMove
+	and a
+	ret nz
+
+	ld a, MONMENUITEM_MILKDRINK
+	call AddMonMenuItem
+	ret
```

## 17. OPTIONAL TM Flag Array or Polished Crystal Compatibility

## 18. OPTIONAL New Pokecrystal16 Compatibility

Let me know if you have any questions, you can find me in the Pret discord server.


