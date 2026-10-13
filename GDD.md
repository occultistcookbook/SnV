# Scum & Villainy — Game Design Document

**Version:** 0.2, core redesign · **Date:** 2026-10-13 · **Engine:** Godot 4.7.2, GDScript
**Audience:** Claude Code, which reworks the existing greybox to match this document, and the design team.

---

## 0. How to read this document

Version 0.1 was built as a greybox. Playing it showed that the shift did not work, so the team redesigned the core: how a shift runs, what the counter and the tables are for, how plots move, what the player sees. This document describes the **second greybox**. Section 19 lists what changed since v0.1, for work on the existing code.

Every rule carries a tag:

| Tag | Meaning |
|---|---|
| `[D 6.1]` | **Decided.** A design decision of the team, card 6.1 in the design zettelkasten. Implement as written. |
| `[GB]` | **Greybox assumption.** Not a design decision. A placeholder that makes the build work. Keep it in data files so the team can change it without touching code. |
| `[GB, open 2.2.4]` | **Placeholder for an open question.** The team has not decided this yet, but the core needs it. Card 2.2.4 is open; the rule given here is a placeholder. Keep it isolated and data-driven: it will change. |

Build rules:

1. Build only what is in scope (section 1.3). Do not build anything from section 15 ("Next update") or section 16 ("Cut"), not even as a stub or a disabled button.
2. Every tunable number lives in data files, not in code (sections 11 and 13).
3. All content lives in data files: traits, skills, professions, drinks, plots, meeting events, names, texts.
4. Where this document is silent, choose the simplest option that does not contradict a `[D]` rule, and record it in `GREYBOX_NOTES.md` at the repository root.
5. All player-facing strings live in data files. The game language is not decided yet `[GB]`; write greybox strings in English.

---

## 1. Vision

### 1.1 The game in one paragraph

A story simulator with indirect control, with Crusader Kings 3 as the main reference. The player owns a bar on a space station and never leaves the counter. Residents of the station come in with their troubles: a debt, a hunter on their trail, loneliness. The bar is where those troubles get solved, on the player's initiative `[D 1.9]`: the player decides who sits with whom and pours one special drink. The people decide for themselves what to do. Every shift is a race against timers, closest to Cultist Simulator `[D 1.9]`.

### 1.2 Pillars

1. **The owner stands behind the counter.** There is little direct control in this game, by design. `[D 1.1]`
2. **The owner changes circumstances; the person decides.** `[D 1.2]`
3. **The owner listens.** Guests choose what to talk about. There are no dialogue choices. `[D 2.6]` `[D 2.7]`
4. **Fog of war in layers.** You see the surface at once and character over time. Personality and skills are never shown as numbers. `[D 1.3]`
5. **Generated people in authored plots, as in Wildermyth.** The game picks who has the need; the player casts the other roles by seating. `[D 1.6]`
6. **The bar is where problems get solved, and a shift is a race against timers.** `[D 1.9]`
7. **The whole world comes to the bar.** `[D 1.5]`

### 1.3 Greybox scope

In scope:

- One bar on one station, one season with a fixed length and an epilogue.
- The bar seen from behind the counter: 3 counter seats, 3 tables for two.
- Shifts with a timer on every waiting guest, pause-and-act, conversations with a visible fast-forward and a catch.
- Generated residents with hidden personality, hidden skills and traits, some visible at once, some revealed by talking.
- Two player levers: seating guests at tables, pouring one drink per guest.
- Meetings at tables that move plots: the character picks an option by personality, a skill check decides success, an event window shows the result.
- Needs with a deadline; plots that move on between shifts.
- A guest flow mixing plot guests, regulars and dark horses.
- Opinion, relations, obligations, one reputation, rumours.
- Sample content (section 12) and a developer debug overlay.

Out of scope: see section 15 ("Next update") and section 16 ("Cut").

### 1.4 References

| Game | What we take |
|---|---|
| Crusader Kings 3 | Hidden personality and visible traits; skills; decisions weighted by personality; skill checks, by threshold or by roll; opinion with reasons; relations; event windows that pause the game; delayed events. |
| Wildermyth | Generated heroes cast into authored story roles. |
| Cultist Simulator | Timers on everything; the player pauses, plans and acts. |
| This Is the Police | After the shift ends you can still deal with the last people. |
| VA-11 Hall-A, Coffee Talk | A bar as the stage; regulars come back across evenings. |
| RimWorld | Systemic stories. We do **not** take the storyteller-director (`[D 8.4]` is cut). |

---

## 2. Core loop

### 2.1 One shift

1. The bar opens. Guests appear one by one in free counter seats. If all three counter seats are taken, nobody new comes in. `[D 9.2]` `[D 2.2.1]`
2. A guest at the counter waits under a timer. When the timer runs out, the guest leaves. `[D 9.2]` `[D 2.1]`
3. While a guest waits, the player can talk to them, pour them one drink, or seat them at a table. `[D 9.2]`
4. Talking pauses the game. When the talk ends, time fast-forwards visibly for a moment, and the player may catch one guest who would otherwise leave. `[D 2.9]` `[D 1.4.2]`
5. A guest seated alone at a table waits there until the bar closes. `[D 9.3]`
6. Seating a second guest at that table starts a meeting. Nobody can intervene. When the meeting ends, both leave and an event window shows what happened and which trait decided it. `[D 9.3]` `[D 3.9]`
7. At closing time nobody new comes in, but the guests already inside can still be served. `[D 9.4]`
8. Between shifts the world lives off-screen: plots move on their timers, consequences land, new needs appear. The player learns about it later from talk and rumours. `[D 1.5]`

The player can pause at any moment and act while paused. `[D 1.4.1]`

### 2.2 One season

A run lasts a known, fixed time and ends with an epilogue. `[D 1.7]` The greybox season lasts **28 days** `[GB]`.

There is no win or lose state in the greybox. Rewards for solved problems are in the next update `[D 1.10]`. The owner's death would end a run `[D 1.8]`, but the owner is not a simulated character in the greybox (block 12 is in the next update), so this never happens.

---

## 3. Time and controls

### 3.1 Clock

- Time runs continuously, with day and night. The bar is open part of the day and closed the rest. `[D 1.4]`
- While the bar is closed, time skips to the next opening. Nothing interrupts it: there is no knock on the door. `[D 1.5]` `[D 1.5.1]`

Greybox values `[GB]`:

| Parameter | Value |
|---|---|
| Opening hours | 18:00 to 04:00 |
| Clock speed at 1× | 1 real second = 1 game minute |
| Speed controls | Pause, 1×, 2×, 4× |
| Closed time | Skipped at once, with a short "Day N" card |
| Season length | 28 days |

### 3.2 Pause `[D 1.4.1]`

- **Space** toggles pause, as does the Pause button.
- While paused, the player can act: seat guests, pour, start a conversation. Hover popups work while paused.
- During a fast-forward (section 3.3) Space does not take effect at once. The press is remembered, and pause switches on as soon as the fast-forward ends.

### 3.3 Conversation, fast-forward and catch `[D 1.4.2]` `[D 2.1]`

- A conversation pauses the game while the player reads. `[D 1.4]`
- A conversation costs evening time: its `time_cost` in game minutes (default **30** `[GB]`).
- When the conversation closes, the clock does not jump. It **fast-forwards visibly** through the time cost in **1.5 real seconds** `[GB]` (allowed range 1–2 s `[D 1.4.2]`). Timers tick down on screen; guests whose timers run out leave; meetings may end.
- **Catch.** During the fast-forward the player may click one guest who is still seated at the counter. Only the first such click counts. The caught guest does not leave during this fast-forward even if their timer runs out; when the fast-forward ends, their timer is set to the larger of its current value and **10 game minutes** `[GB]`. The fast-forward always plays to the end.
- **How the catch is shown** `[GB, open 1.4.3]`: the caught guest gets a highlight ring until they leave the counter.
- Event windows (section 6.5) pause the game but cost no time and trigger no fast-forward. `[D 3.9]`

---

## 4. The bar

### 4.1 View and layout `[D 9.1]`

The view is from behind the counter onto the hall: the counter runs across the bottom of the screen, three counter seats face the player, and three tables for two stand in the hall behind them.

| Spot | Seats | Role |
|---|---|---|
| Counter seat 1–3 | 1 each | Waiting spot. A guest arrives here, waits under a timer, can talk and drink. `[D 9.2]` |
| Table A–C | 2 each | Meeting spot. `[D 9.3]` |

There is no door queue and no other seat. The rest of block 9 (zones, atmosphere, house rules, opening hours as design) is in the next update.

### 4.2 The counter `[D 9.2]`

- Every guest at the counter has a timer. When it reaches zero, the guest leaves. `[D 2.1]`
- Counter timer length: **60 to 90 game minutes**, random per visit `[GB]`.
- What the player can do with a counter guest: talk (section 6.4), pour (section 4.4), seat at a table (section 4.3).
- Counter neighbours do not affect each other in this build. `[D 9.2]`
- If all three counter seats are taken, no new guest arrives. `[D 9.2]`
- What happens when a guest leaves unserved is decided later `[D 2.1.1]`. In this build the guest simply leaves, with no other effect.

### 4.3 Tables and meetings `[D 9.3]` `[D 4.1]`

- The player seats a counter guest at a table by dragging them to a free table seat. Guests reach a table only from the counter `[GB]`.
- Once seated at a table, a guest cannot be moved or removed. `[D 4.10]` `[D 9.3]`
- A guest alone at a table has no timer and waits until the bar closes. `[D 9.3]`
- Seating a second guest at an occupied table starts a **meeting** between the two (section 8.4).
- Meeting duration depends on its type `[D 9.3]`: plot meeting **60**, need meeting **60**, plain meeting **30** game minutes `[GB]`.
- The player cannot intervene in a meeting. When it ends, both guests leave with a short leaving animation, and the event window appears. `[D 9.3]` `[D 3.9]`
- Quarrels, fights, snippets and rumours all come out of meetings; there are no background bar events. `[D 9.3]` `[D 4.11]`

