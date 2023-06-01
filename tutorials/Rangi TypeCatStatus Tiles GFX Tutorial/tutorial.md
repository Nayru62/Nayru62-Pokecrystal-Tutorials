Credit Rangi and Polished Crystal
Credit Idain for helping too

The code for this feature was adapted from [Polished Crystal by Rangi](https://github.com/Rangi42/polishedcrystal).

This will change all instances of menus using Pokemon's Types or Pokemon's Statuses (Burn, Poison, Toxic, Sleep, Paralyze, Freeze, Faint) and change the text to custom gfx tiles made by Rangi. If you format the graphic's files the same way as this version, you can edit the tiles however you like.

This code should be compatible with any other features.
Small edits will need to be made for using Physical/Special Split, or using the Fairy Type (or any other custom Types).

## Baseline Information

You'll see where I describe some gfx tiles as "light" or "dark".

Normal Text tiles use 2 colors, White and Black. Their file type is translated from .png to .1bpp because the color coding only takes 1 Bit, Per Pixel (1BPP). 0 is white, 1 is black, etc. 1 bit of color.

Other Graphics tiles, can have 4 colors, and therefore are translated from .png to .2bpp (2 Bits Per Pixel, 0 is white, 1 is Light Grey, 2 is Dark Grey, 3 is Black = 2 bits of Color) Two of colors are always White and Black, the other two are denoted by two shades of gray, the "Light" shade and "Dark" shade. Each shade corresponds to a color defined in .pal (color palette files) or .asm files where we define color palettes. You can use any color for either shade of gray, the light/dark shade itself doesn't affect the actual color you choose, it's just a way to differentiate 4 different colors via White, Black, LightGray, DarkGray.

So why do we have a few .pngs of the Types and Stauses that are using Black where we will have a color?
Because we can change the default color palettes to use other colors instead of White or Black. But this will affect everything on the screen unless we properly denote the areas of the screen where we want to affect the color palettes.

This is done through the CGB Layouts. I highly recommend using and becoming familiar with the emulator/debugger [BGB](https://bgb.bircd.org/) as it has a robust VRAM viewer that helps you properly visualize each tile, where it's coming from, and what the address of the space of the screen its on, and which palette that screen area is currently using, as well as the palettes themselves and the RBG color values for when you want to match colors but dont know the values.

When we load the GFX files, we must load the tiles into VRAM, represented visually by the Tiles Grid, which you can see below in the middle image. Each Tile has a Value, from $00 to $7F. And, each Tile has a Memory Address. For example, the very bottom Tile on the very left, is Tile $70, and its address is $9700. The tile next to it on the right, is Tile $71, its address is $9710, Tile $72 is to the right of Tile $71, its Tile address is $9720, etc.

After the Tile is loaded into VRAM, we decide where it goes on the screen, and when. This is usually done in ```\engine\``` code via .asm files. These files are also where we apply or change the CGB Layout when we transition between screens or menus.

The Screen is broken down into a Grid, and each square has its own (Hardware? or VRAM?) Map Address, but it's much easier for us to refer to an individual space on the screen by specifying Grid coordinates (X,Y). We often use the function ```hlcoord X, Y``` to convert the X,Y coordinates to the Map Address, and by loading a Tile Value into the Map Address, we put that tile on the screen exactly where we want it.

The Screen is actually much larger than we are normally able to see, that's why there is so much empty/placeholder spaces in the first image below. We generally don't need to worry about this unused space, but it's important to remember it's there when calculating the Map Addresses.

The Colors are determined by the CGB Layout, which also essentially uses Grid coordinates to select areas of the Screen to apply a designated Palette to. For each square on the Screen, the Square has Attributes in addition to the Map Address, and the Tile Value. The Attributes is where the Palette chosen is stored in Memory, and this Attribute Table is stored in WRAM. 

## Advanced Information

By changing the Attributes for a square or a designated rectangle/area of the Screen (via the CGB Layout), we can also flip or rotate the Tile's orientation, but this is honestly really underused, even though by using it you can save a lot of space by using the same tile in different orientations instead of loading a different copy of each tile. 

By editing a Square's Attribute (via the CGB Layout), we can also tell it to use the 2nd VRAM bank of Tiles (VRAM1), instead of the default VRAM Bank (VRAM0). This can allow us to do some fancy tile layouts when space is tight. But it's easy to get confused until you get the hang of using this whole other VRAM bank. You can see how each Bank is equal in size, in the middle image below, there are roughly two seperate columns. The left column of Tiles is VRAM0, the default VRAM bank. The column to the right is VRAM1. That's why in the box that says Tile Address, you see 0:9000. The 0 in front of the colon denotes the Tile is in VRAM0 and not VRAM1. If the Tile was in VRAM1, the address would be 1:9000 and the Tile would appear in the column on the right. 

In the example images below, I am used a more advanced layout in my Pokedex, with many more custom tiles than just the Pokemon Type. So, I ended up loading the Type GFX Tiles (which in the example, Crobat, Poison & Flying) into VRAM1 instead of VRAM0. That's why the Type GFX Tiles appear in the right-side column of Tiles.

These are advanced concepts, but you should be aware, as we do eventually touch on and use these concepts, as they are both very powerful.

![bgb_VRAM_View_example1](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/6ab68e5b-3824-45a2-8f02-73e6e9385b49)
![bgb_VRAM_View_example2](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/4e1d9914-9cac-4ded-a5e7-52e458addd70)
![bgb_VRAM_View_example3](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/2182ef59-ffaf-45ee-b297-69b1627adba4)

## **This is a lot of information, so DON'T BE ALARMED IF IT DOESN'T IMMEDIATLEY MAKE SENSE!!!!!!!!**

**If you follow this tutorial to the Tee, using BGB and learning exactly how GFX works in detail isn't necessary.** But I highly recommend it, especially if you want to eventually edit the layouts of menus, the battle screen, the pokedex, stats screen, etc. If you understand how the GFX tiles are implemented and how the CGB layout controls and applies the palettes, you can change the current layout to your personal preference.

## 1. Add GFX Files

- [gfx\battle\types.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/cc2ef5a1-820c-4d9d-8bcf-9548eea1cc89)
- [gfx\stats\types_dark.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/b384c952-0c71-4211-843d-054989bdd09a)
- [gfx\stats\types_light.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/60fe4053-d82c-4b27-b633-f587a5412357)
- [gfx\battle\categories.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/f65ce911-ad0f-4f9f-887f-fda26983c82a)
- [gfx\battle\status.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/be3a54dd-84b2-40e2-a33c-39741ff36fed)
- [gfx\battle\status-enemy.png](https://github.com/Nayru62/Nayru62-Pokecrystal-Tutorials/assets/110363717/4eae2b97-a689-43dd-9354-9f51587a8d66)

- Create ```gfx\rangi_gfx.asm```:
```
TypeIconGFX::
INCBIN "gfx/battle/types.1bpp"

CategoryIconGFX::
INCBIN "gfx/battle/categories.2bpp"

StatusIconGFX:
INCBIN "gfx/battle/status.2bpp"

EnemyStatusIconGFX:
INCBIN "gfx/battle/status-enemy.2bpp"

TypeLightIconGFX::
INCBIN "gfx/stats/types_light.2bpp"

TypeDarkIconGFX::
INCBIN "gfx/stats/types_dark.2bpp"
```
- Create ```gfx\types_cats_status_pals.asm```:

NOTE: Remove the Palette for FAIRY if you're not using Fairy Type.
```
+StatusIconPals:
; OK
	RGB 31, 31, 31
; PSN
	RGB 27, 11, 27
; PAR
	RGB 30, 20, 00
; SLP
	RGB 17, 17, 17
; BRN
	RGB 31, 08, 02
; FRZ
	RGB 09, 18, 31
; FNT
	RGB 31, 31, 31
; TOX
	RGB 27, 06, 28

CategoryIconPals:
; PHYSICAL
	RGB 31, 28, 00
	RGB 27, 04, 02
; SPECIAL
	RGB 27, 31, 31
	RGB 00, 14, 29
; STATUS
	RGB 31, 31, 31
	RGB 21, 21, 14

TypeIconPals:
; NORMAL
	RGB 21, 21, 14
; FIGHTING
	RGB 27, 04, 02
; FLYING
	RGB 22, 17, 30
; POISON
	RGB 22, 07, 19
; GROUND
	RGB 29, 24, 12
; ROCK
	RGB 24, 20, 07
; BUG
	RGB 21, 23, 06
; GHOST
	RGB 15, 11, 18
; STEEL
	RGB 23, 23, 25
; FIRE
	RGB 31, 15, 04
; WATER
	RGB 11, 18, 30
; GRASS
	RGB 11, 25, 11
; ELECTRIC
	RGB 31, 24, 06
; PSYCHIC
	RGB 31, 09, 15
; ICE
	RGB 16, 27, 27
; DRAGON
	RGB 15, 07, 31
; DARK
	RGB 15, 11, 09
; FAIRY
	RGB 31, 20, 29
; UNKNOWN T
	RGB 13, 19, 19
```

## 2. Load Custom Tiles

This is what we are using ```gfx\rangi_gfx.asm``` for, we need to edit ```main.asm``` so everything is loaded when we compile. I loaded in a Bank with enough space to load all of them in the same bank, GFX files can be bulky.

- Edit ```main.asm```:
```diff
SECTION "Print Party", ROMX

INCLUDE "engine/printer/print_party.asm"
+INCLUDE "gfx/rangi_gfx.asm"
```

## 3. Set-Up Utility Fuctions

- Edit ```home\copy```:

(This function we are adding is from Polished Crystal, it just works and is used a lot.)
```diff
	pop af
	rst Bankswitch
	ret
	
+FarCopyColorWRAM::
+	ld a, BANK("GBC Video")
FarCopyWRAM::
	ldh [hTempBank], a
	ldh a, [rSVBK]
```

- Edit ```data\predef_pointers.asm```:

This allows us to be able to use certain locally defined functions elsewhere (FIX!!!!! with/without using FarCall? find out) and adds usability and convenience when we do not want to use extra WRAM to pass variables????
```diff
	add_predef TradeAnimation
	add_predef CopyMonToTempMon
	add_predef ListMoves
-	add_predef PlaceNonFaintStatus
	add_predef Unused_PlaceEnemyHPLevel
	add_predef ListMovePP
	add_predef GetGender
...
	add_predef ConvertMon_1to2
	add_predef NewPokedexEntry
	add_predef Unused_AnimateMon_Slow_Normal
-	add_predef PlaceStatusString
	add_predef LoadMonAnimation
	add_predef AnimateFrontpic
	add_predef Unused_HOF_AnimateAlignedFrontpic
	add_predef HOF_AnimateFrontpic
+	add_predef GetStatusConditionIndex
+	add_predef Player_PlaceNonFaintStatus
+	add_predef Enemy_PlaceNonFaintStatus
	dbw -1, DummyEndPredef ; pointless
```


## 4. Edit ```engine\gfx\color.asm```:

**WARNING:** This is going to seem complicated, but for now just trust the process, and we will learn what each Function does as it comes up.

- Add these Utility Functions in ```engine\gfx\color.asm```:

Add them together as I have them here, somewhere in the middle of the file. I added this entire block right above function ```LoadStatsScreenPals```


```
LoadCPaletteBytesFromHLIntoDE:
	ldh a, [rSVBK]
	push af
	ld a, BANK("GBC Video")
	ldh [rSVBK], a
.loop
	ld a, [hli]
	ld [de], a
	inc de
	dec c
	jr nz, .loop
	pop af
	ldh [rSVBK], a
	ret

LoadStatsScreenStatusIconPalette:
	ld de, wTempMonStatus
	jr LoadPlayerStatusIconPalette.statusindex
LoadPlayerStatusIconPalette:
	ld a, [wPlayerSubStatus2]
	ld de, wBattleMonStatus
.statusindex	
	farcall GetStatusConditionIndex
	ld hl, StatusIconPals
	ld c, d
	ld b, 0
	add hl, bc
	add hl, bc
	;ld de, wBGPals1 palette PAL_BATTLE_BG_STATUS + 2
	ld de, wBGPals1 palette 6 + 2
	ld bc, 2
	jp FarCopyColorWRAM

LoadEnemyStatusIconPalette:
	ld a, [wEnemySubStatus2]
	ld de, wEnemyMonStatus
	farcall GetStatusConditionIndex
	ld hl, StatusIconPals
	ld c, d
	ld b, 0
	add hl, bc
	add hl, bc
	;ld de, wBGPals1 palette PAL_BATTLE_BG_STATUS + 4
	ld de, wBGPals1 palette 6 + 4
	ld bc, 2
	jp FarCopyColorWRAM

LoadBattleCategoryAndTypePals:
	ld a, [wPlayerMoveStruct + MOVE_TYPE]
	and CATG_MASK
	swap a
	srl a
	srl a
	dec a
	ld b, a
	ld a, [wPlayerMoveStruct + MOVE_TYPE]
	and TYPE_MASK
	; Skip Bird
	cp BIRD
	jr c, .type_adjust_done
 	cp UNUSED_TYPES
	dec a
	jr c, .type_adjust_done
	sub UNUSED_TYPES
.type_adjust_done
	ld c, a
	;ld de, wBGPals1 palette PAL_BATTLE_BG_TYPE_CAT + 2 ; if I was properly using Constants lol
	ld de, wBGPals1 palette 5

LoadCategoryAndTypePals:
; we need the white palette lol kill me
; im too dumb to do this any other way
	ldh a, [rSVBK]
	push af
	ld a, BANK(wBGPals1)
	ldh [rSVBK], a
	ld a, LOW(PALRGB_WHITE)
	ld [de], a
	inc de
	ld a, HIGH(PALRGB_WHITE)
	ld [de], a
	inc de
	pop af
	ldh [rSVBK], a
	;

	ld hl, CategoryIconPals
	ld a, b
	add a
	add a
	push bc
	ld c, a
	ld b, 0
	add hl, bc
	ld bc, 4
	push de
	call FarCopyColorWRAM
	pop de

	ld hl, TypeIconPals
	pop bc
	ld a, c
	add a
	ld c, a
	ld b, 0
	add hl, bc
	inc de
	inc de
	inc de
	inc de
	ld bc, 2
	jp FarCopyColorWRAM

LoadMonBaseTypePal:
	ld hl, TypeIconPals
	ld a, c ; c is the type ID
	add a
	ld c, a
	ld b, 0
	add hl, bc
	ld bc, 2
	jp FarCopyColorWRAM

LoadSingleBlackPal:
	ldh a, [rSVBK]
	push af
	ld a, BANK(wBGPals1)
	ldh [rSVBK], a
	xor a
	ld [de], a
	inc de
	ld [de], a
	inc de

	pop af
	ldh [rSVBK], a
	ret

InitPartyMenuStatusPals:
	ld hl, StatusIconPals
	ld c, $1 ; PSN
	ld b, 0
	add hl, bc
	add hl, bc
	ld de, wBGPals1 palette 4 + 2
	ld bc, 2
	call FarCopyColorWRAM

	ld hl, StatusIconPals
	ld c, $2 ; PAR
	ld b, 0
	add hl, bc
	add hl, bc
	ld de, wBGPals1 palette 5 + 2
	ld bc, 2
	call FarCopyColorWRAM

	ld hl, StatusIconPals
	ld c, $3 ; SLP
	ld b, 0
	add hl, bc
	add hl, bc
 	ld de, wBGPals1 palette 6 + 2
	ld bc, 2
	call FarCopyColorWRAM

	ld hl, StatusIconPals
	ld c, $4 ; BRN
	ld b, 0
	add hl, bc
	add hl, bc
	ld de, wBGPals1 palette 4 + 4
	ld bc, 2
	call FarCopyColorWRAM

	ld hl, StatusIconPals
	ld c, $5 ; FRZ
	ld b, 0
	add hl, bc
	add hl, bc
	ld de, wBGPals1 palette 5 + 4
	ld bc, 2
	call FarCopyColorWRAM
	ret

; Input: E must contain the offset of the selected palette from PartyMenuOBPals.
SetFirstOBJPalette::
	ld hl, PartyMenuOBPals
	ld d, 0
	add hl, de
 	ld de, wOBPals1
	ld bc, 1 palettes
	ld a, BANK(wOBPals1)
	call FarCopyWRAM
	ld a, TRUE
 	ldh [hCGBPalUpdate], a
 	call ApplyPals
 	ret
```

- Edit function ```LoadStatsScreenPals``` in ```engine\gfx\color.asm```:
```diff
	ld a, [hli]
	ld [wBGPals1 palette 0], a
	ld [wBGPals1 palette 2], a
+	ld [wBGPals1 palette 6], a
+	ld [wBGPals1 palette 7], a
	ld a, [hl]
	ld [wBGPals1 palette 0 + 1], a
	ld [wBGPals1 palette 2 + 1], a
+ 	ld [wBGPals1 palette 6 + 1], a
+	ld [wBGPals1 palette 7 + 1], a
	pop af
	ldh [rSVBK], a
	call ApplyPals
```

- Finally, edit ```engine\gfx\color.asm``` and Load our new palettes we defined in ```gfx/types_cats_status_pals.asm```:
```diff
INCLUDE "data/maps/environment_colors.asm"

+ INCLUDE "gfx/types_cats_status_pals.asm"

PartyMenuBGMobilePalette:
INCLUDE "gfx/stats/party_menu_bg_mobile.pal"
```

Now, we are ready to edit individual engine files and start seeing some changes!

## 5. Pokedex

- Edit function ```NewPokedexEntry``` in ```engine\pokedex\new_pokedex_entry.asm```:

This removes a flashing effect after we leave the Pokedex splash screen that comes up after we catch a new species for the first time. That flash highlights our Type tiles in a very ugly way, since the flash effect overrides our palettes it looks very unnatural. This edit changes the flash to simply fade to black. A small but important change.
```diff
	pop af
	ld [wPokedexStatus], a
	call MaxVolume
-	call RotateThreePalettesRight
+	farcall Pokedex_BlackOutBG
	ldh a, [hSCX]
	add -POKEDEX_SCX
	ldh [hSCX], a
```

- Edit ```Pokedex_InitDexEntryScreen``` in ```engine\pokedex\pokedex.asm```:

We're simply moving when the CGB Layout is applied, honestly not sure if this is even necessary, PLZ DOUBLE CHECK BEFORE PUBLISH!!!!

This function prepares the GFX for when we go from the main scrolling list of Pokemon in the Pokedex, to actually viewing the specific species Entry in the Pokedex, where you see its information, can select to view Area of where to find it, and can see its height, weight, and info blurb if you've caught this species.

```diff
	xor a
	ldh [hBGMapMode], a
	call ClearSprites
+	call Pokedex_GetSelectedMon
+	ld [wCurPartySpecies], a
+	ld a, SCGB_POKEDEX
+	call Pokedex_GetSGBLayout

	call Pokedex_LoadCurrentFootprint
	call Pokedex_DrawDexEntryScreenBG
	call Pokedex_InitArrowCursor
...
	call WaitBGMap
	ld a, $a7
	ldh [hWX], a
-	call Pokedex_GetSelectedMon
-	ld [wCurPartySpecies], a
-	ld a, SCGB_POKEDEX
-	call Pokedex_GetSGBLayout

	ld a, [wCurPartySpecies]
	call PlayMonCry
	call Pokedex_IncrementDexPointer
```
We are editing what is actually shown on the Pokemon Entry page next!

- Edit ```enginge\pokedex\pokedex_2.asm``` by adding our new function ```Dex_PrintMonTypeTiles```:

I added it at the end of the file.

```
Dex_PrintMonTypeTiles:
	ld a, [wTempSpecies] 
        ; when we select the Pokemon's Dex Entry from the main scroll menu, the species num is in wTempSpecies
        ; but some functions like GetBaseData uses wCurSpecies instead of wTempSpecies
	ld [wCurSpecies], a	
	call GetBaseData
	ld a, [wBaseType1]
; Skip Bird Type, if needed
	cp BIRD
	jr c, .type1_adjust_done
	cp UNUSED_TYPES
	dec a
	jr c, .type1_adjust_done
	sub UNUSED_TYPES
.type1_adjust_done
; this number in 'a' now corresponds to the new GFX tiles and Palettes we've added, as an index;
; 0 is Normal, 1 is Fighting, 2 is Flying, etc
; we know the size of each Type (made of a group of 4 tiles)
; so we use some math to determine the address of only the 4 tiles we need to load
; load the tiles into Tile $70, since we are loading 4 tiles at once they will also use $71, $72, and $73
	ld hl, TypeLightIconGFX
	ld bc, 4 * LEN_2BPP_TILE
	call AddNTimes
	ld d, h
	ld e, l
	ld hl, vTiles2 tile $70
	lb bc, BANK(TypeLightIconGFX), 4 ; 4 tiles wide
        ; TypeLightIconGFX is defined in gfx\rangi_gfx.asm
        ; hl is the DESTINATION ADDRESS, Tile in VRAM
        ; de is the SOURCE ADDRESS, from the 2bpp file
	call Request2bpp ; Loads the 4 tiles (in the .2bpp file, made from the .png file) into VRAM
; 2nd Type
	ld a, [wBaseType2]
; Skip Bird
	cp BIRD
	jr c, .type2_adjust_done
	cp UNUSED_TYPES
	dec a
	jr c, .type2_adjust_done
	sub UNUSED_TYPES
.type2_adjust_done
; load type 2 tiles
	ld hl, TypeDarkIconGFX
	ld bc, 4 * LEN_2BPP_TILE
	call AddNTimes
	ld d, h
	ld e, l
	ld hl, vTiles2 tile $74 ; Will also be using Tiles $75, $76, and $77 since the Type is 4 tiles wide
	lb bc, BANK(TypeDarkIconGFX), 4 ; 4 tiles wide
        ; TypeDarkIconGFX is defined in gfx\rangi_gfx.asm
        ; hl is the DESTINATION ADDRESS, Tile in VRAM
        ; de is the SOURCE ADDRESS, from the 2bpp file
	call Request2bpp ; Loads the 4 tiles (in the .2bpp file, made from the .png file) into VRAM

	hlcoord 9, 1 ; this is the Square we will instert the first Type Tile
	ld [hl], $70
	inc hl ; hl is now pointing to hlcoord 10, 1 (appears A, 1 in hex)
	ld [hl], $71
	inc hl ; hl is now pointing to hlcoord 11, 1 (appears B, 1 in hex)
	ld [hl], $72
	inc hl ; hl is now pointing to hlcoord 12, 1 (appears C, 1 in hex)
	ld [hl], $73
	inc hl ; hl is now pointing to hlcoord 13, 1 (appears D, 1 in hex)

        ; checking if the pokemon species only has one type
        ; if so, we are done and can leave
	ld a, [wBaseType1]
	ld b, a
	ld a, [wBaseType2]
	cp b
	ret z

	ld [hl], $74 ; First Tile of Type2
	inc hl ; hl is now pointing to hlcoord 14, 1 (appears E, 1 in hex)
	ld [hl], $75
	inc hl ; hl is now pointing to hlcoord 15, 1 (appears F, 1 in hex)
	ld [hl], $76
	inc hl ; hl is now pointing to hlcoord 16, 1 (appears 10, 1 in hex)
	ld [hl], $77
	ret
```

Not so bad to follow how GFX loading works, right?

- Edit function ```DisplayDexEntry``` in ```engine\pokedex\pokedex_2.asm" to call ```Dex_PrintMonTypeTiles```:
```diff
DisplayDexEntry:
+	call Dex_PrintMonTypeTiles
	call GetPokemonName
	hlcoord 9, 3
	call PlaceString ; mon species
```

Now, we need to make sure the right colors are used on our newly loaded and placed Type Tiles. By the time ```DisplayDexEntry``` is called, we have already loaded the palettes we need via applying the Pokedex CGB Layout, remember when we called ```call Pokedex_GetSGBLayout``` in ```engine\pokedex\pokedex.asm``` right before we got into the Species Entry screen? In more complex scenarios, you may be switching entire CGB Layouts on the fly depending on what is happening, and you do so by specifying the CGB Layout index in ```b``` then calling ```GetSGBLayout```. Inside the pokedex, there's an additional abstraction that probably isn't needed but I have deemed it too annoying to actually remove, ```Pokedex_GetSGBLayout``` simply puts the index given in ```a``` into ```b```, then calls ```GetSGBLayout``` normally... 

- Change the Pokedex CGB Layout, by editing function ```_CGB_Pokedex``` in ```engine\gfx\cgb_layouts.asm```:
```diff
.is_pokemon
	call GetMonPalettePointer
	call LoadPalette_White_Col1_Col2_Black ; mon palette
+; black background
+	ld de, wBGPals1 palette 7	
+	call LoadSingleBlackPal
+; mon type 1	
+	ld a, [wTempSpecies]
+	ld [wCurSpecies], a	
+	call GetBaseData
+	ld a, [wBaseType1]
+; Skip Bird
+	cp BIRD
+	jr c, .type1_adjust_done
+	cp UNUSED_TYPES
+	dec a
+	jr c, .type1_adjust_done
+	sub UNUSED_TYPES
+.type1_adjust_done
+; load the 1st type pal 
+	ld c, a
+	ld de, wBGPals1 palette 7 + 2
+	farcall LoadMonBaseTypePal	
+; mon type 2
+	ld a, [wBaseType2]
+; Skip Bird
+	cp BIRD
+	jr c, .type2_adjust_done
+	cp UNUSED_TYPES
+	dec a
+	jr c, .type2_adjust_done
+	sub UNUSED_TYPES
+.type2_adjust_done
+; load the 2nd type pal 
+	ld c, a
+	ld de, wBGPals1 palette 7 + 4
+	farcall LoadMonBaseTypePal

.got_palette
	call WipeAttrmap
	hlcoord 1, 1, wAttrmap
	lb bc, 7, 7
	ld a, $1 ; green question mark palette
	call FillBoxCGB
+
+; mon base types
+	hlcoord 9, 1, wAttrmap
+	lb bc, 1, 8
+	ld a, $7 ; mon base type pals
+	call FillBoxCGB
+
	call InitPartyMenuOBPals
	ld hl, PokedexCursorPalette
	ld de, wOBPals1 palette 7 ; green cursor palette
	ld bc, 1 palettes
	ld a, BANK(wOBPals1)
	call FarCopyWRAM
	call ApplyAttrmap
	call ApplyPals
	ld a, TRUE
	ldh [hCGBPalUpdate], a
	ret
```

- Here is the detailed breakdown of what we just did:

I'm sure you noticed we are doing the same exact thing to fix the Type number so that it's a proper index that we can use to get our Palette for the Type, again, Normal is 0, Fighting is 1, Flying is 2, etc.

We are using Palette 7 for our Type Tiles, as it is unused in the Vanilla pokedex.

The first color loaded into a Palette is the background color, denoted by White in the .png files. The pokedex has a black background color scheme, so that's why we load Black into the first Palette slot of Palette 7 via 
```
; black background
	ld de, wBGPals1 palette 7	
	call LoadSingleBlackPal
```

Each Palette slot is 2 bytes, for a total of 8 bytes per Palette. So, to Load the Light Grey designated color into Palette 7 (after we calculated our Type1 index) we added 2 bytes onto the Palette 7 pointer. The type index is in 'a', but a Farcall will clobber 'a' because it uses 'a' to specify the Bank the Farcall destination is in, so we move the Type Index  into 'c' where ```LoadMonBaseTypePal``` expects it to be. 

```
; load the 1st type pal 
	ld c, a
	ld de, wBGPals1 palette 7 + 2
	farcall LoadMonBaseTypePal
```

We do the same once we calculate the index for Type2, but now we must add 4 bytes onto the Palette 7 pointer to load into the Dark Grey slot (slot 3):

NOTE: proper ettiquette demands not using naked numbers not even 7 for Palette 7, really need to make defined and descriptive CONSTANTS instead, FIX BEFORE PUBLISHING

```
; load the 2nd type pal 
	ld c, a
	ld de, wBGPals1 palette 7 + 4
	farcall LoadMonBaseTypePal
```

Finally, we edit the Screen's Attribute Map (wAttrmap) to tell it that our specified area of 1x8 Tiles starting @ hlcoord 9,1 will be using Palette 7.

```
; mon base types
	hlcoord 9, 1, wAttrmap ; DO NOT FORGET TO SPECIFY 'wAttrmap' HERE
	lb bc, 1, 8 ; area = HEIGHT in 'b', WIDTH in 'c'
	ld a, 7 ; Palette 7
	call FillBoxCGB ; fills in a rectangle/box, there are other options to edit wAttrmap we will use later on
```

After this point, at the end of the ```_CGB_Pokedex``` and EVERY CGB LAYOUT, we MUST do the following to actually apply the Palette configurations and the wAttrmap designations. This is a vanailla function so it's already done for us. But later on, when we are making entirely new CGB Layouts, we cannot forget to end them with this:

```
        ...
	call ApplyAttrmap
	call ApplyPals
	ld a, TRUE
	ldh [hCGBPalUpdate], a
	ret
```

## 6. Stats Screen

## 7. Mon Menu / Party Menu

## 8. Battle
