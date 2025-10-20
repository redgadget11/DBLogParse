# DBLogParse
A comprehensive DuelingBook log parser designed to extract structured game data from raw duel logs. This project enables card tracing, game state reconstruction, and data extraction for statistical analysis, machine learning, and reinforcement learning research on Yu-Gi-Oh! gameplay.

# Functionality
The code needs to take in a ".json" log file and extract the gamestate for each action/event/play. This state needs to be coherent with the gamestate shown by the DuelingBook parser for each action/event/play. In order to do so, we define a state object that should contain all the data associated with a game and turn by turn is populated and updated to reflect the gamestate. The gamestate object we use is defined bellow. If there is a need for extra properties they can and should be easily added extending our definition.

```python
persistent_state = {
    # general
    "turn": 0,
    ### me
    "me_life": 8000,
    "me_hand": [],
    "me_GY": [],
    "me_banished": [],
    "me_field": {
        "M-1": None,
        "M-2": None,
        "M-3": None,
        "M-4": None,
        "M-5": None,
        "S-1": None,
        "S-2": None,
        "S-3": None,
        "S-4": None,
        "S-5": None,
    },
    "me_phase": None,
    ### opponent
    "op_life": 8000,
    "op_hand": [],
    "op_GY": [],
    "op_banished": [],
    "op_field": {
        "M-1": None,
        "M-2": None,
        "M-3": None,
        "M-4": None,
        "M-5": None,
        "S-1": None,
        "S-2": None,
        "S-3": None,
        "S-4": None,
        "S-5": None,
    },
    "op_phase": None,
}
```
A decision was made to keep every card as an object created by the "assign_id" function. "None" values on the field and arrays should be populated by objects of this kind:
```python
{
  "name": "Bottomless Trap Hole",
  "id": 11,
  "state": "private",
  "object_id": 27,
  "battle_position": null,
  "face_up": null
},
{
  "name": "Blackwing - Shura the Blue Flame",
  "id": 13,
  "state": "private",
  "object_id": 3,
  "battle_position": null,
  "face_up": null
}
```
While in hand, and of the following kind while on field (BTH is set in S-3 "spell-trap zone 3" and Shura is in in M-4 for example in face up defense):
```python
"S-3": {
  "name": "Bottomless Trap Hole",
  "id": 11,
  "state": "private",
  "object_id": 27,
  "battle_position": null,
  "face_up": False
},
"M-4": {
  "name": "Blackwing - Shura the Blue Flame",
  "id": 13,
  "state": "public",
  "object_id": 3,
  "battle_position": "DEF",
  "face_up": True
}
```
"object_id" and "id" differ in a significant way and should be kept together. "id" is the id we assign when the card object is created. Each player has its own set of ids that are assigned at the start of the game. "object_id" is given by the game, however if the card is hidden we do not always obtain it immediately. It is very important to keep track of both ids as one is what we maintain and the other is how we verify (if available) who's card is it.

**The majority of the issues with this project arise from id maintenance and following cards from action to action.**

# Extracting Data
Data extraction happens from 3 places within the individual "play" log. The "card" property itself if we are lucky, the "public_log" property or "private_log" otherwise. The parsing of logs is difficult as for most actions logs differ in implementation, however there is a "log_message" parser that works okayish already implemented. It could be updated and simplified.

The plays we are interested in are usually in the following categories:
- Card Movement (Special summons e.g. deck/gy/hand->field, Sets hand->field) as the state should change.
- Life points changes
- Phase changes

The plays we are **not** interested in are usually in the following categories (for now):
- Players messages
- Thumbs up
- Any other play that does not change the gamestate.

That being said, we must implement a parser that uses "card" properties, or "public/private_log" info to parse cards from "play" logs into our state. I present all possible actions from each part of the Yugioh Field such that we can address them systematically.

## HAND
Monster Actions ->
<img width="128" height="227" alt="image" src="https://github.com/user-attachments/assets/919b3f80-759f-49e2-b25a-77c4fcb87ad1" />
Spell Actions ->
<img width="130" height="172" alt="image" src="https://github.com/user-attachments/assets/133476c0-2653-4729-ae2e-1ac64262d7c2" />
Trap Actions ->
<img width="128" height="173" alt="image" src="https://github.com/user-attachments/assets/e70d1d5b-5bc0-4c0d-a1b6-e1571d91f0b5" />

