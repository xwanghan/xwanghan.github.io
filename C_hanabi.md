# Hanabi 
Hanabi environment is introduced by [The Hanabi Challenge: A New Frontier for AI Research](), `4 players full`

### Action space `(int, 40)`:
- `Play (5)`
- `Discard (5)`
- `Reveal colors or rank (30)`

### Obs space `(float, 1379)`:
- `Hands info (4*126=504)`:
  - For each player, `hand info (25*5=125)`, and current player info is all 0
  - For each player, `if missing a card (1)`
- `Board info (66)`:
  - `remaining deck size (50-5*4=30)`
  - `state of the fireworks (5*5=25)`
  - `information tokens remaining (8)`
  - `life tokens remaining (3)`
- `Discards info (50)`
- `Last action (59)`:
  - `Acting player index (4)`
  - `The MoveType (4)`
  - `Target player index (4)`, relative to acting player, if a reveal move
  - `Color revealed (5)`, if a reveal color move
  - `Rank revealed (5)`, if a reveal rank move
  - `Reveal outcome (5)`, each bit is 1 if the card was hinted at
  - `Position played/discarded (5)`
  - `Card played/discarded (25)`
  - `Successful and/or added information token (if play action) (2)`
- `V0Belief (float, 4*5*(5*5+5+5)=700)`:
  - For each players' cards, `list possible colors and ranks, (int)`
  - Compute `each specific color and rank cards' count in hands`, and compute `possibility of color and rank (float)` 
  for each players' cards, introduced by [BAD]()



# Hat principle
Hat principle is introduced by [How to Make the Perfect Fireworks Display: Two Strategies for Hanabi](), 
it offers an approach to send information efficiently


### When agent needs to play action, it will decide whether to send information:
- If not, decide `play` or `discard` action (Choose 1 from 10)
- If yes, send `coded information` by `reveal` action (30 actions to embed)

### `Information` includes 2 parts:
- `Hat information`, dominate in games, as it will transmit information in higher efficiency
- `Naive information`, simply introduced by `reveal` action

By changing `Offset` in `information embeding session`, we can control `Naive information`


### Coded information:
- Obtain each players' `hat` (`A`, `B`, `C`)
- Sum all `hats` and mod to get `Hat information`: `Info = (A + B + C) mod 30`
- Embed `information` to `reveal` action, `Action = (Info + Offset) mod 30`

### Decode information:
- For each `reveal` action being played by an agent, all other agents will update their `information`
- For `reveal` action played by other players, `A` will decode `Hat information` by:
  - Reverse `Offset`, get `information`, `Info = (Action - Offset) mod 30`
  - Get `A's hat`, `Hat = (Info - B - C - D) mod 25`
  - Then` A's the most playable card's` `rank` and `color` is `A//5` and `A%5`



# Recommendation strategy 

`Hat information` refers to `recommend action` from others' view

### Hat definition:
- From one player's view, we can get others' `recommend action (10)`

### Changed action space `(int, 2)`:
- `Play or Discard (10)`
- `Send information (1)`, will embed to a `reveal` action, with 30 values

### Changed obs space `(int, 2)`:
- `Hands info (4*126=504)`:
  - ...
- `Board info (66)`:
  - ...
- `Discards info (50)`
- `Last action (61)`:
  - ...
- `V0Belief (float, 4*5*(5*5+5+5)=700)`:
  - ...
- `Hat`, or `recommended action (10)`

### Recommendation agent
- Have `4` copes, choose `recommendation action` via `recommendation obs` for 4 different `targets`
- When computing `hat`, get `recommendation action` for `3 other players`

### Recommendation action space `(int, 10)`:
- `Play or Discard (10)`

### Recommendation obs space `(float, 1001)`:
- `Target hands info (126)`:
  - For target player, `hand info (25*5=125)`, and current player info is all 0
  - For target player, `if missing a card (1)`
- `Board info (66)`:
  - ...
- `Discards info (50)`
- `Last action (59)`:
  - ...
- `V0Belief (float, 4*5*(5*5+5+5)=700)`:
  - ...

### Offset select:
- ...



# Information strategy 

### Hat definition:
- When constructing `V0Belief`, we can get player's each cards' `possible colors and ranks (int)` and 
`each specific color and rank cards' count in hands`
- From `state of the fireworks`, we can get which specific `rank` and `color` can to `play`
- Then, we can get each hand's `possibility to play`, and `the most playable card` for each player
- `Hat` is player's `most playable card's id` `(25)`

### Changed action space `(int, 11)`:
- `Play or Discard (10)`
- `Send information (1)`, will embed to a `reveal` action, with 30 values

### Offset select:
`Offset selecting protocol` should be public information, thus agents can decode their own hat, 
there are serval approaches to decide `Offset`:
- Constant `Offset`, a number `0 < C < 25`
- One to one mapped `Offset`, a mapping function `Offset = f(A)`:
  - [x] `Simply implemented by a shuffle method`: 
    - select `1` mapping from total `30!`, and evaluate, record
    - `slow`, Because `Naive information` is less import than `Hat information`, 
    and we don't update parameters of agent. We don't know agent whether well utilized shuffled `Naive information`, 
    or shuffled `Naive information` has more useful information
- Public mapping agent, `Offset = f(Public Obs)`, 
where `Public Obs` includes: `Board info`, `Discards info`, ` Last action`, `V0Belief`:
  - [x] `Cheat game`:
    - Extend agents' `action space` to original size `(10+30=40)`, when choosing `reveal` action, 
    it will `additionally` send `Hat information`, and `reveal` action's corresponding `Naive information`
    - Then optimize, and extract `Cheat game's Offset`, `Public Obs` to supervise train `f()`
    - `slow`, And may lose information when supervise training `f()`
  - [x] `2-level optimization`:
    - Define a `public mapping agent`, which `action space` is `offset (25)`
    - Load pretrained `Information Strategy Agents'` parameter, then `2-level optimize`
    - `quicker`, but `harder` to optimize
  - [ ] ...
- ...





# End