### 4.4 Pouring `[D 10.3]` `[D 10.1]` `[D 10.2]`

- The player can pour only for a guest at the counter: pour first, seat after.
- One drink per guest per visit. That is why it matters.
- The player picks a drink from the fixed menu (section 12.5). There is no mixing.
- A drink is a **temporary trait**: it shifts personality, skills, or both. It lasts until the end of that guest's meeting; if the guest never has a meeting, until they leave.
- A poured drink may make the guest say a short line `[D 10.3]`. In this build the line is flavour only `[GB]`; what it should give is in the next update `[D 10.3.2]`.
- Drinks are free in the greybox. Money is in the next update. `[GB]`
- There is no refusing to pour and no "throw out". `[D 4.10]`

### 4.5 Closing `[D 9.4]`

- At 04:00 the bar closes: no new guests arrive.
- Guests already inside stay. Counter timers keep running; the player can still talk, pour and seat; meetings in progress finish.
- Guests alone at tables stay until the counter is empty and no meeting is running, so a late match is still possible `[GB]`. Then they leave, and the shift ends.

### 4.6 What the owner cannot do

- Throw anyone out or refuse a drink. `[D 4.10]`
- Send a drink "from" another guest. `[D 4.1.1]`
- Ask questions, answer, advise. `[D 2.6]` `[D 2.7]`
- Tell someone a piece of information. This lever is in the next update. `[D 4.2]`
- Blackmail, spread a rumour on purpose, hide someone or hand them over. `[D 4.4]` `[D 4.3]` `[D 4.8]`
- Step into a meeting or a fight. `[D 9.3]` `[D 4.11]`
- Leave the bar. `[D 1.5]`

### 4.7 Neutrality `[D 5.4]`

The bar takes no side. Enemies may end up under one roof; tension is handled by seating, and fights happen only as consequences of meetings.

---

## 5. Characters

### 5.1 Personality, skills, traits `[D 6.1]`

Three separate things, as in CK3:

- **Personality**: hidden scales. Interactions with mechanics and people's decisions rely on them.
- **Skills**: hidden values. They decide whether what a person tries actually works. `[D 6.8.1]`
- **Traits**: named sets of shifts to personality and/or skills. There are no pairs of opposite traits.

The player never sees personality or skills, as numbers or otherwise; the developer overlay shows them. `[D 1.3]` `[D 6.8.1]` The player judges people by their traits.

**Effective value** = base value + sum of trait shifts + active drink shifts, clamped to the range `[GB]`. Every check in the game uses effective values.

#### Personality scales `[GB]`

Six scales, range −100 to +100, base values random in −15..+15.

| Scale | High means |
|---|---|
| `boldness` | Takes risks, acts |
| `compassion` | Helps, forgives |
| `greed` | Wants more, bends rules for gain |
| `honor` | Keeps word, pays debts |
| `sociability` | Seeks company, talks |
| `rationality` | Thinks before acting |

#### Skills `[GB, open 6.8.2]`

Which skills exist and how many is open. The placeholder: five skills, range 0 to 20, base values random in 2..8.

| Skill | Covers |
|---|---|
| `brawn` | Physical strength, intimidation, standing your ground |
| `wits` | Scheming, lying, seeing through a trick |
| `charm` | Persuading, comforting, winning someone over |
| `means` | Money, credit, access to goods |
| `craft` | Fixing, piloting, technical help |

### 5.2 Traits `[D 6.1]` `[D 1.3]`

Traits come in two kinds:

- **External traits**: visible on everyone at once. Strength, frailty, wealth; also professions `[D 6.8]` and injuries `[D 6.12]`.
- **Character traits**: hidden until revealed by a counter conversation (section 6.4), the event window (section 6.5) or a rumour.

Each resident gets `[GB]`: one profession, 0–1 body or wealth trait, 2 character traits, and possibly injuries.

Authoring rule `[GB]`: a trait should make it guessable which skill or scale it moves. A "Hulking" person should be strong; the player must be able to plan from traits. (See section 17, item 4.)

### 5.3 Professions `[D 6.8]`

A profession is an external trait. It shifts personality and skills and sets the routine: on which days and at what hour the person comes to the bar. How exactly the routine works is open `[GB, open 2.2.2]`; the placeholder is the schedule table in section 12.3: allowed days, an arrival window and a visit chance.

### 5.4 Injuries `[D 6.12]`

An injury is an external trait, for example "Limping" or "Scarred". Injuries come from plot outcomes and fights.

### 5.5 Needs and goals `[D 6.4]` `[D 6.7]`

- There are no need scales. A need comes from a plot: the plot's owner role carries it.
- Goals live in plot roles. People without a role have no goal and start no intrigues. `[D 6.10]`

### 5.6 Population `[D 1.6]`

- At the start of the season the game generates **14 residents** `[GB]`.
- Old relations and family are set at generation `[D 7.2]`: **2 family pairs, 3 old friendships, 1 old rivalry** `[GB]`. Obligations: **2 to 3** `[GB]` `[D 7.4]`.
- **Plot strangers** appear only when a plot needs them (a bounty hunter, a racketeer) and leave when their part is over. `[D 2.2]`
- **Dark horses** are strangers who walk in without a reason. `[D 2.2.1]` See section 5.7.
- Background departures and arrivals run as in `[D 6.12]`: now and then someone without a role leaves the station and a new generated person arrives later. A person with a role never leaves this way. Greybox values `[GB]`: one departure check every **4 days ±1**; a replacement arrives **1 to 2 days** later.

### 5.7 Guest flow `[D 2.2]` `[D 2.2.1]`

The day's flow mixes three kinds of guests: mostly plot guests, some regulars, a few dark horses. The player does not know in advance who will come. `[D 2.2.1]`

- **Plot guests.** Every resident who holds a role in an active plot comes to the bar every shift. `[D 2.2]` Plot strangers do the same while their plot is active `[GB]`.
- **Regulars.** Residents without a role come by their profession's schedule `[GB, open 2.2.2]`.
- **Dark horses.** Strangers generated on arrival: **2 per shift** `[GB]`, each with 1–2 external traits that hint at useful skills, and hidden character traits. A dark horse who takes a role in a plot becomes a resident; otherwise they never come back `[D 2.2.1]`.

**Arrival rule** `[GB, open 2.2.3 and 2.2.4]`:

1. At the start of each shift the game builds the list of expected guests: all plot guests, the regulars who pass their schedule roll, and the dark horses. Each gets an arrival time inside their window (plot guests and dark horses: anywhere in the first 6 hours of the shift).
2. A guest whose arrival time has come waits off-screen as pending.
3. Whenever a counter seat is free and at least **15 game minutes** `[GB]` have passed since the last arrival, one pending guest walks in. Priority: plot guests whose need has the nearest deadline first, then other plot guests, then regulars and dark horses by arrival time.
4. Pending guests who never get a seat before closing do not come that day. The player never sees the pending list (except in the debug overlay).

So a need holder "always comes" only if the player keeps a counter seat free for them. This is the placeholder for the open conflict between `[D 2.2]` and `[D 9.2]` (card 2.2.4).

Balance target `[GB, open 2.2.3]`: about **10 to 14** expected guests per shift, enough that the player always has to choose whom to let go.

---

## 6. Information and fog of war

### 6.1 Layers `[D 1.3]`

| Layer | What | How the player learns it |
|---|---|---|
| Surface | Name, profession, external traits, injuries, mood by the face | Visible at once |
| Character | Character traits | Counter conversation, behaviour signals, event window after a meeting, rumours |
| Secrets and plans | Needs, plot roles, secret relations, a plot's second floor | Counter conversation and rumours |
| Never | Personality and skill values | Not shown (debug overlay only) |

"Who the person came with" `[D 1.3]` is not in this build: guests arrive one by one `[GB]`. Companies are in the next update `[D 7.3]`.

**Mood by the face** `[GB]`: counter timer below 25% → impatient; holds a need whose deadline is less than 1 day away → worried; otherwise → neutral.

### 6.2 Knowledge model `[GB]`

The game tracks which facts the player knows. A fact is one of: a person has trait X; a person has need N; A and B have relation R; A owes B; plot P ended with outcome O. Popups and the epilogue show only known facts.

### 6.3 Guest popup `[D 3.2]`

- Hovering any guest, at the counter or at a table, shows a popup with what the player needs to decide. `[D 3.2]`
- Right click pins the popup so it stays open for study; right click again or Esc unpins. `[D 3.2]`

Popup contents `[GB, open 3.2.1]`:

- Name, profession, external traits, injuries.
- Known character traits; unknown slots shown as "?".
- The guest's known need, if any: short text and days left.
- Counter timer left (counter guests only).
- The drink poured this visit, or "No drink yet".
- State: waiting at the counter, waiting alone at a table, in a meeting.

Relations and opinions are not shown in the popup in this build. Where the player sees opinion is in the next update. `[D 7.1.1]`

### 6.4 Counter conversation `[D 2.9]`

- The player starts a conversation by clicking a counter guest. The game pauses while the player reads; afterwards the fast-forward of section 3.3 plays.
- The guest decides what to talk about, by a ladder. Each conversation takes the first step that applies:
  1. **Plot or need.** The guest's current plot step has talk the player has not heard yet. Telling it reveals the need (and the plot role) as a fact.
  2. **A character trait.** The guest still has an unknown character trait. The conversation reveals one.
  3. **A rumour.** The guest knows a rumour the player does not: about someone else's need or trait (section 6.6). The conversation reveals its fact.
  4. Otherwise: small talk, flavour only `[GB]`.
- A guest can be talked to more than once per visit; each conversation costs time `[GB]`.
- While the player listens to one guest, the others' timers keep running (through the fast-forward).

**Second floor** `[D 2.5]`: a plot step's `second_floor` text replaces the normal text in a special moment. Trust is in the next update `[D 2.4]` and intoxication is cut `[D 10.4]`, so in this build the only trigger is despair: the guest's need deadline is less than **1 day** away `[GB]`.

