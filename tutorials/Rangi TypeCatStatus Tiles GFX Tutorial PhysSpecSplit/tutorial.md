## Step 1: Changes in engine\gfx\color.asm
- Edit function ```LoadBattleCategoryAndTypePals```:
```diff
LoadBattleCategoryAndTypePals:
-; determine Move's Category
-	; in Vanilla, need to check Move Power first to see if Status Move
-	; then determine Phys/Spec based on Type if attacking move
-	ld a, [wPlayerMoveStruct + MOVE_POWER]
-	and a
-	jr nz, .nonstatusmove
-	ld a, 2 ; Status Move
-	jr .getcategoryGFX
-.nonstatusmove
	ld a, [wPlayerMoveStruct + MOVE_TYPE]
+	and ~TYPE_MASK ; Phys/Spec split only
+	swap a ; Phys/Spec split only
+	srl a ; Phys/Spec split only
+	srl a ; Phys/Spec split only
+	dec a ; Phys/Spec split only
-	cp SPECIAL
- jr c, .phys
-	ld a, 1 ; category index for Special
-	jr .getcategoryGFX
-.phys
-	xor a
-	; fallthrough
-.getcategoryGFX
-	; 'a' contains Category index: 0 for Phys, 1 for Spec, 2 for Status
	ld b, a ; Move Category Index
	ld a, [wPlayerMoveStruct + MOVE_TYPE]
  ...
 ```
 
 ## Step 2: Changes in engine\gfx\cgb_layouts.asm
 - Edit function ```_CGB_MoveList```:
 ```diff
_CGB_MoveList:
         ...
         call LoadPalette_White_Col1_Col2_Black
         call WipeAttrmap

+	ld hl, Moves + MOVE_TYPE
-; determine Move's Category
-	; in Vanilla, need to check Move Power first to see if Status Move
-	; then determine Phys/Spec based on Type if attacking move
-	ld a, [wCurSpecies]
-	dec a
-	ld hl, Moves + MOVE_POWER
-	ld bc, MOVE_LENGTH
-	call AddNTimes
-       ld a, BANK(Moves)
-	call GetFarByte
-	cp 2
-	jr c, .no_power ; means it's a status move
-	jr nz, .nonstatusmove
-.no_power
-	ld a, 2 ; Status Move category index
-	jr .getcategoryGFX
-.nonstatusmove
         ld a, [wCurSpecies]
         dec a
-	ld hl, Moves + MOVE_TYPE
         ld bc, MOVE_LENGTH
         call AddNTimes
         ld a, BANK(Moves)
         call GetFarByte
+       and ~TYPE_MASK ; Specific to Phys/Spec split
+	swap a ; Specific to Phys/Spec split
+	srl a  ; Specific to Phys/Spec split
+	srl a  ; Specific to Phys/Spec split
+	dec a  ; Specific to Phys/Spec split
-	
-	cp SPECIAL
-	jr c, .phys
-	ld a, 1 ; category index for Special
-	jr .getcategoryGFX
-.phys
-	xor a
-	; fallthrough
-.getcategoryGFX
-	; 'a' contains Category index: 0 for Phys, 1 for Spec, 2 for Status
-
         add a ; double the index
         add a ; quadrouple the index
         ; since entries of CategoryIconPals are 4 bytes (2 colors, 2 bytes each) instead of normal 2 bytes (1 color) 
 
         ...
 
         call AddNTimes
         ld a, BANK(Moves)
         call GetFarByte
+	and TYPE_MASK
         ld c, a ; farcall will clobber a for the bank
         farcall GetMonTypeIndex
         ld a, c
         ...
  
 ```
 
  ## Step 3: Changes in engine\pokemon\mon_stats.asm
  ```diff
GetMonTypeIndex:
	; type in c, because farcall clobbers a
	ld a, c
-	and TYPE_MASK ; Phys/Spec Split only
	; Skip Bird
	cp BIRD
	jr c, .done
  ```