## FIELD
Monster Actions Face Up ATK ->
<img width="76" height="200" alt="image" src="https://github.com/user-attachments/assets/8bd28ece-06cc-4e1d-855a-6a14a4bd6a1a" />
Monster Actions Face Up DEF ->
<img width="112" height="206" alt="image" src="https://github.com/user-attachments/assets/a3d0a5b0-e7a4-44f7-8ed5-9ab9d0c39a80" />
Monster Actions Face Down DEF ->
<img width="111" height="196" alt="image" src="https://github.com/user-attachments/assets/1d3f06d1-bfc6-48a7-8292-7961bdbd7b72" />
Extra Monster Actions Face Up ATK ->
<img width="81" height="224" alt="image" src="https://github.com/user-attachments/assets/56628cd9-0e9b-4a52-bce6-d3e8d28e66d5" />
Extra Monster Actions Face Up DEF ->
<img width="115" height="196" alt="image" src="https://github.com/user-attachments/assets/817106b6-171b-4521-b0f7-495d062d1d70" />
Extra Monster Actions Face Down ATK ->
<img width="114" height="179" alt="image" src="https://github.com/user-attachments/assets/0fc2ae36-0263-41a0-b913-0a14fbd16c3f" />
Spell Actions Face Up ->
<img width="78" height="187" alt="image" src="https://github.com/user-attachments/assets/c4543a36-87ed-4f28-a0d3-76998840790c" />
Spell Actions Face Down ->
<img width="77" height="177" alt="image" src="https://github.com/user-attachments/assets/44b54e35-43fa-451d-8d82-358f10ce6ba4" />
Trap Actions Face Up ->
<img width="77" height="195" alt="image" src="https://github.com/user-attachments/assets/245f53fa-173f-4078-890f-b2e589200988" />
Trap Actions Face Down ->
<img width="81" height="177" alt="image" src="https://github.com/user-attachments/assets/182bca66-69fa-4358-871b-0841cb7d2cd1" />

## GRAVEYARD
Monster Actions ->
<img width="79" height="159" alt="image" src="https://github.com/user-attachments/assets/736401d3-2fe1-4c7e-9ec5-a9faae3cb117" />
Extra Monster Actions ->
<img width="86" height="171" alt="image" src="https://github.com/user-attachments/assets/584bb78d-7946-4fcd-8a6a-9af1a647ef11" />
Spell Actions ->
<img width="74" height="157" alt="image" src="https://github.com/user-attachments/assets/60adfb2f-8ab4-4acd-99eb-967b0b619e41" />
Trap Actions ->
<img width="78" height="159" alt="image" src="https://github.com/user-attachments/assets/7ccd0c19-11ad-4cc5-919b-79bcb96fc84e" />

## BANISH
Monster Actions ->
<img width="80" height="158" alt="image" src="https://github.com/user-attachments/assets/980939ac-d30d-49d6-804a-9db2a9c9d904" />
Extra Monster Actions ->
<img width="80" height="161" alt="image" src="https://github.com/user-attachments/assets/8c9a1dd9-dcec-4fd8-850e-5b0099dedb9f" />
Spell Actions ->
<img width="78" height="127" alt="image" src="https://github.com/user-attachments/assets/f0092a65-1d0d-4456-8f97-4503a8145e82" />
Trap Actions ->
<img width="81" height="129" alt="image" src="https://github.com/user-attachments/assets/ce98edd1-3125-48a5-b3b5-9c1263731fae" />

## DECK
Monster Actions ->
<img width="80" height="155" alt="image" src="https://github.com/user-attachments/assets/e13dddf9-5091-4023-837e-16e93f237306" />
Spell Actions ->
<img width="77" height="127" alt="image" src="https://github.com/user-attachments/assets/ff6d5cdc-d4e1-4254-9395-893ed591edb2" />
Trap Actions ->
<img width="77" height="127" alt="image" src="https://github.com/user-attachments/assets/7fafd407-fd96-4d61-b66b-26beb320065f" />

## EXTRA-DECK
Extra Monster Actions ->
<img width="80" height="138" alt="image" src="https://github.com/user-attachments/assets/8d997d40-0932-409b-af40-89e6e6682446" />

## OPPONENT FIELD AND HAND
Can only Target

## OPPONENT GRAVEYARD
Monster Actions ->
<img width="80" height="102" alt="image" src="https://github.com/user-attachments/assets/38dad2a1-7d25-44f3-b4a1-fd556821ebdf" />
Spell Actions ->
<img width="81" height="79" alt="image" src="https://github.com/user-attachments/assets/b22aff74-043b-4102-a1c9-75d4a8a00f5f" />
Trap Actions ->
<img width="80" height="92" alt="image" src="https://github.com/user-attachments/assets/bb2b7c9f-1bde-4285-88ed-516bb8563f74" />

## OPPONENT BANISH
Monster Actions ->
<img width="77" height="116" alt="image" src="https://github.com/user-attachments/assets/513391f6-043a-4b26-af71-dba0560a5a70" />
Spell Actions ->
<img width="81" height="85" alt="image" src="https://github.com/user-attachments/assets/9002087c-5809-46c5-9e9a-f9ecd9287fa9" />
Trap Actions ->
<img width="76" height="89" alt="image" src="https://github.com/user-attachments/assets/6654fc31-8f07-4972-abce-3a70cbdf2600" />











# Things missing now due to complexity 
- Field spell cards