### 6.5 Event window after a meeting `[D 3.9]`

When a meeting ends: both guests play a leaving animation, then a modal event window opens, as in CK3. It pauses the game and costs no time.

The window shows:

- Who met.
- What happened: the text of the chosen option and its result (success or failure).
- **The trait that decided it.** The game picks it `[GB]`:
  - If the option had a skill check: the acting person's trait (or drink) with the largest shift on that skill in the direction of the result (positive on success, negative on failure).
  - Otherwise, or if no trait moved that skill: the trait (or drink) that added most to the chosen option's weight.
  - If nothing qualifies, no trait is shown.
- The shown trait becomes known to the player, if it was not already. A drink is named as such ("the Weld did the talking").

This is the main way the player learns consequences. `[D 3.6]`

### 6.6 Rumours `[D 3.4]`

- A rumour is true but incomplete: it carries one real fact, never distorted.
- Rumours do not spread on their own. They are born from meetings and plot outcomes and move only between people who meet. `[D 3.4]`
- The player hears rumours only in counter conversations (ladder step 3). `[D 2.9]`

Greybox rules `[GB]`:

- When a plot starts with a need, the game seeds a rumour about that need into 1–2 residents who have a relation with the need holder or share their profession.
- When a meeting reveals a deciding trait, the game seeds a rumour about that trait into the other participant.
- An outcome may seed a rumour (section 11.4). Off-screen outcomes always do.
- When two guests meet at a table, each passes every rumour they know to the other.
- If a rumour's seed person is unbound, gone, or is the rumour's own subject, the rumour goes to a random resident who has a relation with the subject, otherwise to any random resident.

### 6.7 Snippets `[D 3.6]`

Snippets are short speech bubbles during meetings and are consequences of meetings, not background chatter. Each meeting option and meeting event may carry 1–2 snippet lines shown while the meeting runs `[GB]`. Every snippet goes into the snippet log with the game time. Snippets are flavour and reveal no facts `[GB]`.

### 6.8 Behaviour signals `[D 3.7]`

Signals are behaviour only, with no labels: a person fidgets, glances at the door. Implementation `[GB]`: every character trait and plot step can list signals; while the person is in the bar, every **20 to 40 game minutes** one of their signals plays as a short unlabeled emote line in a bubble. A signal never adds a fact.

---

## 7. Relationships

### 7.1 Opinion `[D 7.1]`

As in CK3: opinion is a number made of reasons, each with its own weight and duration. Opinion is directed; range −100..+100, clamped `[GB]`. A reason has an id, a text, a value and an expiry date or none.

Opinion is used in plot conditions and option weights. Where the player sees it, and what place it takes in the new meeting scheme, is in the next update `[D 7.1.1]`: in this build it is visible only in the debug overlay `[GB]`.

Sources `[GB]`: compatibility (permanent, computed once per pair as in v0.1: average absolute difference of base scales plus trait shifts, mapped from 0 → +20 to 80+ → −20), relations (section 7.2), meeting events and plot outcomes.

### 7.2 Relations `[D 7.2]`

Friend, enemy, lover, mentor, family: separate relations on top of opinion, as in CK3. They add a big reason to opinion and last longer than opinion swings.

- Family and old relations are set at generation.
- New relations come only from plot outcomes. `[D 7.2]` Whether plain meetings may create them is open (card 6.10.1); in this build they may not `[GB, open 6.10.1]`.
- A relation can be secret, for example lovers. It stays unknown until a rumour or a plot reveals it.

Opinion reasons per relation `[GB]`: friend +40, enemy −50, lover +60, mentor/student +30, family +30.

### 7.3 Obligations `[D 7.4]` `[D 4.5]`

- An obligation is one more relation type: "owes". Money and a favour are the same thing here.
- It appears at generation or from a plot outcome, adds an opinion reason (debtor → creditor −10, creditor → debtor −5 `[GB]`), and lives until the debt is closed. There are no small debts.
- The game considers it when it casts roles and weighs options. A debtor's behaviour changes by character; there is no separate action. `[D 4.5]` Greybox rule `[GB]`: options that pay a debt back get +30% weight when the debtor's `honor ≥ 20`.

### 7.4 Reputation `[D 3.5]`

One reputation for everybody. Loud events in the bar change it. Greybox values `[GB]`: range 0..100, start 50; a fight −5. Effect: regulars' visit chance is multiplied by `0.75 + reputation / 200`.

---

## 8. Story engine

### 8.1 Plots, roles and needs `[D 1.6]` `[D 6.4]`

Writers write plots with roles like "debtor", "runaway", "lonely heart". A plot has three kinds of roles:

| Role kind | Who fills it | When |
|---|---|---|
| **Owner** | The game. The owner carries the plot's need. | When the plot starts |
| **Game-cast** | The game: a resident chosen by conditions, or a newly generated plot stranger | When the plot starts |
| **Player-cast** | The player, by seating: **anyone whose traits fit** the role and who holds no other role | When the meeting that needs the role starts |

