
Finally got around to writing this after getting the idea two years ago lol. 

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

## 1. Adding the New CanPartyLearnMove Function we'll be using

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):


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
+	ld [wPutativeTMHMMove], a
+	push de
+	farcall CanLearnTMHMMove
+	pop de
+.check
+	ld a, c
+	and a
+	jr nz, .yes
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

FieldMoveFailed:
```

## 2. TryCutOW


Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):
```diff
TryCutOW::
	ld de, ENGINE_HIVEBADGE
	call CheckEngineFlag
	jr c, .cant_cut
+;;;;;;;;;;;;;
+	ld a, HM_CUT
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .cant_cut
+;;;;;;;;;;;;;
+	ld d, CUT
+	call CheckPartyCanLearnMove
+	jr z, .yes
+;;;;;;;;;;;;;
	ld d, CUT
	call CheckPartyMove
	jr c, .cant_cut
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
## 3. TrySurfOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

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

	ld de, ENGINE_FOGBADGE
	call CheckEngineFlag
	jr c, .quit

+;;;;
+	ld a, HM_SURF
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .quit
+;;;;
+	ld d, SURF
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+;;;;
+
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
```diff
TryWaterfallOW::
	ld b,b
	ld de, ENGINE_RISINGBADGE
	call CheckEngineFlag
	jr c, .failed
+;;;;;;;;;
+	ld a, HM_WATERFALL
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .failed
+;;;;;;;;;
+	ld d, WATERFALL
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+;;;;;;;;;
	ld d, WATERFALL
	call CheckPartyMove
	jr c, .failed
+.yes
	call CheckMapCanWaterfall
	jr c, .failed
```

## 5. TryStrengthOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

```diff
TryStrengthOW:
	ld de, ENGINE_PLAINBADGE
	call CheckEngineFlag
	jr c, .nope

+;;;;;;;;;
+	ld a, HM_STRENGTH
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .nope
+;;;;;;;;;
+	ld d, STRENGTH
+	call CheckPartyCanLearnMove
+	and a
+	jr z, .yes
+;;;;;;;;;

	ld d, STRENGTH
	call CheckPartyMove
	jr c, .nope

+.yes
	ld hl, wBikeFlags
	bit BIKEFLAGS_STRENGTH_ACTIVE_F, [hl]
	jr z, .already_using
```

## 6. TryWhirlpoolOW

Edit [engine\events\overworld.asm](../blob/master/engine/events/overworld.asm):

```diff
TryWhirlpoolOW::
+;;;;;;;;;;;
+	ld a, HM_WHIRLPOOL
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .failed
+;;;;;;;;;;
+	ld d, WHIRLPOOL
+	call CheckPartyCanLearnMove
+	jr z, .yes
+;;;;;;;;;;;
	ld d, WHIRLPOOL
	call CheckPartyMove
	jr c, .failed

+.yes
	ld de, ENGINE_GLACIERBADGE
	call CheckBadge
	jr c, .failed

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


```diff
TryHeadbuttOW::
+	ld a, TM_HEADBUTT
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .no
+
+	ld d, HEADBUTT
+	call CheckPartyCanLearnMove
+	jr z, .can_use ; cannot learn headbutt
+
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


```diff
HasRockSmash:
	ld d, ROCK_SMASH
	call CheckPartyMove
+	jr nc, .yes
-	jr nc, .yes
+;;;;;;;;;;;;;
+	ld a, TM_ROCK_SMASH
+	ld [wCurItem], a
+	ld hl, wNumItems
+	call CheckItem
+	jr z, .no
+;;;;;;;;;;;;;
+	ld d, ROCK_SMASH
+	call CheckPartyCanLearnMove
+	jr z, .yes
+
+.no
	ld a, 1
	jr .done
+.yes
	xor a
	jr .done
+.done
	ld [wScriptVar], a
	ret
```

## 9. Editing the Pokemon Submenu

Edit [engine\pokemon\mon_submenu.asm](../blob/master/engine/pokemon/mon_submenu.asm):


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

## 17. Polished Crystal by Rangi / TM Flag Array Compatibility

## 18. New Pokecrystal16 by Vulcandth Compatibility

Let me know if you have any questions, you can find me in the Pret discord server.


