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
    {
      "name": "Bottomless Trap Hole",
      "id": 11,
      "state": "private",
      "object_id": 27,
      "battle_position": null,
      "face_up": False
    },
    {
      "name": "Blackwing - Shura the Blue Flame",
      "id": 13,
      "state": "public",
      "object_id": 3,
      "battle_position": "DEF",
      "face_up": True
    }
```