The game sets only the owner and the parts the story needs to exist (the creditor of a debt, the hunter on someone's trail). Everyone else the player casts by seating. The same need with a different person gives a different story. `[D 1.6]`

A player-cast role has a `fits` condition, written with traits and professions `[GB]`. It is checked against the person's real traits, not against what the player knows `[GB]`: knowing helps the player, but a lucky guess works too.

### 8.2 Needs with a deadline `[D 2.8]` `[D 2.2]`

- A plot step can be a **need**: it has a deadline in days. Default: **2 days** `[GB]`.
- While the plot is active, every role holder comes to the bar every shift (section 5.7).
- The need is solved by a meeting at a table (section 8.4). If its deadline passes, the step's `on_timeout` happens off-screen. Success and expiry change the world differently. `[D 2.8]`
- A meeting result may move the plot to another step (a new need with its own deadline), keep the current step, or end the plot with an outcome.

### 8.3 Where new plots come from `[GB, open 8.1]`

Where new needs come from, and how soon after an outcome a person gets a new one, is open. The placeholder:

- At the start of the season, **3 plots** start.
- Between shifts `[D 1.5]`, the game starts new plots: at most **2 per night**, with at most **6 active plots** at once.
- Candidates: plots not on cooldown whose owner and game-cast roles can all be bound. Roles are bound in order; each later role may refer to earlier ones; simple backtracking as in v0.1. Among candidates the game picks by plot weight; among people for a role, by the role's weight.
- A person can become an owner again only **2 days** after their previous plot ended.
- In the bar, nothing starts plots. Quarrels, fights and rumours come only from meetings. `[D 9.3]`

### 8.4 Meetings `[D 6.10]` `[D 9.3]`

When a second guest is seated at a table, the game classifies the meeting:

1. **Plot meeting.** Both guests hold roles in the same plot, and the plot's current step lists a meeting for that pair of roles. The plot is guaranteed to move. `[D 6.10]`
2. **Need meeting.** One guest holds a role in a plot whose current step lists a meeting between that role and a player-cast role; the other guest holds no role and fits that player-cast role. The game binds the other guest to the role. This also moves the plot, guaranteed.
3. **Plain meeting.** Anything else. The two interact by their traits, and not always well (section 8.6).

If more than one plot meeting or need meeting matches, the one with the nearest deadline wins `[GB]`.

### 8.5 Resolving a plot or need meeting `[D 6.10]`

It happens in two steps, as in CK3: the character decides, then a skill check decides whether it works.

1. **Decision.** The meeting names an **actor** role and lists **options**. Options whose `requires` condition fails are skipped. Each remaining option has a weight from the actor's effective personality `[GB]`:
   ```
   weight = max(0, base + Σ effective_scale × factor)   # factors per option, from data
   ```
   plus modifiers such as the obligation rule in section 7.3. The game picks one option by weighted random `[GB]`. If every weight is 0, it picks the first available option.
2. **Check.** The chosen option may carry a skill check. The type is set per option `[D 6.10]`:
   - `threshold`: success if the actor's effective skill ≥ `target`.
   - `roll`: success chance = clamp(`base` + effective skill × `per_point`, 0.05, 0.95) `[GB]`.
   - none: always success.
3. **Result.** Success or failure has its own text, effects and next step:
   - `outcome:<id>` ends the plot with that outcome;
   - `step:<id>` enters that step; entering a need step starts its deadline anew;
   - `stay` keeps the current step and deadline. A player-cast role bound by this meeting is released, so another person can be tried next time `[GB]`.
4. The meeting runs for its duration (section 4.3), with the option's snippets. Effects apply when it ends. Both guests leave; the event window shows the result and the deciding trait (section 6.5).

Drinks change both steps: they shift personality, which changes option weights, and skills, which change checks. `[D 10.3]`

### 8.6 Plain meetings `[GB, open 6.10.1]`

How a meeting between two people outside a common plot branches is open. The placeholder: the game picks one **meeting event** (section 11.6) by weight among those whose conditions hold for the pair. Weights use both people's personality and their compatibility. Meeting events may change opinion, start a fight, injure someone or change reputation. They do not create new relations in this build, because `[D 7.2]` allows new relations only from plot outcomes.

The event window shows the event's text and the deciding trait, as for plot meetings.

### 8.7 Off-screen: timers and outcomes `[D 8.2]` `[D 8.3]`

- Plot deadlines run in game time, during shifts and between them.
- When a deadline passes, `on_timeout` happens off-screen. Its outcome applies its effects and seeds its rumour; the player learns of it from rumours and talk. `[D 8.3]`
- The outcome of a meeting the player saw is shown at once in the event window. `[D 8.3]`
- A participant coming back to tell how a story ended is in the next update. `[D 8.3.1]`
- After an outcome, all roles are freed. Dark horses who held a role become residents `[D 2.2.1]`; plot strangers leave the station `[GB]`.

### 8.8 One role at a time `[D 8.6]`

- A person holds one role at a time. A person with a role cannot be bound to a player-cast role of another plot; seating them with a need holder of another plot gives a plain meeting.
- A person with a role never leaves the station through the background. `[D 6.12]`
- Stories collide through seating and through consequences: one plot's outcome changes relations and obligations, and new plots find conditions in them.

### 8.9 Empty role `[D 8.6.1]`

If a role holder dies or leaves, the plot ends with its special "someone is gone" outcome (`on_vacant`). It is an ordinary outcome under section 8.7.

### 8.10 Delayed events `[D 8.5]`

As in CK3, an outcome can schedule an event for later: "in two weeks they will come for him". A scheduled event stores its bindings, its earliest day and its latest day `[GB]`. Delayed events are checked once per day, between shifts `[GB]`: if the event is due and its conditions hold, its effects apply; if its latest day passes first, it is dropped.

---

## 9. Drinks

### 9.1 Menu `[D 10.1]` `[D 10.2]`

A small fixed menu, six to eight drinks, all available from the first evening. A drink is picked from the menu; there is no mixing. Quality, signature cocktails and a growing menu are in the next update `[D 10.1.1]`.

### 9.2 A drink is a temporary trait `[D 10.3]`

- A drink shifts personality, skills, or both, in any combination. Special effects come later.
- It is poured only at the counter, at most once per guest per visit.
- It lasts until the end of the guest's meeting, or until the guest leaves.
- It counts as a trait for the "deciding trait" of the event window (section 6.5).
- A poured drink may trigger a short line; in this build, flavour only `[GB]`.

Intoxication is cut `[D 10.4]`. There is no drink counter. Conditions may test which drink a guest got this visit (section 11.2).

Why a player would choose not to pour is in the next update `[D 10.3.1]`.

---

## 10. Season end and epilogue

- The season ends after the last shift of day 28 `[GB]`. `[D 1.7]`
- The epilogue screen `[GB]` lists: every plot that ended this season and its outcome as the player knows it (unknown outcomes show as "You never found out how this ended"); who left or died; the relations and obligations the player knows at the end; final reputation.
- A per-shift summary screen is in the next update `[D 14.1]`. The epilogue format (block 14) is a placeholder too.

---

## 11. Data formats

Everything in this section is `[GB]`: a proposed format, not a design decision. Claude Code may adjust field names, but must keep content in data files.

### 11.1 Files

```
res://data/tuning.json          all [GB] numbers (section 13)
res://data/scales.json          personality scale ids and names (debug only)
res://data/skills.json          skill ids, names, range (debug only)
res://data/traits.json          character traits, external traits, professions, injuries
res://data/drinks.json          the menu
res://data/names.json           name pools
res://data/plots/*.json         one file per plot
res://data/meeting_events.json  plain-meeting events
res://data/delayed_events.json  delayed events
```

### 11.2 Conditions

A condition is a JSON object. Role names start with `$` and refer to people bound in the current plot or event. Inside a `fits` condition, the role's own id refers to the person being tested.

| Condition | Meaning |
|---|---|
| `{"all": [c1, c2]}` / `{"any": [...]}` / `{"not": c}` | Logic |
| `{"trait": ["$x", "greedy"]}` | Has the trait (any kind: character, external, profession, injury) |
| `{"profession": ["$x", "pilot"]}` | Profession |
| `{"scale": ["$x", "boldness", ">=", 20]}` | Effective personality scale, drink included |
| `{"skill": ["$x", "brawn", ">=", 10]}` | Effective skill, drink included |
| `{"opinion": ["$a", "$b", ">=", 30]}` | A's opinion of B |
| `{"compatible": ["$a", "$b"]}` | Compatibility > 0 (section 7.1) |
| `{"relation": ["$a", "$b", "friend"]}` | Relation exists |
| `{"owes": ["$a", "$b"]}` | A owes B |
| `{"drink": ["$x", "weld"]}` | Got this drink this visit |
| `{"day": [">=", 5]}` | Season day |

Implicit rules for every resident role: alive, on the station, holds no other role `[D 8.6]`.

### 11.3 Effects

| Effect | Meaning |
|---|---|
| `{"opinion": {"from": "$a", "to": "$b", "id": "bailed_me_out", "text": "Bailed me out", "value": 25, "days": 30}}` | Add an opinion reason (no `days` = permanent) |
| `{"remove_opinion": {"from": "$a", "to": "$b", "id": "..."}}` | Remove a reason |
| `{"relation": {"a": "$a", "b": "$b", "type": "lover", "secret": true}}` | Add a relation (plot outcomes only, `[D 7.2]`) |
| `{"end_relation": {"a": "$a", "b": "$b", "type": "friend"}}` | Remove a relation |
| `{"obligation": {"debtor": "$a", "creditor": "$b"}}` | Add "owes" |
| `{"clear_obligation": {"debtor": "$a", "creditor": "$b"}}` | Close it |
| `{"add_trait": {"who": "$x", "trait": "scarred"}}` | Add a trait (injuries) |
| `{"kill": "$x"}` / `{"depart": "$x"}` | Remove from the station |
| `{"reputation": -5}` | Change reputation |
| `{"schedule": {"event": "theft_discovered", "days": [5, 12], "chance": 0.5, "bind": {"$mark": "$mark", "$thief": "$thief"}, "latest_days": 20}}` | Delayed event `[D 8.5]` |
| `{"rumour": {"seed": "$x", "text": "...", "fact": {...}}}` | Create a rumour known first by `$x` |
| `{"reveal": {...fact...}}` | The player learns a fact |
| `{"chance": 0.3, "effects": [...]}` | Apply the inner effects with this chance |
| `{"start_plot": {"plot": "protection", "bind": {"$shopkeeper": "$shopkeeper"}, "ignore_cooldown": true}}` | Start a plot with some roles pre-bound (used by delayed events) |

Facts: `{"type": "trait", "who": "$x", "trait": "greedy"}`, `{"type": "need", "who": "$x"}`, `{"type": "relation", "a": "$a", "b": "$b", "rel": "lover"}`, `{"type": "obligation", "debtor": "$a", "creditor": "$b"}`, `{"type": "outcome"}` (the current plot's outcome).

### 11.4 Plot format

```jsonc
{
  "id": "...", "title": "...",
  "weight": 10,                 // chance to be picked among candidates (section 8.3)
  "cooldown_days": 6,           // before this plot can start again
  "roles": [
    { "id": "$owner", "cast": "game", "owner": true, "pool": "resident", "where": {...}, "weight": {"base": 1, "scales": {...}} },
    { "id": "$partner", "cast": "game", "pool": "resident" /* or "stranger" */, "where": {...} },
    { "id": "$stranger", "cast": "game", "pool": "stranger", "make": {"profession": "bounty_hunter", "traits": ["cynical"]} },
    { "id": "$helper", "cast": "player", "fits": {...} }      // bound when a need meeting starts
  ],
  "start": "need",
  "steps": {
    "need": {
      "deadline_days": 2,                                      // makes this step a need
      "talk": [ { "speaker": "$owner", "text": "...", "second_floor": "...", "time_cost": 30 } ],
      "signals": { "$owner": ["*glances at the door*"] },
      "on_enter": [ /* effects */ ],
      "meetings": [
        { "pair": ["$owner", "$helper"], "actor": "$helper", "options": [
          { "id": "...", "text": "...",
            "requires": {...},                                 // optional
            "weight": {"base": 10, "scales": {"compassion": 0.4}},
            "check": {"type": "threshold", "skill": "means", "target": 10},   // or roll, or none
            "snippets": ["..."],
            "success": { "text": "...", "effects": [...], "goto": "outcome:..." },
            "failure": { "text": "...", "effects": [...], "goto": "stay" } }
        ]}
      ],
      "on_timeout": "outcome:..."
    }
  },
  "outcomes": {
    "...": { "effects": [...], "rumour": { "seed": "$partner", "text": "...", "fact": {...} }, "epilogue": "..." }
  },
  "on_vacant": { "$owner": "outcome:owner_gone", "$partner": "outcome:partner_gone" }
}
```

Rules:

- `cast: "game"` roles are bound when the plot starts (section 8.3). `cast: "player"` roles are bound when a need meeting starts (section 8.4).
- A stranger role creates a new person with the given profession and traits. Plot strangers come to the bar every shift while the plot is active and leave the station when it ends.
- `talk` is what each speaker says the next time the player talks to them at the counter (ladder step 1, section 6.4), once per step. Hearing it reveals the speaker's need `[GB]`. `second_floor` replaces `text` in despair (section 6.4).
- `meetings` lists which pairs of roles move this step. `pair` is unordered. `actor` decides.
- `goto` targets: `outcome:<id>`, `step:<id>`, `stay`.
- An outcome frees all roles. If it was reached off-screen, its rumour is seeded; if reached in a meeting, the rumour is seeded too and the event window has already shown the result.
- `epilogue` is the line shown on the epilogue screen if the player knows the outcome.
- Text templates: `{debtor}` → the person's name, `{debtor.profession}` → profession name.

### 11.5 Delayed event format

```jsonc
{ "id": "theft_discovered", "where": {...}, "effects": [...] }
```

Bindings come from the `schedule` effect.

### 11.6 Meeting event format (plain meetings)

```jsonc
{
  "id": "argument",
  "where": { "opinion": ["$a", "$b", "<=", -10] },   // $a and $b are the two guests, both orders are tried
  "weight": { "base": 10, "a": {"boldness": 0.2}, "b": {"boldness": 0.2}, "compatibility": -0.5 },
  "text": "{a} and {b} start arguing about nothing, then about everything.",
  "snippets": ["You always do this.", "Do what?"],
  "effects": [ ... ]
}
```

`compatibility` multiplies the pair's compatibility (−20..+20) into the weight.

---

## 12. Sample content

All content in this section is `[GB]`: written to make the greybox playable, not decided by the team. Writers will replace it.

### 12.1 Personality scales and skills

Scales: `boldness`, `compassion`, `greed`, `honor`, `sociability`, `rationality` (section 5.1).
Skills: `brawn`, `wits`, `charm`, `means`, `craft` (section 5.1, `[GB, open 6.8.2]`).

### 12.2 Character traits (hidden until revealed)

No opposite pairs `[D 6.1]`.

| id | Name | Personality | Skills | Signals |
|---|---|---|---|---|
| `chatterbox` | Chatterbox | sociability +25 | charm +2 | *talks to whoever sits nearest* |
| `hot_headed` | Hot-headed | boldness +15, rationality −20 | brawn +1 | *puts the glass down too hard* |
| `greedy` | Greedy | greed +25, compassion −5 | — | *counts the change twice* |
| `kind_hearted` | Kind-hearted | compassion +25 | — | *slides a napkin to a neighbour* |
| `proud` | Proud | honor +15, sociability −10 | — | *sits very straight, never looks around* |
| `reckless` | Reckless | boldness +25, rationality −10 | — | *rocks the chair on two legs* |
| `romantic` | Romantic | sociability +10, compassion +10, rationality −10 | charm +2 | *keeps glancing across the room* |
| `loyal` | Loyal | honor +25 | — | *keeps the seat beside them free* |
| `cynical` | Cynical | compassion −15, rationality +10 | wits +2 | *snorts at the next table's toast* |
| `ambitious` | Ambitious | greed +10, boldness +10, honor −5 | wits +1 | *taps a datapad, impatient* |
| `sly` | Sly | honor −10, rationality +5 | wits +4 | *answers a question with a question* |

### 12.3 External traits and professions (visible at once)

Body and wealth traits:

| id | Name | Personality | Skills |
|---|---|---|---|
| `hulking` | Hulking | boldness +5 | brawn +6 |
| `frail` | Frail | boldness −5 | brawn −4 |
| `well_off` | Well-off | greed +5 | means +6 |
| `shabby` | Shabby | — | means −3 |

Professions `[D 6.8]`; schedule `[GB, open 2.2.2]`. Station days run 0..6; day 6 is the rest day.

| id | Name | Personality | Skills | Comes | Arrives | Visit chance |
|---|---|---|---|---|---|---|
| `dock_worker` | Dock worker | boldness +10, sociability +10 | brawn +3 | days 0–5 | 19:00–21:00 | 70% |
| `pilot` | Pilot | boldness +15, rationality +5 | craft +4 | any day | 22:00–01:00 | 40% |
| `mechanic` | Mechanic | rationality +10, sociability −5 | craft +6 | days 0–6 | 00:00–02:00 | 60% |
| `clerk` | Station clerk | honor +10, boldness −10 | wits +3 | days 0–4 | 18:00–19:30 | 50% |
| `medic` | Medic | compassion +15, sociability −5 | craft +3, charm +1 | days 0–6 | 02:00–03:00 | 50% |
| `trader` | Trader | greed +15, sociability +10 | means +4, charm +2 | days 1–6 | 20:00–23:00 | 60% |
| `miner` | Ore miner | boldness +10, rationality −5 | brawn +4 | 3 days on, 1 off | 18:30–20:30 | 80% |

Stranger-only professions, used by plots:

| id | Name | Personality | Skills |
|---|---|---|---|
| `bounty_hunter` | Bounty hunter | boldness +20, compassion −10 | brawn +5, wits +3 |
| `racketeer` | Racketeer | greed +15, boldness +15 | brawn +4, means +2 |

Dark horses `[GB]`: a random resident profession, one guaranteed body or wealth trait, two hidden character traits.

### 12.4 Injuries `[D 6.12]`

| id | Name | Personality | Skills |
|---|---|---|---|
| `limping` | Limping | boldness −10 | brawn −2 |
| `scarred` | Scarred | sociability −5, boldness +5 | — |

### 12.5 Drinks `[D 10.1]` `[D 10.3]`

Seven drinks, all available from day 1.

| id | Name | Personality | Skills | Line when poured |
|---|---|---|---|---|
| `weld` | Weld | boldness +25, rationality −10 | brawn +2 | "That burns. Good." |
| `zero_g` | Zero-G | boldness +10, sociability +10 | charm +1 | "Feels lighter already." |
| `station_water` | Station Water | rationality +15, sociability −5 | wits +2 | "Clear head. Fine." |
| `black_hole` | Black Hole | compassion +15, boldness −5 | — | "Sweet. Too sweet. Thanks." |
| `red_giant` | Red Giant | sociability +25, rationality −10 | charm +2 | "Now we're talking." |
| `cold_vacuum` | Cold Vacuum | compassion −15, rationality +10 | wits +1 | "Cold. Like this place." |
| `oath` | Oath | honor +20, greed −10 | — | "An oath drink? Who am I promising?" |

### 12.6 Plots

Five plots. Plot A is given in full JSON as the reference. Plots B to E follow the same format; their structure is given compactly, and Claude Code writes them out as JSON. Every plot has an owner with a need, at least one player-cast role, and at least one meeting the player can arrange.

#### Plot A — Calling in the Debt (full)

The debtor needs money. The creditor is cast by the game. The helper is whoever the player seats with the debtor: anyone well-off or a trader who isn't owed already. Seating the debtor with the creditor gives a different meeting.

```json
{
  "id": "calling_in_the_debt",
  "title": "Calling in the Debt",
  "weight": 10,
  "cooldown_days": 6,
  "roles": [
    { "id": "$debtor", "cast": "game", "owner": true, "pool": "resident",
      "where": {"any": [{"trait": ["$debtor", "greedy"]}, {"trait": ["$debtor", "reckless"]}, {"scale": ["$debtor", "greed", ">=", 10]}]},
      "weight": {"base": 1, "scales": {"greed": 0.5}} },
    { "id": "$creditor", "cast": "game", "pool": "resident",
      "where": {"owes": ["$debtor", "$creditor"]},
      "weight": {"base": 1} },
    { "id": "$helper", "cast": "player",
      "fits": {"all": [
        {"any": [{"trait": ["$helper", "well_off"]}, {"profession": ["$helper", "trader"]}]},
        {"not": {"owes": ["$debtor", "$helper"]}}
      ]} }
  ],
  "start": "need",
  "steps": {
    "need": {
      "deadline_days": 2,
      "talk": [
        { "speaker": "$debtor",
          "text": "{creditor} wants the money back. All of it. I need someone with money, someone I don't already owe.",
          "second_floor": "If I can't pay, I'm gone. I already looked at the shuttle schedule. Don't tell {creditor}.",
          "time_cost": 30 },
        { "speaker": "$creditor",
          "text": "{debtor} owes me. I'm a patient person. Up to a point.",
          "time_cost": 20 }
      ],
      "signals": { "$debtor": ["*glances at the door every time it opens*", "*orders, then forgets the drink*"] },
      "meetings": [
        { "pair": ["$debtor", "$helper"], "actor": "$helper", "options": [
          { "id": "pay_off", "text": "{helper} offers to pay off the debt.",
            "weight": {"base": 10, "scales": {"compassion": 0.4, "greed": -0.3}},
            "check": {"type": "threshold", "skill": "means", "target": 10},
            "snippets": ["How much?", "Don't make that face. I'll cover it."],
            "success": { "text": "{helper} paid {creditor} off. Now {debtor} owes {helper} instead, but can breathe.",
                         "goto": "outcome:bailed_out" },
            "failure": { "text": "{helper} wanted to help, but doesn't have that kind of money.",
                         "effects": [ {"opinion": {"from": "$debtor", "to": "$helper", "id": "tried_to_help", "text": "Tried to help", "value": 10, "days": 14}} ],
                         "goto": "stay" } },
          { "id": "loan_at_interest", "text": "{helper} offers a loan, at a price.",
            "weight": {"base": 10, "scales": {"greed": 0.5}},
            "check": {"type": "threshold", "skill": "means", "target": 10},
            "snippets": ["Twenty percent. A week.", "That's robbery.", "That's business."],
            "success": { "text": "{helper} lent the money at a steep rate. {debtor} is off one hook and on another.",
                         "goto": "outcome:new_hook" },
            "failure": { "text": "{helper} talked big about a loan, but there was no money behind it.",
                         "goto": "stay" } },
          { "id": "turn_away", "text": "{helper} wants nothing to do with it.",
            "weight": {"base": 5, "scales": {"compassion": -0.4, "rationality": 0.2}},
            "snippets": ["Not my problem."],
            "success": { "text": "{helper} heard {debtor} out and said no.",
                         "effects": [ {"opinion": {"from": "$debtor", "to": "$helper", "id": "turned_me_away", "text": "Turned me away", "value": -10, "days": 14}} ],
                         "goto": "stay" } }
        ]},
        { "pair": ["$debtor", "$creditor"], "actor": "$debtor", "options": [
          { "id": "beg_for_time", "text": "{debtor} begs {creditor} for more time.",
            "weight": {"base": 10, "scales": {"honor": 0.3, "boldness": -0.2}},
            "check": {"type": "roll", "skill": "charm", "base": 0.2, "per_point": 0.04},
            "snippets": ["Two more days. That's all I'm asking."],
            "success": { "text": "{creditor} gave {debtor} two more days.", "goto": "step:need" },
            "failure": { "text": "{creditor} was done waiting.", "goto": "outcome:collected" } },
          { "id": "skip_station", "text": "{debtor} decides to run.",
            "weight": {"base": 5, "scales": {"boldness": 0.4, "rationality": -0.3}},
            "snippets": ["Sure. Tomorrow. I'll have it tomorrow."],
            "success": { "text": "{debtor} promised everything and took the night shuttle.", "goto": "outcome:fled" } },
          { "id": "pay_with_favour", "text": "{debtor} offers a favour instead of money.",
            "weight": {"base": 5, "scales": {"greed": 0.3}},
            "check": {"type": "roll", "skill": "wits", "base": 0.2, "per_point": 0.04},
            "snippets": ["What if I could get you something better than money?"],
            "success": { "text": "{debtor} talked {creditor} into taking a favour instead.", "goto": "outcome:favour" },
            "failure": { "text": "{creditor} saw through the offer.", "goto": "outcome:collected" } }
        ]}
      ],
      "on_timeout": "outcome:collected"
    }
  },
  "outcomes": {
    "bailed_out": {
      "effects": [
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
        {"obligation": {"debtor": "$debtor", "creditor": "$helper"}},
        {"opinion": {"from": "$debtor", "to": "$helper", "id": "bailed_me_out", "text": "Bailed me out", "value": 25, "days": 30}}
      ],
      "rumour": {"seed": "$creditor", "text": "Heard someone paid off {debtor}'s debt.",
                 "fact": {"type": "obligation", "debtor": "$debtor", "creditor": "$helper"}},
      "epilogue": "{helper} paid {debtor}'s debt to {creditor}."
    },
    "new_hook": {
      "effects": [
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
        {"obligation": {"debtor": "$debtor", "creditor": "$helper"}},
        {"opinion": {"from": "$debtor", "to": "$helper", "id": "charges_interest", "text": "Charges interest", "value": -10, "days": 30}}
      ],
      "rumour": {"seed": "$creditor", "text": "{helper} is lending money now. At a price.",
                 "fact": {"type": "obligation", "debtor": "$debtor", "creditor": "$helper"}},
      "epilogue": "{debtor} swapped one creditor for another: {helper}."
    },
    "favour": {
      "effects": [
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
        {"opinion": {"from": "$creditor", "to": "$debtor", "id": "paid_in_kind", "text": "Paid in kind", "value": 5, "days": 30}}
      ],
      "epilogue": "{debtor} settled with {creditor} by a favour, not money."
    },
    "fled": {
      "effects": [
        {"depart": "$debtor"},
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}}
      ],
      "rumour": {"seed": "$creditor", "text": "A {debtor.profession} left on the night shuttle. Owed money, they say.",
                 "fact": {"type": "outcome"}},
      "epilogue": "{debtor} fled the station rather than pay."
    },
    "collected": {
      "effects": [
        {"add_trait": {"who": "$debtor", "trait": "limping"}},
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
        {"opinion": {"from": "$debtor", "to": "$creditor", "id": "had_me_beaten", "text": "Had me beaten", "value": -40, "days": 60}}
      ],
      "rumour": {"seed": "$creditor", "text": "{creditor} got the money back from someone. The hard way.",
                 "fact": {"type": "outcome"}},
      "epilogue": "{creditor} collected from {debtor}, the hard way."
    },
    "debtor_gone": {
      "effects": [ {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}} ],
      "rumour": {"seed": "$creditor", "text": "{debtor} is gone, and the money with them.", "fact": {"type": "outcome"}},
      "epilogue": "{debtor} was gone before the debt was settled."
    },
    "creditor_gone": {
      "effects": [ {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}} ],
      "rumour": {"seed": "$debtor", "text": "{creditor} is gone. Nobody to pay now.", "fact": {"type": "outcome"}},
      "epilogue": "{creditor} was gone, and {debtor}'s debt with them."
    }
  },
  "on_vacant": { "$debtor": "outcome:debtor_gone", "$creditor": "outcome:creditor_gone" }
}
```

#### Plot B — Lonely Hearts (romance)

- **Roles:**
  - `$lonely` (owner): resident; `romantic` or `sociability ≥ 15`. Weight: base 1, sociability 0.3.
  - `$match` (player-cast): fits `romantic`, `kind_hearted` or `chatterbox`; not family of `$lonely`.
- **Step `need`** (deadline 3 days):
  - Talk (`$lonely`): "Every night I go home to an empty bunk. I'd like to talk to someone who listens. Really listens."
  - Second floor: "I keep thinking it's me. That there's something wrong with me."
  - Signals: *keeps glancing at the door, hopeful*.
  - Meeting `$lonely` + `$match`, actor `$lonely`:
    - `open_up`: base 10, sociability 0.3, boldness 0.3. Roll `charm`, base 0.25, per point 0.03. Success: "{lonely} and {match} talked until closing, then kept talking outside." → `outcome:together`. Failure: "{lonely} said too much too soon. {match} found an excuse to leave." Opinion `$lonely → $match` "Pulled away" −5, 14 days → `stay`.
    - `keep_it_light`: base 8, rationality 0.3, boldness −0.2. No check. "{lonely} kept it light. {match} laughed a lot. Not a romance, but a friend." → `outcome:friends`.
  - Timeout → `outcome:alone`.
- **Outcomes:**
  - `together`: relation `lover`, secret. Opinion both ways "Said it first" / "Heard it first" +10, 30 days. Rumour (seed `$match`): "Someone saw {lonely} and {match} leave together." Reveals the lover relation.
  - `friends`: relation `friend`. Rumour (seed `$match`): "{lonely} and {match} get on well, apparently."
  - `alone`: no effects. Epilogue: "{lonely} spent the season alone."
  - `lonely_gone`, `match_gone` (on vacant): close the plot.

#### Plot C — Night Shuttle (runaway)

- **Roles:**
  - `$runaway` (owner): resident; `reckless` or `boldness ≥ 20`.
  - `$hunter` (game-cast): stranger, profession `bounty_hunter`, trait `cynical`.
  - `$guard` (player-cast): fits `hulking`, or profession `dock_worker` or `miner`.
- **Step `need`** (deadline 2 days):
  - Talk (`$runaway`): "Someone's asking about me. A hunter. I need someone big next to me when they find me."
  - Second floor: "I took something that wasn't mine, a long time ago. They don't forget."
  - Talk (`$hunter`): "Looking for a {runaway.profession} named {runaway}. Old business. You seen them?"
  - Signals: `$runaway` *sits with their back to the wall*; `$hunter` *studies every face that comes in*.
  - Meeting `$runaway` + `$guard`, actor `$guard`:
    - `stand_guard`: base 10, honor 0.3, boldness 0.3. Threshold `brawn ≥ 10`. Success: "{guard} had a quiet word with the hunter. The hunter took the next shuttle out." → `outcome:scared_off`. Failure: "{guard} meant well. The hunter wasn't impressed." → `stay`.
    - `back_off`: base 5, boldness −0.3, rationality 0.3. No check. "{guard} wanted no part of it." Opinion `$runaway → $guard` "Left me hanging" −10, 14 days → `stay`.
  - Meeting `$runaway` + `$hunter`, actor `$runaway`:
    - `run`: base 6, boldness 0.3, rationality −0.2. No check. "{runaway} slipped out the back." → `outcome:fled`.
    - `talk_it_out`: base 6, honor 0.3, rationality 0.2. Roll `charm`, base 0.15, per point 0.04. Success → `outcome:settled`. Failure → `outcome:caught`.
    - `fight`: base 4, boldness 0.4, rationality −0.3. Roll `brawn`, base 0.1, per point 0.04. Success → `outcome:stood_ground`. Failure → `outcome:caught`.
  - Timeout → `outcome:caught`.
- **Outcomes:**
  - `scared_off`: opinion `$runaway → $guard` "Watched my back" +25, 30 days.
  - `fled`: `$runaway` departs. Rumour (seed `$hunter` before leaving; falls back per section 6.6): "{runaway} left the station in a hurry."
  - `settled`: opinion `$runaway → $hunter` "Let it go" +10. Rumour: "{runaway} and some hunter shook hands, of all things."
  - `stood_ground`: `$runaway` gets `scarred`; reputation −5 (a fight in the bar).
  - `caught`: `$runaway` departs. Rumour: "They took {runaway} off the station last night."
  - `runaway_gone`, `hunter_gone` (on vacant): close the plot.
- Plot strangers leave when the plot ends (section 8.7).

#### Plot D — Cargo Bay 7 (theft)

- **Roles:**
  - `$thief` (owner): resident; `greed ≥ 20` and `honor ≤ 0`.
  - `$mark` (game-cast): resident, profession `trader`.
  - `$partner` (player-cast): fits `greedy`, `reckless` or `sly`.
- **Step `need`** (deadline 3 days):
  - Talk (`$thief`): "There's a container in Bay 7 nobody's watching. I need a second pair of hands. Someone who doesn't ask questions."
  - Second floor: "It's {mark}'s. {mark} has it coming."
  - Meeting `$thief` + `$partner`, actor `$partner`:
    - `in`: base 10, greed 0.4, boldness 0.2. Roll `wits`, base 0.3, per point 0.03. Success → `outcome:clean_job`. Failure → `outcome:caught_in_the_act`.
    - `out`: base 8, honor 0.4, rationality 0.2. No check. "{partner} wanted no part of it, and {thief} lost the nerve." → `outcome:called_off`.
  - Timeout → `outcome:called_off`.
- **Outcomes:**
  - `clean_job`: relation `friend` between `$thief` and `$partner`, secret. Schedule `theft_discovered` in 5–12 days, chance 0.5, binding `$mark`, `$thief`. Rumour (seed `$partner`): "Something went missing from Bay 7. Nobody saw anything."
  - `caught_in_the_act`: `$thief` gets `limping`. Opinion `$thief → $partner` "Left me behind" −20, 30 days. Rumour (seed `$mark`): "{thief} got caught poking around Bay 7."
  - `called_off`: no effects.
  - `someone_gone` (on vacant, any role): close the plot.

#### Plot E — Protection (faction flavour `[D 5.5]`)

- **Roles:**
  - `$shopkeeper` (owner): resident, profession `trader`.
  - `$racketeer` (game-cast): stranger, profession `racketeer`.
  - `$backer` (player-cast): fits `hulking` or `hot_headed`.
- **Step `need`** (deadline 2 days):
  - Talk (`$shopkeeper`): "Some new 'friends' want a cut of everything I sell. I need someone tough next to me when they come back."
  - Talk (`$racketeer`): "Nice place. Be a shame if something happened to the people who drink here." The bar itself is not the target `[D 5.4]`.
  - Meeting `$shopkeeper` + `$backer`, actor `$backer`:
    - `stand_by`: base 10, boldness 0.3, compassion 0.2. Threshold `brawn ≥ 10`. Success → `outcome:backed_up`. Failure: "{backer} tried to look dangerous. Nobody was fooled." → `stay`.
    - `ask_for_a_cut`: base 6, greed 0.5. No check. "{backer} will help. For a share." → `outcome:new_protector`.
  - Meeting `$shopkeeper` + `$racketeer`, actor `$shopkeeper`:
    - `pay`: base 8, rationality 0.3, boldness −0.3. No check → `outcome:paid_up`.
    - `refuse`: base 6, boldness 0.4, honor 0.2. Roll `wits`, base 0.1, per point 0.04. Success → `outcome:stood_firm`. Failure → `outcome:beaten`.
  - Timeout → `outcome:paid_up`.
- **Outcomes:**
  - `backed_up`: relation `friend` between `$shopkeeper` and `$backer`. Rumour (seed `$backer`): "{shopkeeper}'s new friends got the message."
  - `new_protector`: obligation `$shopkeeper` owes `$backer`.
  - `paid_up`: opinion `$shopkeeper → $racketeer` "Bleeding me dry" −30. Schedule `they_come_back` in 7–14 days, chance 0.5, binding `$shopkeeper`.
  - `stood_firm`: rumour (seed `$shopkeeper`): "{shopkeeper} told the racketeers where to go. And they went."
  - `beaten`: `$shopkeeper` gets `scarred`; then as `paid_up`.
  - `someone_gone` (on vacant, any role): close the plot.

The bar stays neutral in all of these `[D 5.4]`: the owner never takes a side; the player only seats and pours.

### 12.7 Plain-meeting events `[GB, open 6.10.1]`

| id | Condition | Weight | Effects |
|---|---|---|---|
| `good_company` | compatible | base 10; sociability 0.2 each; compatibility 1 | Opinion both ways "Good company" +10, 14 days |
| `swap_stories` | — | base 8 | Opinion both ways "Good talk" +5, 14 days |
| `awkward_silence` | — | base 6; sociability −0.2 each | None |
| `argument` | opinion either way ≤ −10, or not compatible | base 10; boldness 0.2 each; compatibility −0.5 | Opinion both ways "Argued" −10, 14 days |
| `fight` | opinion `$a → $b` ≤ −30, and `$a` has `boldness ≥ 20` or got `weld` | base 5; `$a` boldness 0.3, `$a` rationality −0.3 | Reputation −5; opinion both ways "Fought me" −20, 30 days; 30% chance `$b` gets `scarred` |

Rumours always pass both ways in any meeting (section 6.6). The owner cannot stop a fight `[D 4.11]`.

### 12.8 Delayed events

| id | From | Condition | Effects |
|---|---|---|---|
| `theft_discovered` | Plot D | `$mark` and `$thief` on the station | Opinion `$mark → $thief` "Robbed me" −40; rumour seeded by `$mark`: "{mark} knows who emptied Bay 7." |
| `they_come_back` | Plot E | `$shopkeeper` on the station and holds no role | Starts plot E again with the same shopkeeper, ignoring the cooldown |

### 12.9 Names

`names.json` holds about 40 first names and 30 family names with a mixed, spacefaring feel `[GB]`, for example: Ines Okafor, Tomas Vey, Rook Halloran, Mira Sato, Dov Petrenko, Ama Lindqvist, Kasimir Oduya, Lena Brask, Jory Castellane, Wen Achterberg. No duplicates within a run.

---

## 13. Tuning

All `[GB]` numbers live in `res://data/tuning.json`, grouped by section: `time` (hours, speeds, conversation cost, fast-forward length, catch leftover), `counter` (timer range), `tables` (meeting durations), `flow` (spawn gap, dark horses per shift, arrival windows, priority order), `needs` (default deadline, despair threshold), `plots` (plots at season start, new plots per night, active limit, owner cooldown), `checks` (roll clamp), `opinion`, `relations`, `obligations`, `reputation`, `rumours`, `signals`, `generation`, `background`, `season`.

---

## 14. Interface, controls and technical notes

Everything in this section is `[GB]` unless tagged. Block 14 ("Run and presentation") is in the next update; the greybox needs only working placeholder screens. Visuals are grey shapes and text: no art, no animation beyond simple movement, the leaving animation and bubbles.

### 14.1 Main screen: the bar `[D 9.1]`

- **Bar view.** Seen from behind the counter. The counter runs across the bottom of the screen with three seats facing the player. Behind it, the hall with three tables for two.
- **Guests.** Grey figures with a name label, a profession tag, external trait tags, an injury mark if any, and a mood icon. Counter guests also show a timer bar. A caught guest shows a highlight ring (section 3.3).
- **Drink shelf.** A row of labelled bottles along the player's side of the counter, one per drink on the menu.
- **HUD (top):** day `N / 28`, clock, "Open" or "Closed", speed buttons (Pause, 1×, 2×, 4×), reputation.
- **Snippet log (side or bottom):** every snippet with its game time `[D 3.6]`. Newest on top.
- **Popup:** on hover over any guest; pinned by right click (section 6.3).

### 14.2 Interactions

| Action | Control |
|---|---|
| Talk | Left click a counter guest (a click, not a drag) `[D 2.9]` |
| Seat at a table | Drag a counter guest onto a free table seat `[D 9.3]` |
| Pour | Drag a bottle from the shelf onto a counter guest who has not had a drink this visit `[D 10.3]` |
| Pin or unpin a popup | Right click a guest; Esc also unpins `[D 3.2]` |
| Pause | Space or the Pause button `[D 1.4.1]` |
| Catch | Left click a counter guest during the fast-forward `[D 1.4.2]` |
| Close a window | "Continue" in the conversation window, "OK" in the event window |

There are no other actions. In particular: no "refuse", no "throw out", no "send a drink from…", no moving people between tables, no dialogue choices, no "tell" (section 4.6).

### 14.3 Conversation window `[D 1.4]` `[D 2.6]` `[D 2.9]`

- Opens as a modal and pauses the game.
- Shows the speaker's name and the text, paged; the only button is "Continue". No choices.
- On close, the fast-forward of section 3.3 plays.

### 14.4 Event window `[D 3.9]`

- Opens as a modal after the leaving animation of a meeting and pauses the game.
- Shows both names, the result text, and the deciding trait as a tag ("Decided by: Greedy"); the only button is "OK".
- On close, the game resumes at the same moment: no time cost, no fast-forward.

### 14.5 Other screens

- **New game:** a seed field (empty means random) and "Start".
- **Day card:** a short "Day N" card between shifts (section 3.1).
- **Epilogue:** section 10.
- **No save or load** in the greybox.

### 14.6 Developer overlay

Toggled with F3. Not part of the design; a test tool, like CK3's debug mode for hidden values.

- Every person's base and effective personality and skills, active drink, all traits including unknown ones.
- All relations, obligations and opinions with reasons, including secret ones.
- Active plots: roles, current step, deadline, and for the last meeting: option weights, the chosen option, the check type and its success chance or threshold, the deciding trait.
- The pending arrivals list with priorities.
- Rumours each person knows. Scheduled delayed events.
- Buttons: skip 1 hour, skip to next shift, force-start a chosen plot, spawn a dark horse, reveal all facts.

### 14.7 Technical notes

- Godot 4.7.2, GDScript, 2D.
- Suggested autoloads: `Data` (loads JSON), `Clock` (time, speed, pause, fast-forward), `World` (people, traits, scales, skills, opinion, relations, obligations, reputation), `Flow` (expected and pending guests, arrivals), `StoryEngine` (plots, needs, meetings, options, checks, outcomes, delayed events, new plots between shifts), `Knowledge` (facts known to the player, rumours).
- Suggested scenes: `Main`, `BarView`, `DrinkShelf`, `GuestPopup`, `SnippetLog`, `ConversationWindow`, `EventWindow`, `DayCard`, `Epilogue`, `DebugOverlay`, `NewGame`.
- One seeded random number generator for the whole simulation, so a seed replays the same season given the same player actions.
- Simulation logic must not depend on rendering, so it can run headless.
- **Autoplay check:** a command-line mode that runs a full 28-day season with an automatic player and prints a summary. The automatic player: talks once to every new counter guest; pours a random drink for half of them; seats a counter guest with a known need at an empty table; when a table holds one guest, seats there a counter guest who fits a player-cast role of that guest's plot, if any (using true state), otherwise a random counter guest whose timer is below 30%. The summary: guests expected, arrived, never got in, left from the counter unserved; meetings by type; plots started, outcomes reached and by which path (meeting or timeout); departures, deaths; errors.

---

## 15. Next update: not in this build

The team moved all of this to the next update. Do not build any of it.

| Card | Topic | Note |
|---|---|---|
| `[D 1.10]` | Rewards for solved problems | |
| `[D 2.1.1]` | Cost of a guest who left unserved | In this build the guest just leaves. |
| `[D 2.4]` | Trust | Not clear yet whether it is needed. |
| `[D 4.2]` | Telling information ("what to tell" lever) | Feels too direct. |
| `[D 4.6]` | Mail through the bar, notes | |
| `[D 4.9]` | Giving someone a job at the bar | |
| `[D 4.12]` | Hosting an evening | |
| `[D 4.13]` | The bar as an exchange | |
| `[D 5.1]` | Bartender's code | |
| `[D 5.3]` | Reputation with different circles | One reputation in this build. |
| `[D 6.2]` | Beliefs and values | |
| `[D 6.3]` | Origin as a trait, prejudice | |
| `[D 6.5]` | Mood, stress and breakdowns | |
| `[D 6.9]` | Past and secrets | |
| `[D 6.11]` | Memory tags | |
| `[D 7.1.1]` | Where the player sees opinion, its place in meetings | Opinion is tracked but hidden in this build. |
| `[D 7.3]` | Companies | |
| `[D 8.3.1]` | Coming back to tell how a story ended | Rumours cover it in this build. |
| `[D 10.1.1]` | Drink quality, signature cocktails, a growing menu | |
| `[D 10.3.1]` | Why not pour | |
| `[D 10.3.2]` | What the line after a drink gives; a new trigger for the second floor | |
| `[D 14.1]` | Shift summary screen | |
| Block 9, rest | Zones, atmosphere, house rules, opening hours as design | View, counter, tables and closing are decided. |
| Block 11 | The station | |
| Block 12 | The owner as a character | |
| Block 13 | Bar economy: money, supplies, staff | Drinks are free in this build. |
| Block 14 | Run and presentation | Screens and epilogue are placeholders. |

Also deferred inside decided cards: more than one role per person `[D 8.6]`; passing an empty role to someone else `[D 8.6.1]`; a zoomed-out view of the station `[D 1.5]`; aging `[D 6.12]`; special drink effects `[D 10.3]`.

---

## 16. Cut: never build

| Card | What is cut |
|---|---|
| `[D 1.5.1]` | Knock on the door while the bar is closed |
| `[D 2.3]` | Crowds and a rush-hour versus quiet-hour trade-off |
| `[D 2.7]` | Owner's answers and advice; any dialogue choice |
| `[D 3.3]` | A separate relationship map screen |
| `[D 3.5]` | A witness mechanic |
| `[D 3.8]` | Informants; chatterboxes spreading rumours faster |
| `[D 4.1.1]` | Sending a drink "from" someone, and the lie coming out |
| `[D 4.3]` | Spreading a rumour on purpose |
| `[D 4.4]` | Blackmail with secrets |
| `[D 4.7]` | Alibis as a mechanic (fine as plot content) |
| `[D 4.8]` | Hiding someone or handing them to security (fine as plot content) |
| `[D 4.10]` | Kick-out, ban list, face control, refusing to pour |
| `[D 4.14]` | "Dirty tricks" as a separate lever |
| `[D 5.2]` | "What people think of the owner" as a separate value |
| `[D 5.6]` | Security interest as a parameter |
| `[D 6.6]` | Addiction |
| `[D 6.10]` | CK3-style schemes; intrigues started by people without a role |
| `[D 7.2]` | Relations appearing from opinion thresholds |
| `[D 8.4]` | Storyteller-director |
| `[D 8.7]` | Station chronicle |
| `[D 8.8]` | Memory wall, items left in the bar, toasts for the departed |
| `[D 10.4]` | Intoxication, a drink counter |
| `[D 10.5]` | Drinks on the house |

---

## 17. Open questions, placeholders and tensions

### 17.1 Open cards and their placeholders

The team has not decided these. The build uses the placeholder; keep each one isolated and data-driven.

| Card | Question | Placeholder in this build | Section |
|---|---|---|---|
| 1.4.3 | How the player sees that a guest is caught | Highlight ring | 3.3 |
| 2.2.2 | How a profession sets arrivals | Allowed days, arrival window, visit chance | 5.3, 12.3 |
| 2.2.3 | How many guests per shift, in what mix | 10–14 expected; 2 dark horses; one arrival per 15 min into a free seat | 5.7 |
| 2.2.4 | Need holder "always comes" vs. full counter | Priority by nearest deadline; no seat by closing → no visit | 5.7 |
| 3.2.1 | What the popup shows | List in 6.3; no opinions or relations | 6.3 |
| 6.8.2 | Which skills, how many | brawn, wits, charm, means, craft; 0–20 | 5.1 |
| 6.10.1 | How a meeting outside a common plot branches; can it create relations | Meeting events by weight; no new relations | 8.6, 12.7 |
| 8.1 | Where new needs come from, how soon after an outcome | 3 plots at start; up to 2 new per night, 6 active; owner cooldown 2 days | 8.3 |

### 17.2 Tensions resolved for this build

1. **Opinion is tracked but hidden.** `[D 7.1]` keeps opinion as in CK3, but where the player sees it is in the next update `[D 7.1.1]`. Opinion drives conditions and weights; only the debug overlay shows it.
2. **Second floor without trust or drink.** `[D 2.5]` opens the real secret at a special moment. Trust is in the next update and intoxication is cut, so only despair (deadline under 1 day) opens it. A new trigger is in the next update `[D 10.3.2]`.
3. **"Who they came with" without companies.** `[D 1.3]` shows it, but guests arrive one by one in this build.
4. **Hidden skills and readable checks.** Skills are hidden `[D 6.8.1]`, so a failed check could feel random. The event window names the deciding trait, and traits must hint at the skills they move (section 5.2).
5. **`fits` uses real traits.** A player-cast role accepts anyone who really fits, even if the player has not learned the trait yet.
6. **Lone table guests at closing.** `[D 9.3]` keeps them until the bar closes, and `[D 9.4]` lets the player serve the last guests; they stay until the counter is empty so a late match is possible.
7. **The owner is not a character.** Block 12 is in the next update, so the owner's death `[D 1.8]` cannot happen and debts to the owner `[D 4.5]` do not exist.
8. **Reputation effects.** `[D 3.5]` says loud events change reputation; nothing decided says what reputation does. The greybox scales regulars' visit chance.
9. **No money.** Block 13 is in the next update; drinks are free.
10. **Game language.** Not decided; greybox strings are English, all in data files.

---

## 18. Acceptance criteria

The build is done when all of these hold:

1. A new game with a fixed seed generates 14 residents with a profession, 0–1 body or wealth trait, 2 character traits, base personality and skills, family pairs, old friendships, a rivalry and 2–3 obligations; 3 plots start.
2. The bar is seen from behind the counter: 3 counter seats and 3 tables for two; there is no other seat and no door queue.
3. Guests arrive only into free counter seats, by the arrival rule of section 5.7; nobody arrives while the counter is full; pending guests without a seat by closing do not come that day.
4. Counter timers run, and a guest whose timer reaches zero leaves.
5. The player's actions are exactly: talk, pour, seat at a table, pin a popup, pause, catch. Nothing from section 4.6 exists.
6. A drink can be poured only at the counter, once per guest per visit; it shifts personality and skills until the end of that guest's meeting, visible in the developer overlay.
7. A guest at a table cannot be moved; a guest alone at a table waits until closing.
8. A conversation pauses the game and follows the ladder of section 6.4; after it, the clock fast-forwards visibly through the time cost in about 1.5 seconds.
9. During the fast-forward, the first click on a counter guest catches them and they do not leave; later clicks are ignored; Space pressed during the fast-forward pauses the game right after it.
10. Space toggles pause, and every action works while paused.
11. Hovering any guest shows the popup of section 6.3; right click pins it; unknown traits show as "?"; personality and skill values never appear outside the overlay.
12. Meetings are classified as in section 8.4; plot and need meetings always move their plot; options are picked by weight; checks follow their type; drinks visibly change weights and check chances in the overlay.
13. Every meeting ends with the leaving animation and the event window with the result and the deciding trait; that trait becomes known; the window costs no time.
14. Need deadlines run during shifts and between them; timed-out needs reach their outcome off-screen and seed a rumour.
15. Rumours pass only between people who meet, and the player learns them only in counter conversations.
16. New plots start between shifts within the limits of section 8.3; no person holds two roles; every role holder comes to the bar every shift (subject to a free seat).
17. Two dark horses come per shift; a dark horse cast into a role becomes a resident; the others never return.
18. After 04:00 nobody new arrives; the guests inside can still be served; lone table guests leave once the counter is empty and no meeting runs.
19. If a role holder dies or leaves, the plot ends with its "someone is gone" outcome; delayed events fire later and still check their conditions.
20. The season ends after day 28 and shows the epilogue.
21. The autoplay check runs a full season for 20 different seeds without errors; across those runs every sample plot reaches at least one outcome through a meeting; the same seed run twice gives identical summaries.
22. Nothing from sections 15 and 16 exists in the build.

---

## 19. What changed since v0.1

For work on the existing greybox. Section numbers refer to this document.

**Removed or replaced**

| v0.1 | v0.2 |
|---|---|
| Top-down view; door queue; counter seat + 3 tables + booth (11 seats) | View from behind the counter; 3 counter seats + 3 tables for two; no door (4.1) |
| Patience 0–100 with drains and a roll at zero | One timer per counter guest; at zero the guest leaves (4.2) |
| Only the counter guest had a card; conversations opened by themselves | A popup on every guest; the player clicks to talk to any counter guest (6.3, 6.4) |
| The clock jumped after a conversation | Visible fast-forward with a catch (3.3) |
| "Refuse" action | Removed (4.6) |
| Treat on someone's behalf; `lie_comes_out` | Removed (4.6, 16) |
| `drinks_tonight`; several drinks; 90-minute effects | One drink per visit, lasting until the end of the meeting; shifts skills too (4.4, 9) |
| Plot decisions rolled every pulse while the owner sat in the bar | Resolved at a table meeting: option by personality, then a skill check (8.4, 8.5) |
| Requests checked every pulse | Needs with a deadline, solved by meetings (8.2) |
| Pulse in the bar starting plots and generic events (quarrel, fight, rowdy, knock) | Plots start between shifts (8.3); quarrels and fights are plain-meeting events (8.6, 12.7); no knock |
| Rumours spread every pulse between people sitting together and were learned on the spot | Rumours pass only in meetings and are learned only in counter talk (6.6) |
| Occasions raising visit chance; return-to-tell visits | Every role holder comes every shift; return-to-tell is in the next update (5.7, 15) |
| Chatterbox spreads rumours faster | Cut (16) |
| Opinion with reasons in the guest card | Tracked, hidden from the player (7.1) |

**Added**

- Skills, external traits, and drinks that shift skills (5.1, 5.2, 9.2).
- Pause and act; Space remembered during the fast-forward (3.2).
- The catch (3.3).
- The event window with the deciding trait (6.5).
- The guest flow: expected and pending guests, arrival priority, dark horses (5.7).
- Player-cast roles (`fits`), meetings with an actor and options, and `outcome` / `step` / `stay` targets in the plot format (8.1, 11.4).
- Plain-meeting events (8.6, 12.7).
- Serving the last guests after closing (4.5).

**Kept**

Personality scales; opinion with reasons; relations; obligations; one reputation; the knowledge model; behaviour signals; delayed events; empty roles; background departures; season and epilogue; one seeded RNG; headless simulation; the developer overlay; the autoplay check.
