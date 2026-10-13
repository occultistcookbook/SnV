# Scum & Villainy — Game Design Document

**Version:** 0.1, greybox · **Date:** 2026-10-13 · **Engine:** Godot 4.7.2, GDScript
**Audience:** Claude Code, which builds the greybox from this document in one pass, and the design team.

---

## 0. How to read this document

This GDD describes the **greybox**: the first playable build. It contains everything the team has decided so far, plus the smallest set of assumptions needed to make the build playable.

Every rule carries a tag:

| Tag | Meaning |
|---|---|
| `[D 6.1]` | **Decided.** A design decision of the team, card 6.1 in the design zettelkasten. Implement as written. |
| `[GB]` | **Greybox assumption.** Not a design decision. A placeholder that makes the build work. Keep it in data files so the team can change it without touching code. |

Build rules:

1. Build only what is in scope (section 1.3). Do not build anything from section 15 ("Next update") or section 16 ("Cut"), not even as a stub or a disabled button.
2. Every tunable number lives in data files, not in code (section 12 and section 13).
3. All content lives in data files: traits, professions, drinks, plots, events, names, texts.
4. Where this document is silent, choose the simplest option that does not contradict a `[D]` rule, and record it in `GREYBOX_NOTES.md` at the repository root.
5. All player-facing strings live in data files. The game language is not decided yet `[GB]`; write greybox strings in English.

---

## 1. Vision

### 1.1 The game in one paragraph

A story simulator with indirect control, with Crusader Kings 3 as the main reference. The player owns a bar on a space station. Colourful residents of the station come in and bring their stories with them. The player never leaves the bar and never tells anyone what to do. The player only changes circumstances: who sits next to whom, what goes into the glass, who "sent" the drink. The people decide for themselves.

### 1.2 Pillars

1. **The owner stands behind the counter.** There is little direct control in this game, by design. `[D 1.1]`
2. **The owner changes circumstances; the person decides.** `[D 1.2]`
3. **The owner listens.** Guests choose what to talk about. There are no dialogue choices. `[D 2.6]` `[D 2.7]`
4. **Fog of war in layers.** You see the surface at once, character over time, and secrets only by luck or rumour. `[D 1.3]`
5. **Generated people in authored plots, as in Wildermyth.** Writers write plots with roles; the game casts generated people into those roles. Runs do not repeat. `[D 1.6]`
6. **The whole world comes to the bar.** `[D 1.5]`

### 1.3 Greybox scope

In scope:

- One bar on one station, one season with a fixed length and an epilogue.
- Continuous time with day and night; the bar is open, then closed.
- Generated residents with hidden personality scales and visible traits.
- Three player levers: seating, pouring, treating someone on behalf of another person; plus refusing to pour.
- Conversations at the counter that pause the game.
- Fog of war: guest card, snippets, rumours, behaviour signals.
- Opinion with reasons, relations, obligations, one shared reputation.
- Story engine: pulse with conditions, authored plots with roles, decisions inside plots, timers, resolutions, delayed events.
- Background arrivals and departures.
- A small fixed drink menu.
- Sample content (section 12) and a developer debug overlay.

Out of scope: see section 15 ("Next update") and section 16 ("Cut").

### 1.4 References

| Game | What we take |
|---|---|
| Crusader Kings 3 | Hidden personality values and visible traits, opinion with reasons, relations, decisions weighted by personality, pulse events with cooldowns, delayed events, events that pause the game. |
| Wildermyth | Generated heroes cast into authored story roles. |
| VA-11 Hall-A, Coffee Talk | A bar as the stage; regulars come back across evenings and finish their stories. |
| RimWorld | Systemic stories. Note: we do **not** take the storyteller-director (`[D 8.4]` is cut). |

---

## 2. Core loop

### 2.1 One evening

1. The bar opens. Guests arrive: regulars by their routine, anyone by an occasion. `[D 2.2]`
2. Arriving guests wait at the door. The player seats them. `[GB]`
3. The player pours drinks, reseats people, refuses someone, or sends a drink "from" somebody. `[D 1.2]` `[D 4.1]` `[D 4.10]`
4. Whoever sits at the counter talks. The game pauses while the player reads. When the talk ends, the clock jumps forward. `[D 1.4]` `[D 2.1]`
5. Snippets of other guests' talk drift to the counter on their own. `[D 3.6]`
6. Decisions inside plots ripen while their owners sit in the bar. `[D 6.10]`
7. People leave. The bar closes. Time fast-forwards until the next opening; someone may knock on the door. `[D 1.5]`

### 2.2 One season

A run lasts a known, fixed time, for example the length of the lease, and ends with an epilogue. `[D 1.7]` The greybox season lasts **28 days** `[GB]`.

There is no win or lose state in the greybox `[GB]`. The run ends at the end of the season. The owner's death would also end a run `[D 1.8]`, but the owner is not a simulated character in the greybox (block 12 is in the next update), so this never happens.

---

## 3. Time

- Time runs continuously, with day and night. The bar is open part of the day and closed the rest. `[D 1.4]`
- Conversations work like CK3 events: while a conversation is open, the game is paused. `[D 1.4]`
- A conversation costs evening time. While the player reads, the game is paused; when the conversation ends, the clock jumps forward by the scene's time cost. `[D 2.1]`
- The owner never leaves the bar. While the bar is closed, time fast-forwards, but someone may knock on the door. `[D 1.5]`

Greybox values `[GB]`:

| Parameter | Value |
|---|---|
| Opening hours | 18:00 to 04:00 |
| Clock speed at 1× | 1 real second = 1 game minute |
| Speed controls | Pause, 1×, 2×, 4× |
| Closed time | Fast-forwards at 60× until 18:00, stops for a knock |
| Conversation time cost | Per scene in data; default 30 game minutes |
| Season length | 28 days |

A possible zoomed-out camera over the station `[D 1.5]` is not part of the greybox.

---

## 4. The bar and the player's levers

### 4.1 Layout `[GB]`

Block 9 ("The bar as a place") is in the next update, but seating is a core lever, so the greybox needs a layout.

| Spot | Seats | Notes |
|---|---|---|
| Counter seat | 1 | The talk seat. Only the guest here has a guest card and talks to the owner. `[D 3.2]` |
| Table A, Table B, Table C | 2 each | |
| Booth | 4 | |
| Door | queue | Arriving guests wait here until seated. |

People at the same table, or the counter guest and nobody else, count as **sitting together**. There is no crowd and no rush hour: the bar never holds more people than seats. `[D 2.3]`

### 4.2 Seating `[D 1.2]` `[D 4.1]`

- Arriving guests wait at the door. The player drags each one to a free seat. `[GB]`
- The player may reseat anyone at any time by dragging. `[GB]`
- Waiting at the door drains patience (section 5.6). `[GB]`
- Sitting together is what moves opinion (section 7.1) and what lets decisions and events fire (sections 8.4 and 8.10).

### 4.3 Pouring `[D 1.2]` `[D 10.1]` `[D 10.2]`

- A seated guest periodically wants a drink. The guest does not name one; the player chooses `[GB]`.
- The player picks a drink from the fixed menu. There is no mixing. `[D 10.2]`
- A drink temporarily shifts the guest's hidden personality scales. Through the scales it changes the guest's decisions. `[D 6.1]` `[D 10.3]`
- Every drink served adds 1 to the guest's `drinks_tonight` counter `[GB]` (see section 9.3 on intoxication).
- Drinks are free in the greybox. Money is in the next update. `[GB]`

### 4.4 Refusing to pour `[D 4.10]`

Refusing is the only way to "throw someone out". The owner just does not pour. The guest then decides alone, through patience (section 5.6): leaves, takes offence, or starts trouble.

There is no ban list, no face control, no "kick out" button. `[D 4.10]`

### 4.5 Treating on someone's behalf `[D 4.1]`

The player can send a drink to guest B "from" guest A, whether or not that is true.

1. The player picks the recipient B, a drink, and the claimed sender A: any other guest currently in the bar.
2. B gets the drink's scale shifts as usual.
3. B gains an opinion reason toward A: "Treated me", **+15 for 14 days** `[GB]`.
4. Truth check `[GB]`: if A's opinion of B is **+20 or higher**, A plays along and the treat counts as true. Otherwise it is a lie.
5. A lie schedules a delayed event "The lie comes out" (section 8.8) **3 to 10 days** later, chance **40%** `[GB]`. When it fires: B loses the "Treated me" reason, gains "Was fooled about a drink" toward A at **−10 for 14 days**, and reputation drops by **2** `[GB]`.

A treat changes opinion but never creates an obligation. `[D 7.4]`

### 4.6 What the owner cannot do

- Ask questions, answer, advise. `[D 2.6]` `[D 2.7]`
- Tell someone a piece of information. This lever is in the next update `[D 4.2]`, even though `[D 1.2]` lists "what to tell" among the levers.
- Blackmail, spread a rumour on purpose, step into a fight, hide or hand someone over to security. `[D 4.4]` `[D 4.3]` `[D 4.11]` `[D 4.8]`
- Leave the bar. `[D 1.5]`

### 4.7 Fights `[D 4.11]`

A fight is only a consequence. The owner does not step in. A fight can be prevented beforehand by seating and drinks. When a fight happens, it changes parameters, as any loud event does: reputation, or something worse. `[D 3.5]` Fights are a generic event (section 12.7).

### 4.8 Neutrality `[D 5.4]`

The bar takes no side. Enemies may end up under one roof; tension is handled by seating, and fights happen as consequences.

---

## 5. Characters

### 5.1 Two layers, as in CK3 `[D 6.1]`

- **Hidden layer: personality scales.** Mechanics and people's decisions rely on them. The player never sees them (except in the developer overlay).
- **Visible layer: traits.** Each trait is a set of shifts on the scales. There are no pairs of opposite traits. The player sees traits through the fog of war.
- A drink temporarily shifts the scales.

Greybox scales `[GB]`: six scales, range −100 to +100.

| Scale | High means |
|---|---|
| `boldness` | Takes risks, acts |
| `compassion` | Helps, forgives |
| `greed` | Wants more, bends rules for gain |
| `honor` | Keeps word, pays debts |
| `sociability` | Seeks company, talks |
| `rationality` | Thinks before acting |

**Effective scale** = base value + sum of trait shifts + sum of active drink shifts, clamped to −100..+100 `[GB]`. Base values are random in −15..+15 `[GB]`. Every check in the game uses the effective scale.

### 5.2 Traits `[D 6.1]`

- A trait is a named set of scale shifts plus flavour text and behaviour signals (section 6.5).
- No two traits form an opposite pair. `[D 6.1]`
- Each resident gets 2 personality traits `[GB]`, one profession `[D 6.8]`, and possibly injuries `[D 6.12]`.
- The trait "Chatterbox" brings more news and spreads rumours faster. There is no separate informant role. `[D 3.8]`

### 5.3 Professions `[D 6.8]`

A profession is a visible trait. It shifts the scales and sets the routine: when the person comes to the bar. There are no skills in the greybox; skills are in the next update `[D 6.8.1]`. Professions are visible at once (fog layer 1).

### 5.4 Injuries `[D 6.12]`

An injury is a visible trait, for example "Limping" or "Scarred". Injuries come from plot resolutions and fights. They are visible at once.

### 5.5 Needs and goals

- There are no need scales. Needs show only through occasions and plots: someone broke asks for a loan, someone lonely looks for company. `[D 6.4]`
- Goals live in plot roles. A goal comes with the role: the debtor wants to pay off, the runaway wants to fly away. People without a role have no goal. `[D 6.7]`
- A person without a role does not start intrigues. `[D 6.10]`

### 5.6 Patience `[D 2.1]`

Attention is time plus guests' patience. A guest who is not served or not heard loses patience; at zero the guest leaves, takes offence, or starts trouble.

Greybox rules `[GB]`:

| Rule | Value |
|---|---|
| Patience range | 0 to 100, starts at 100 each visit |
| Waiting at the door | −2 per game minute |
| Wanting a drink and not served | −1 per game minute after 15 minutes |
| Wanting to talk and not at the counter | −0.5 per game minute after 30 minutes |
| Refused a drink | −40 at once |
| Being served | +20 |
| At zero | Roll by scales: high `boldness` and low `rationality` → rowdy (may trigger a fight event); low `sociability` → leaves quietly; otherwise → leaves offended (reputation −1) |

A guest "wants to talk" when it has something to say (section 6.6).

### 5.7 Population and generation `[D 1.6]`

- At the start of the season the game generates **14 residents** `[GB]`: name, profession, 2 personality traits, base scales.
- Old relations and family are set at generation `[D 7.2]`: **2 family pairs, 3 old friendships, 1 old rivalry** `[GB]`.
- Obligations can exist from generation `[D 7.4]`: **2 to 3** `[GB]`.
- Strangers are not part of the population. They appear only when a plot brings them for an occasion `[D 2.2]`, and leave when their part is over `[GB]`.

### 5.8 Routine and occasions `[D 2.2]` `[D 6.8]`

- Regulars come by routine; the profession sets when. An occasion changes why they came today.
- Strangers come only for an occasion.
- Occasions come from plots: a plot step, news, a request, a return to tell how it ended (section 8).
- People who are close (friend, family, lover relation) and whose routines overlap may arrive together `[GB]`. The arrival shows "came with X" `[D 1.3]`. Companies as a system are in the next update `[D 7.3]`.

### 5.9 Background arrivals and departures `[D 6.12]`

Meaningful arrivals, departures, injuries and deaths happen through plot resolutions. On top of that runs a background on a cooldown, like guests and wanderers in CK3: now and then someone without a role leaves the station, and a new generated person arrives in their place. The player notices it from a snippet or from a new face at the counter. **A person with a role never leaves this way.** `[D 6.12]`

Greybox values `[GB]`: one departure check every **4 days ±1**; the leaver is a random resident without a role; a replacement arrives **1 to 2 days** later.

Aging is in the next update. `[D 6.12]`

---

## 6. Information and fog of war

### 6.1 Three layers `[D 1.3]`

| Layer | What | How the player learns it |
|---|---|---|
| 1. Surface | Profession, injuries, mood by the face, who the person came with | Visible at once |
| 2. Character | Personality traits | Conversations at the counter and behaviour |
| 3. Secrets and plans | Plot roles, secret relations, a plot's "second floor" | Rumours or chance. Trust is in the next update `[D 2.4]`. |

Names are visible at once `[GB]`.

**Mood by the face** `[GB]`: stress is in the next update `[D 6.5]`, so the greybox mood icon is derived from simple state: patience below 30 → irritated; an active plot step with a deadline within 2 days → worried; otherwise → neutral.

### 6.2 Knowledge model `[GB]`

The game tracks which facts the player knows. A fact is one of: a person has trait X; A and B have relation R; A owes B; A holds role R in plot P; plot P ended with outcome O. The guest card and the snippet log show only known facts, except opinion (section 6.3).

### 6.3 Guest card `[D 3.2]` `[D 3.3]` `[D 7.1]`

- Only the guest at the counter has a card. The card is fairly detailed.
- There is no separate relationship map. Relations show in the counter guest's card: all those the player already knows. Secret relations open later. `[D 3.3]`
- Opinions show **with the number and the reasons**, as in CK3. `[D 7.1]` (See section 17 on the tension with secrets.)

Card contents:

- Name, profession, visible injuries, known personality traits (unknown slots shown as "?").
- Known relations and obligations.
- Opinions of people currently in the bar and of people with a known relation: number plus list of reasons with values and remaining time.
- The active request, if any, and its time left.
- `drinks_tonight`.

### 6.4 Snippets `[D 3.6]`

Fragments of guests' talk reach the counter on their own; the player does nothing to get them. **The player learns most consequences from snippets.**

- Snippets come from plot steps, resolutions, rumours passing between guests, and background departures.
- A snippet shows as a short speech bubble above the speaker and is added to the **snippet log** with the game time `[GB]`.
- A snippet may carry a fact; when it appears, the player learns that fact.

### 6.5 Behaviour signals `[D 3.7]`

Signals are behaviour only, with no labels: a person fidgets, glances at the door, orders faster than usual. Signals are a hint, not the main source of information. Learning them from run to run gives a bit of knowledge-based play.

Greybox implementation `[GB]`: every trait and every plot step can list signals. While the person is in the bar, every **20 to 40 game minutes** one of their signals plays as a short unlabeled emote line in the bubble (for example "*drums fingers on the table*"). A signal never adds a fact.

### 6.6 Rumours `[D 3.4]` `[D 3.8]`

- A rumour is true but incomplete: it carries a real fact, just a piece of it.
- It spreads from person to person by itself and never gets distorted.
- Chatterboxes bring more news and spread rumours faster. `[D 3.8]`

Greybox rules `[GB]`:

- A rumour is created by a plot resolution or a rumour-tagged plot step. It names one fact and one partial text ("Heard someone left Dock 3 in a hurry").
- Each pulse, every pair sitting together passes each rumour one knows to the other with **20%** chance, **50%** if the teller is a Chatterbox.
- When a rumour passes between two guests in the bar, it appears as a snippet and the player learns its fact.
- A guest at the counter who knows rumours the player does not may tell one (section 6.7). A Chatterbox tells up to two per conversation.

### 6.7 What a guest talks about at the counter `[D 2.6]`

The guest decides what to talk about; the owner listens. Priority `[GB]`:

1. A **return-to-tell** scene: how a story ended (section 8.7).
2. **News of the guest's plot step** (section 8.3).
3. A **request** (section 8.6).
4. A **rumour** the player does not know.
5. **Small talk** that reveals one unknown personality trait of the speaker.

Every conversation of type 1 to 4 also reveals one unknown trait of the speaker with **50%** chance `[GB]`, so character opens through conversations. `[D 1.3]`

### 6.8 Confession at the counter: two floors `[D 2.5]`

A guest talks about the outer trouble freely. The real secret behind it opens only at a special moment: drunkenness, despair (trust, if it ever exists, is in the next update). Not every story has a second floor.

Greybox triggers `[GB]`: a plot step's "second floor" text plays instead of the normal text when `drinks_tonight ≥ 3`, or when the plot's deadline is less than 1 day away.

---

## 7. Relationships

### 7.1 Opinion `[D 7.1]`

As in CK3: opinion is a number made of reasons, each with its own weight and duration. The player sees both the number and the reasons in the guest card.

- Opinion is directed: A's opinion of B can differ from B's opinion of A.
- Range −100..+100, clamped `[GB]`.
- A reason has an id, a text, a value and an expiry date (or none) `[GB]`. Expired reasons are removed.

Greybox reason sources `[GB]`:

| Source | Value | Duration |
|---|---|---|
| Compatibility (computed once per pair, see below) | −20..+20 | permanent |
| Shared a table this evening, compatible pair | +3 per hour together, up to +12 | 14 days |
| Shared a table this evening, incompatible pair | −3 per hour together, down to −12 | 14 days |
| Treated me (section 4.5) | +15 | 14 days |
| Relations (section 7.2) | see table there | while the relation lasts |
| Plot effects | set by the plot | set by the plot |

Compatibility `[GB]`: the average absolute difference of the six base scales plus trait shifts (drinks excluded), mapped linearly from 0 → +20 to 80 or more → −20. A pair is "compatible" when compatibility is positive.

### 7.2 Relations `[D 7.2]`

Friend, enemy, lover, mentor, family: separate relations on top of opinion, as in CK3. They add a big reason to opinion and last longer than opinion swings.

- Family and old relations are set at generation.
- **New relations come only from plot resolutions.** No relation ever appears from an opinion threshold. `[D 7.2]`
- Opinion is a condition the game uses to pick two people for roles. `[D 7.2]`
- A relation can be secret (for example lovers). A secret relation stays unknown to the player until a rumour, snippet or plot reveals it. `[D 3.3]`

Greybox values `[GB]`:

| Relation | Opinion reason, both ways |
|---|---|
| Friend | +40 |
| Enemy | −50 |
| Lover | +60 |
| Mentor / student | +30 |
| Family | +30 |
| Owes (section 7.3) | debtor → creditor −10; creditor → debtor −5 |

### 7.3 Obligations `[D 7.4]` `[D 4.5]`

- An obligation is one more relation type: "owes". Money and a favour are the same thing here.
- It appears at generation or from a plot resolution, adds a reason to opinion, and lives until the debt is closed.
- The game considers it when it casts roles and weighs decisions.
- There are no small debts: a treat changes opinion but never creates an obligation.
- The debtor's behaviour changes by character; there is no separate action. `[D 4.5]`

Debtor behaviour `[GB]` (applies to the debtor's visits while the obligation lasts):

| Debtor's effective scales | Behaviour |
|---|---|
| `honor ≥ 20` | Visits more often (+25% visit chance) and is more willing in "pay back" decisions (+30% weight) |
| `boldness ≤ −10` | Avoids the creditor: while the creditor is in the bar, the debtor's patience drains twice as fast, and seating the debtor at the creditor's table costs the debtor 10 patience |
| `greed ≥ 20` | Prefers decisions that settle the debt with a favour instead of payment (+30% weight on "favour" options) |

`[D 4.5]` speaks about debts to the owner. The owner is not a simulated character in the greybox, so debts to the owner do not exist yet; the rules above apply between residents. `[GB]`

### 7.4 Reputation `[D 3.5]` `[D 5.3]`

- One reputation for everybody. Reputation with different circles is in the next update. `[D 5.3]`
- There is no witness mechanic. A loud event in the bar just changes parameters: reputation, or something worse. `[D 3.5]`

Greybox values `[GB]`: range 0..100, start 50. Effect: regulars' visit chance is multiplied by `0.75 + reputation / 200` (0.75 at 0, 1.25 at 100). Changes: a fight −5, a guest leaving offended −1, a lie revealed −2, a return-to-tell visit +1.

### 7.5 Not in the greybox

Companies `[D 7.3]`, memory tags `[D 6.11]`, trust `[D 2.4]`, beliefs `[D 6.2]`, origin `[D 6.3]`: next update.

---

## 8. Story engine

### 8.1 Plots and roles `[D 1.6]`

Writers write plots with roles like "debtor", "jealous one", "runaway". The game hands the roles to generated people by their traits, by opinion `[D 7.2]` and by obligations `[D 7.4]`. Factions appear only as plot content: a racketeer or an official may come into the bar as part of an authored plot; there is no faction system. `[D 5.5]`

A plot (full format in section 11.4) has:

- **Roles** with casting conditions. A role may be filled by a resident or by a newly generated stranger.
- **Steps**: news, decisions, requests, timers.
- **Outcomes**: each with effects, snippets, rumours, and a return-to-tell scene.
- An **empty-role outcome** ("someone is gone"). `[D 8.6.1]`

### 8.2 The pulse `[D 8.1]`

A pulse with conditions, as in CK3. Every so often the game goes through events and plots whose conditions are met and picks among them by weight. Each has its own cooldown. Conditions are traits, opinion, relations, obligations, who is in the bar now and how much they drank.

There is no director. Pacing comes only from the pulse: frequency, weights and cooldowns. `[D 8.4]`

Greybox values `[GB]`:

| Parameter | Value |
|---|---|
| Pulse interval, bar open | 30 game minutes |
| Pulse interval, bar closed | 3 game hours |
| New plots started per pulse | at most 1 |
| New plots started per day | at most 2 |
| Active plots at once | at most 5 |
| Generic bar events per pulse | at most 1 |
| Decisions | every pulse, for every role holder in the bar with an open decision step |

On each pulse:

1. Advance timers (section 8.3) and fire due delayed events (section 8.8) into the candidate list.
2. Roll open decisions for role holders in the bar (section 8.4).
3. Check open requests (section 8.6).
4. Build candidates: plots that can be cast now and generic events whose conditions hold, excluding those on cooldown. Pick by weight within the limits above.
5. Spread rumours between people sitting together (section 6.6).

### 8.3 Threads across visits `[D 8.2]`

A guest's story stretches over several visits, and the plot moves on even between them. Off-screen it advances by timer: the debt grows, the deadline gets closer, the rival acts. The person brings the news to the bar and makes the decisions there. If the person does not come for a long time, the plot rolls to its resolution on its own, like a request. Part of the story the player learns only from snippets.

- A step can carry a timer. When the timer runs out, the step's `on_timeout` happens off-screen.
- A step marked `news` makes the role holder want to talk (section 6.7) and gives them an occasion to come to the bar `[D 2.2]`: their visit chance for the next day is raised to **90%** `[GB]`.

### 8.4 Decisions `[D 6.10]`

Intrigues are residents' decisions, like Decisions in CK3, not schemes. The game hands out plots and roles; decisions are steps inside a plot: the runaway is already cast, and whether he flees tonight is decided in the bar.

- A decision has conditions and a weight based on the scales.
- **A decision ripens only while the person sits in the bar.** Drinks and neighbours change the chance.
- Consequences come back as snippets.
- A person without a role does not start intrigues.

Chance per pulse `[GB]`:

```
chance = clamp(base + Σ scale_term + Σ neighbour_term + Σ obligation_term, 0, max_chance)
scale_term     = (effective_scale / 100) × factor          # from the decision's data
neighbour_term = bonus defined by the decision for someone sitting together
                 with the role holder (e.g. "the creditor is at the table": −0.10)
obligation_term = bonus defined by the decision when an obligation exists
default base = 0.05, default max_chance = 0.6
```

Effective scales include active drink shifts, so pouring the right drink changes the chance (section 4.3). A successful decision moves the plot to the next step or outcome. A decision that never succeeds stays open until the step's timer ends.

### 8.5 One role at a time `[D 8.6]`

- For the first version a person holds **one role at a time**: each person takes part in only one plot.
- A person cannot be pulled into another plot until the current one ends.
- A person with a role never leaves through the background (section 5.9). `[D 6.12]`
- Stories collide through seating and through consequences: one plot's resolution changes relations and obligations, and the pulse finds conditions for another plot in them.

### 8.6 Requests `[D 2.8]`

A request just hangs, with a timer. The player can try to resolve it, but the conditions may not come together at once. Success and expiry change the world differently.

- A request is a plot step of type `request`. The guest states it at the counter.
- It has a success condition, usually something the player can arrange with the levers, for example "the guest sits together with someone who has the trait Pilot" or "someone with `compassion ≥ 20` and opinion of the guest ≥ 20 shares a table with the guest".
- The game checks the condition every pulse. Success → `on_success`. Timer runs out → `on_timeout`.
- The active request shows in the guest card with the time left.

### 8.7 Resolutions `[D 8.3]`

Snippets plus a return. The resolution happens off-screen. If a participant is alive and still on the station, the resolution becomes their occasion to come to the bar: they come and tell how it all ended. In every other case the player learns it from snippets and rumours. So every outcome needs its own final scene.

- An outcome applies its effects (section 11.3): relations, obligations, opinion reasons, injuries, deaths, departures, reputation, delayed events, rumours.
- An outcome names its **teller** role. If the teller is alive and on the station, a return-to-tell visit is scheduled within **1 to 3 days** `[GB]` with the outcome's final scene. Otherwise, or if the teller would be gone, the outcome's rumour spreads instead.
- After the outcome, all roles are freed.

### 8.8 Delayed events `[D 8.5]`

As in CK3, an outcome can schedule an event for later: "in two weeks they will come for him", "the lie about the drink will come out". When the time comes, the event enters the pulse as a candidate and still checks its conditions. The owner gives no advice `[D 2.7]`, so consequences come from resolutions and from residents' actions.

- A scheduled event stores its bindings (which people it is about), its earliest day and a latest day `[GB]`. If its conditions never hold before the latest day, it is dropped `[GB]`.

### 8.9 Empty role `[D 8.6.1]`

For the first version: if a participant drops out, the plot ends with a special outcome. Every plot has a "someone is gone" ending: the creditor died, so the debt is written off; the lover died, so there is grief. It is an ordinary resolution under section 8.7, and one of the remaining participants comes to tell. A person can drop out suddenly, for example when a delayed event from an old plot catches up with them while they are in a new one. Passing a role to another person may come later.

### 8.10 Generic bar events `[D 8.1]` `[D 4.11]`

Besides plots, the pulse picks short generic events from the state of the bar: a quarrel between people with low opinion at one table, a fight, a knock on the door while the bar is closed `[D 1.5]`. See section 12.7.

---

## 9. Drinks

### 9.1 Menu `[D 10.1]`

For the first version: a small fixed menu, six to eight drinks, each with its own scale shifts, all available from the first evening. Price and quality belong to money and wait for block 13 (next update).

### 9.2 No mixing `[D 10.2]` `[D 10.3]`

A drink is simply picked from the menu. A drink temporarily shifts the scales, and through them people's decisions.

### 9.3 Intoxication: placeholder only

Intoxication is in the next update `[D 10.4]`, but other decided rules depend on "how much someone drank": the second floor of a confession `[D 2.5]`, decision weights `[D 6.10]`, pulse conditions `[D 8.1]`. The greybox therefore keeps only a counter `[GB]`:

- `drinks_tonight` counts drinks served to a guest during the current visit and resets when the guest leaves.
- It can be used in conditions. It has no other effect: no slurred talk, no extra scale shifts beyond each drink's own.

### 9.4 Drink effect duration `[GB]`

A drink's shifts last **90 game minutes** from serving. Shifts from several drinks add up; each expires on its own.

---

## 10. Season end and epilogue

- The season ends at 04:00 on day 28 `[GB]`, when the bar closes for the last time `[D 1.7]`.
- The epilogue screen `[GB]` lists: every plot that ended this season and its outcome as the player knows it (unknown outcomes show as "You never found out how this ended"); who left or died; the relations and obligations the player knows at the end; final reputation.
- The format of the epilogue (block 14) is in the next update; this screen is a placeholder.

---

## 11. Data formats

Everything in this section is `[GB]`: a proposed format, not a design decision. Claude Code may adjust field names, but must keep content in data files.

### 11.1 Files

```
res://data/tuning.json        all [GB] numbers from sections 3–10
res://data/scales.json        scale ids and display names (debug only)
res://data/traits.json        personality traits, professions, injuries
res://data/drinks.json        the menu
res://data/names.json         name pools
res://data/plots/*.json       one file per plot
res://data/events.json        generic bar events
```

### 11.2 Conditions

A condition is a JSON object. Role names start with `$` and refer to people bound in the current plot or event.

| Condition | Meaning |
|---|---|
| `{"all": [c1, c2]}` / `{"any": [...]}` / `{"not": c}` | Logic |
| `{"trait": ["$x", "greedy"]}` | Has the trait |
| `{"scale": ["$x", "boldness", ">=", 20]}` | Effective scale, drinks included |
| `{"opinion": ["$a", "$b", ">=", 30]}` | A's opinion of B |
| `{"compatible": ["$a", "$b"]}` | Compatibility > 0 (section 7.1) |
| `{"relation": ["$a", "$b", "friend"]}` | Relation exists |
| `{"owes": ["$a", "$b"]}` | A owes B |
| `{"in_bar": "$x"}` | Currently in the bar |
| `{"together": ["$a", "$b"]}` | Same table |
| `{"together_where": ["$x", c]}` | Someone sitting together with X satisfies `c`, where `$other` is that someone |
| `{"drinks_tonight": ["$x", ">=", 3]}` | Drinks served this visit |
| `{"profession": ["$x", "pilot"]}` | Profession |
| `{"bar_open": true}` | The bar is open |
| `{"day": [">=", 5]}` | Season day |

Implicit rules for every resident role: alive, on the station, holds no other role `[D 8.6]`.

### 11.3 Effects

| Effect | Meaning |
|---|---|
| `{"opinion": {"from": "$a", "to": "$b", "id": "bailed_me_out", "text": "Bailed me out", "value": 25, "days": 30}}` | Add an opinion reason (no `days` means permanent) |
| `{"remove_opinion": {"from": "$a", "to": "$b", "id": "..."}}` | Remove a reason |
| `{"relation": {"a": "$a", "b": "$b", "type": "lover", "secret": true}}` | Add a relation |
| `{"end_relation": {"a": "$a", "b": "$b", "type": "friend"}}` | Remove a relation |
| `{"obligation": {"debtor": "$a", "creditor": "$b"}}` | Add "owes" |
| `{"clear_obligation": {"debtor": "$a", "creditor": "$b"}}` | Close it |
| `{"add_trait": {"who": "$x", "trait": "scarred"}}` | Add a trait (injuries) |
| `{"kill": "$x"}` / `{"depart": "$x"}` | Remove from the station |
| `{"reputation": -5}` | Change reputation |
| `{"schedule": {"event": "lie_comes_out", "days": [3, 10], "chance": 0.4, "bind": {"$a": "$a"}, "latest_days": 20}}` | Delayed event `[D 8.5]` |
| `{"rumour": {"seed": "$x", "text": "...", "fact": {...}}}` | Create a rumour known first by `$x` |
| `{"snippet": {"speaker": "$x", "text": "..."}}` | Show a snippet now if `$x` is in the bar, otherwise queue it for the next time `$x` is in the bar |
| `{"reveal": {...fact...}}` | The player learns a fact |
| `{"occasion": {"who": "$x", "in_days": 1}}` | `$x` gets an occasion to come to the bar on that evening `[D 2.2]`: a stranger always comes, a resident with 90% chance |

### 11.4 Plot format

```jsonc
{
  "id": "calling_in_the_debt",
  "title": "Calling in the Debt",
  "weight": 10,               // pulse weight
  "cooldown_days": 6,         // before this plot can start again
  "roles": [
    { "id": "$debtor",   "pool": "resident", "where": {...}, "weight": {"base": 1, "scales": {"greed": 0.5}} },
    { "id": "$hunter",   "pool": "stranger", "make": {"profession": "bounty_hunter", "traits": ["cynical"]} }
  ],
  "start": "s1",
  "steps": {
    "s1": {
      "on_enter": [ /* effects */ ],
      "talk":   { "speaker": "$debtor", "text": "...", "second_floor": "...", "time_cost": 30 },
      "signals": { "$debtor": ["*glances at the door*"] },
      "decisions": [ { "id": "...", "who": "$debtor", "requires": {...}, "base": 0.05,
                       "scales": {"boldness": 0.2}, "neighbours": [ {"where": {...}, "bonus": -0.1} ],
                       "max": 0.6, "goto": "outcome:fled" } ],
      "request": { "who": "$debtor", "text": "...", "find": { "bind": "$helper", "where": {...} },
                   "goto": "outcome:bailed_out" },
      "timer":   { "days": 7, "goto": "outcome:collected" }
    },
    "s2": { "check": { "if": {...}, "then": "outcome:a", "else": "outcome:b" } },
    "s3": { "roll":  [ {"weight": 60, "goto": "outcome:a"}, {"weight": 40, "goto": "outcome:b"} ] }
  },
  "outcomes": {
    "fled": { "effects": [...], "teller": "$creditor", "scene": { "text": "...", "time_cost": 30 },
              "rumour": { "seed": "$creditor", "text": "...", "fact": {...} } }
  },
  "on_vacant": { "$debtor": "outcome:debtor_gone", "$creditor": "outcome:creditor_gone" }
}
```

Rules:

- A plot can start when every role can be bound. Roles are bound in order; each later role may refer to earlier ones. Among valid candidates the game picks by the role's weight; if a later role cannot be bound, it tries the next candidate for the earlier role (simple backtracking). `[D 1.6]` `[D 7.2]` `[D 7.4]`
- A stranger role creates a new person with the given profession and traits. A stranger leaves the station after their last scene in the plot, or when the plot ends.
- `talk` is what the speaker says the next time they sit at the counter (section 6.7). It is said once per step. `talk` may also be a list, one entry per speaker. `second_floor` replaces `text` under the conditions of section 6.8.
- A step with `talk` gives the speaker an occasion to come (section 8.3).
- `decisions` are rolled every pulse while `who` is in the bar and `requires` holds (section 8.4).
- `request` is checked every pulse (section 8.6). `find.where` may use `$other` for the person sitting together with `who`; the found person is bound to `find.bind`.
- `timer` starts when the step is entered and runs whether or not anyone is in the bar (section 8.3).
- `check` and `roll` resolve instantly when the step is entered.
- Outcomes free all roles. If the teller is alive and on the station, the outcome's `scene` becomes a return-to-tell visit; otherwise its `rumour` spreads. `[D 8.3]`
- `on_vacant` names the outcome used when a role holder dies or leaves. `[D 8.6.1]`
- Text templates: `{debtor}` → the person's name, `{debtor.profession}` → profession name.

### 11.5 Generic event format

```jsonc
{
  "id": "quarrel",
  "bind": ["$a", "$b"],        // pairs or single people currently in the bar
  "where": {...},
  "weight": 10,
  "cooldown_hours": 24,        // per bound pair or person
  "effects": [...],
  "snippet": { "speaker": "$a", "text": "..." },
  "conversation": { "speaker": "$a", "text": "...", "time_cost": 20 }   // optional, opens a paused scene
}
```

---

## 12. Sample content

All content in this section is `[GB]`: written to make the greybox playable, not decided by the team. Writers will replace it.

### 12.1 Scales

`boldness`, `compassion`, `greed`, `honor`, `sociability`, `rationality` (section 5.1).

### 12.2 Personality traits

No opposite pairs `[D 6.1]`. Chatterbox follows `[D 3.8]`.

| id | Name | Shifts | Signals |
|---|---|---|---|
| `chatterbox` | Chatterbox | sociability +25 | *talks to whoever sits nearest* |
| `hot_headed` | Hot-headed | boldness +15, rationality −20 | *puts the glass down too hard* |
| `greedy` | Greedy | greed +25, compassion −5 | *counts the change twice* |
| `kind_hearted` | Kind-hearted | compassion +25 | *slides a napkin to a neighbour* |
| `proud` | Proud | honor +15, sociability −10 | *sits very straight, never looks around* |
| `reckless` | Reckless | boldness +25, rationality −10 | *rocks the chair on two legs* |
| `romantic` | Romantic | sociability +10, compassion +10, rationality −10 | *keeps glancing across the room* |
| `loyal` | Loyal | honor +25 | *keeps the seat beside them free* |
| `cynical` | Cynical | compassion −15, rationality +10 | *snorts at the next table's toast* |
| `ambitious` | Ambitious | greed +10, boldness +10, honor −5 | *taps a datapad, impatient* |

### 12.3 Professions `[D 6.8]`

Station days run 0..6; day 6 is the rest day `[GB]`.

| id | Name | Shifts | Comes | Arrives | Stays | Visit chance |
|---|---|---|---|---|---|---|
| `dock_worker` | Dock worker | boldness +10, sociability +10 | days 0–5 | 19:00–21:00 | 2–4 h | 70% |
| `pilot` | Pilot | boldness +15, rationality +5 | any day | 22:00–01:00 | 1–3 h | 40% |
| `mechanic` | Mechanic | rationality +10, sociability −5 | days 0–6 | 00:00–02:00 | 1–2 h | 60% |
| `clerk` | Station clerk | honor +10, boldness −10 | days 0–4 | 18:00–19:30 | 1–2 h | 50% |
| `medic` | Medic | compassion +15, sociability −5 | days 0–6 | 02:00–03:00 | 1 h | 50% |
| `trader` | Trader | greed +15, sociability +10 | days 1–6 | 20:00–23:00 | 2–3 h | 60% |
| `miner` | Ore miner | boldness +10, rationality −5 | 3 days on, 1 off | 18:30–20:30 | 2–3 h | 80% |

Stranger-only professions (used by plots):

| id | Name | Shifts |
|---|---|---|
| `bounty_hunter` | Bounty hunter | boldness +20, compassion −10 |
| `racketeer` | Racketeer | greed +15, boldness +15 |

### 12.4 Injuries `[D 6.12]`

| id | Name | Shifts | Visible |
|---|---|---|---|
| `limping` | Limping | boldness −10 | at once |
| `scarred` | Scarred | sociability −5, boldness +5 | at once |

### 12.5 Drinks `[D 10.1]`

Seven drinks, all available from day 1. Each lasts 90 game minutes (section 9.4).

| id | Name | Shifts | Flavour |
|---|---|---|---|
| `weld` | Weld | boldness +25, rationality −10 | Smells of ozone. Burns twice. |
| `zero_g` | Zero-G | boldness +10, sociability +10 | Light, fizzy, takes the weight off. |
| `station_water` | Station Water | rationality +15, sociability −5 | Filtered three times. Tastes of nothing, clears the head. |
| `black_hole` | Black Hole | compassion +15, boldness −5 | Dark, sweet, heavy. |
| `red_giant` | Red Giant | sociability +25, rationality −10 | The party drink. |
| `cold_vacuum` | Cold Vacuum | compassion −15, rationality +10 | Served ice-cold. Makes people sharp. |
| `oath` | Oath | honor +20, greed −10 | An old spacers' drink, poured for promises. |

### 12.6 Plots

Five plots. Plot A is given in full JSON as the reference. Plots B to E follow the same format; their structure is given compactly and Claude Code writes them out as JSON.

#### Plot A — Calling in the Debt (full)

```json
{
  "id": "calling_in_the_debt",
  "title": "Calling in the Debt",
  "weight": 10,
  "cooldown_days": 6,
  "roles": [
    { "id": "$debtor", "pool": "resident", "where": {"any": [{"trait": ["$debtor", "greedy"]}, {"trait": ["$debtor", "reckless"]}, {"scale": ["$debtor", "greed", ">=", 10]}]},
      "weight": {"base": 1, "scales": {"greed": 0.5}} },
    { "id": "$creditor", "pool": "resident", "where": {"owes": ["$debtor", "$creditor"]}, "weight": {"base": 1} }
  ],
  "start": "s1",
  "steps": {
    "s1": {
      "talk": {
        "speaker": "$debtor",
        "text": "{creditor} wants the money back. All of it. By the end of the week.",
        "second_floor": "If I can't pay, I'm gone. I already looked at the shuttle schedule. Don't tell {creditor}.",
        "time_cost": 30
      },
      "signals": { "$debtor": ["*glances at the door every time it opens*", "*orders, then forgets the drink*"] },
      "decisions": [
        { "id": "skip_station", "who": "$debtor", "base": 0.02,
          "scales": {"boldness": 0.15, "rationality": -0.10},
          "neighbours": [ {"where": {"any": [{"relation": ["$debtor", "$other", "friend"]}, {"relation": ["$debtor", "$other", "family"]}, {"relation": ["$debtor", "$other", "lover"]}]}, "bonus": -0.05} ],
          "max": 0.4, "goto": "outcome:fled" }
      ],
      "request": {
        "who": "$debtor",
        "text": "I need someone to cover me. Just this once.",
        "find": { "bind": "$helper", "where": {"all": [
          {"scale": ["$other", "compassion", ">=", 20]},
          {"opinion": ["$other", "$debtor", ">=", 20]},
          {"not": {"owes": ["$debtor", "$other"]}}
        ]}},
        "goto": "outcome:bailed_out"
      },
      "timer": { "days": 7, "goto": "outcome:collected" }
    }
  },
  "outcomes": {
    "bailed_out": {
      "effects": [
        {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
        {"obligation": {"debtor": "$debtor", "creditor": "$helper"}},
        {"opinion": {"from": "$debtor", "to": "$helper", "id": "bailed_me_out", "text": "Bailed me out", "value": 25, "days": 30}}
      ],
      "teller": "$debtor",
      "scene": { "text": "{helper} covered for me. Now I owe {helper} instead, but I can breathe.", "time_cost": 30 },
      "rumour": { "seed": "$helper", "text": "Heard someone paid off {debtor}'s debt.", "fact": {"type": "obligation", "debtor": "$debtor", "creditor": "$helper"} }
    },
    "fled": {
      "effects": [ {"depart": "$debtor"}, {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}} ],
      "teller": "$creditor",
      "scene": { "text": "{debtor} took the night shuttle. If you ever see that face again, you tell me.", "time_cost": 20 },
      "rumour": { "seed": "$creditor", "text": "A {debtor.profession} left on the night shuttle. Owed money, they say.", "fact": {"type": "outcome"} }
    },
    "collected": {
      "effects": [ {"add_trait": {"who": "$debtor", "trait": "limping"}}, {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}},
                   {"opinion": {"from": "$debtor", "to": "$creditor", "id": "had_me_beaten", "text": "Had me beaten", "value": -40, "days": 60}} ],
      "teller": "$debtor",
      "scene": { "text": "Paid. In a way. Don't ask about the leg.", "time_cost": 20 },
      "rumour": { "seed": "$creditor", "text": "{creditor} got the money back from someone. The hard way.", "fact": {"type": "outcome"} }
    },
    "debtor_gone": {
      "effects": [ {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}} ],
      "teller": "$creditor",
      "scene": { "text": "{debtor} is gone, and the money with them. I'll write it off. Pour me something.", "time_cost": 20 }
    },
    "creditor_gone": {
      "effects": [ {"clear_obligation": {"debtor": "$debtor", "creditor": "$creditor"}} ],
      "teller": "$debtor",
      "scene": { "text": "{creditor} is gone. Nobody to pay now. I don't know what I feel.", "time_cost": 20 }
    }
  },
  "on_vacant": { "$debtor": "outcome:debtor_gone", "$creditor": "outcome:creditor_gone" }
}
```

#### Plot B — Unspoken (romance)

- **Roles:**
  - `$admirer`: resident; has `romantic` or `sociability ≥ 10`. Weight: base 1, `sociability` 0.3.
  - `$beloved`: resident; `opinion($admirer → $beloved) ≥ 30`; not family with `$admirer`. Weight: base 1, +2 if compatible.
- **s1:**
  - Talk (`$admirer`): "There's someone. Every time they come in, I forget how to talk."
  - Second floor: "It's {beloved}. Don't you dare."
  - Signals for `$admirer`: *keeps glancing at {beloved}*.
  - Decision `confess` (`$admirer`): requires `together($admirer, $beloved)`. Base 0.05, `boldness` 0.25. Neighbour bonus −0.10 if someone at the table is an enemy of `$admirer`. Max 0.6. Goes to `s2`.
  - Timer: 10 days → `outcome:never_said`.
- **s2 (check):** `opinion($beloved → $admirer) ≥ 25` → `outcome:together`, otherwise → `outcome:turned_down`.
- **Outcomes:**
  - `together`: relation `lover`, secret. Opinion both ways "Said it first" / "Was told" +10 for 30 days. If `$beloved` already has a lover X, schedule the event `found_out` in 3–8 days, chance 0.6, binding `$jealous` = X, `$rival` = `$admirer`. Teller `$admirer`: "I said it. They said it back. Don't tell anyone yet." Rumour (seed `$beloved`): "Someone saw {admirer} and {beloved} leave together." Reveals the lover relation.
  - `turned_down`: opinion `$admirer → $beloved` "Turned me down" −10 for 30 days. Teller `$admirer`: "I said it. They were kind about it. That's worse."
  - `never_said`: no effects. Teller `$admirer`: "I never said it. Maybe that's better."
  - `someone_gone` (on vacant, either role): teller is the remaining person: "{the other} is gone. I never got to… never mind."

#### Plot C — Night Shuttle (runaway)

- **Roles:**
  - `$runaway`: resident; `reckless` or `boldness ≥ 20`.
  - `$hunter`: stranger, profession `bounty_hunter`, trait `cynical`.
- **s1:**
  - `on_enter`: `occasion` for `$hunter` in 1 day (stranger occasion `[D 2.2]`).
  - Talk (`$hunter`): "Looking for a {runaway.profession} named {runaway}. Old business. You seen them?"
  - Timer: 1 day → `s2`.
- **s2:**
  - Talk (`$runaway`): "Someone's asking about me. I need to get off this rock."
  - Second floor: "I took something that wasn't mine, a long time ago. They don't forget."
  - Signals for `$runaway`: *sits with a back to the wall*.
  - Decision `take_shuttle` (`$runaway`): base 0.05, `boldness` 0.2. Neighbour bonus −0.10 for a friend, family or lover at the table. Max 0.6. Goes to `outcome:fled`.
  - Decision `face_it` (`$runaway`): base 0.02, `boldness` 0.1, `honor` 0.1. Neighbour bonus +0.10 for a friend at the table. Max 0.4. Goes to `outcome:stood_ground`.
  - Timer: 5 days → `outcome:caught`.
- **Outcomes:**
  - `fled`: `$runaway` departs. Teller `$hunter`: "Slipped away. They always do, until they don't." Then the hunter leaves.
  - `stood_ground`: `$runaway` gets `scarred`; `$hunter` departs. Teller `$runaway`: "We talked. Then we didn't talk. It's settled."
  - `caught`: `$runaway` departs. No teller; rumour (seed: a random resident who knows `$runaway`): "They took {runaway} off the station last night."
  - `runaway_gone` / `hunter_gone` (on vacant): close the plot; the remaining role holder tells, if a resident.

#### Plot D — Cargo Bay 7 (theft)

- **Roles:**
  - `$thief`: resident; `greed ≥ 20` and `honor ≤ 0`.
  - `$partner`: resident; `opinion($partner → $thief) ≥ 30` or friend of `$thief`.
  - `$mark`: resident; profession `trader`.
- **s1:**
  - Talk (`$thief`): "There's a container in Bay 7 nobody's watching. Easy money."
  - Second floor: "It's {mark}'s. {mark} has it coming."
  - Decision `plan_it` (`$thief`): requires `together($thief, $partner)`. Base 0.10, `greed` 0.2, `honor` −0.15. Max 0.6. Goes to `s2`.
  - Timer: 8 days → `outcome:lost_nerve`.
- **s2:** timer 1 day → `s3`. Signals for `$thief` and `$partner`: *whisper, stop when you pass*.
- **s3 (roll):** 60 → `outcome:clean_job`, 40 → `outcome:caught_in_the_act`.
- **Outcomes:**
  - `clean_job`: relation `friend` between `$thief` and `$partner`, secret. Schedule `theft_discovered` in 5–12 days, chance 0.5, binding `$mark`, `$thief`. Teller `$partner`: "Nothing happened in Bay 7. Nothing at all. Pour."
  - `caught_in_the_act`: `$thief` gets `limping`. Opinion `$thief → $partner` "Left me behind" −20 for 30 days. Teller `$partner`: "{thief} got caught. I ran. What was I supposed to do?"
  - `lost_nerve`: opinion `$partner → $thief` "All talk" −5 for 14 days. Teller `$thief`.
  - `someone_gone` (on vacant): a remaining role holder tells.

#### Plot E — Protection (faction flavour `[D 5.5]`)

- **Roles:**
  - `$shopkeeper`: resident, profession `trader`.
  - `$racketeer`: stranger, profession `racketeer`.
- **s1:**
  - `on_enter`: `occasion` for `$racketeer` in 1 day.
  - Talk (`$racketeer`): "Nice place. Be a shame if something happened to the people who drink here." (Said if the racketeer sits at the counter; the bar itself is not the target `[D 5.4]`.)
  - Talk (`$shopkeeper`): "Some new 'friends' want a cut of everything I sell."
  - Request (`$shopkeeper`): "I need someone tough to stand next to me when they come back." The request finds `$backer`, sitting together with `$shopkeeper`, with `boldness ≥ 30` and `opinion($backer → $shopkeeper) ≥ 20`. Goes to `outcome:backed_up`.
  - Timer: 6 days → `outcome:paid_up`.
- **Outcomes:**
  - `backed_up`: relation `friend` between `$shopkeeper` and `$backer`; `$racketeer` departs. Teller `$shopkeeper`: "{backer} stood there, arms crossed. They left. For now."
  - `paid_up`: opinion `$shopkeeper → $racketeer` "Bleeding me dry" −30. Schedule `they_come_back` in 7–14 days, chance 0.5. `$racketeer` departs. Teller `$shopkeeper`: "I paid. Everybody pays."
  - `someone_gone` (on vacant): close the plot.

The bar stays neutral in all of these `[D 5.4]`: the owner never takes a side; the player only seats and pours.

### 12.7 Generic events and delayed events

| id | Who | Condition | Effects | Cooldown |
|---|---|---|---|---|
| `quarrel` | pair together | opinion either way ≤ −20 | snippet of an argument; both: "Argued" −5 for 7 days | 24 h per pair |
| `fight` | pair together | opinion either way ≤ −40 and (one has `boldness ≥ 20` or `drinks_tonight ≥ 3`) | both leave; reputation −5; 30% one gets `scarred`; both: "Fought me" −20 for 30 days. The owner cannot intervene `[D 4.11]`. | 48 h per pair |
| `rowdy` | one guest | triggered by patience reaching zero with the "rowdy" roll (section 5.6) | 50%: `fight` with the person in the bar they like least, if opinion ≤ −20; otherwise a snippet and reputation −1 | — |
| `knock` | one resident | bar closed; the resident holds a role with an untold `talk` and the step's timer ends within 1 day | opens a conversation at the door, as at the counter `[D 1.5]` | 48 h |
| `lie_comes_out` | delayed, from section 4.5 | recipient and claimed sender both still on the station | section 4.5 step 5 | — |
| `found_out` | delayed, from plot B | `$jealous` and `$rival` on the station | relation `enemy` between them; opinion −40 both ways; rumour seeded by `$jealous`: "{jealous} knows about {rival}." | — |
| `theft_discovered` | delayed, from plot D | `$mark` and `$thief` on the station | opinion `$mark → $thief` "Robbed me" −40; relation `enemy`; rumour seeded by `$mark` | — |
| `they_come_back` | delayed, from plot E | `$shopkeeper` on the station and free | starts plot E again with the same shopkeeper, ignoring the cooldown | — |

Delayed events that hit a person who now holds a role in another plot may remove that person (none of the samples kill, but `fight` can injure). If a delayed event ever kills or removes a role holder, `on_vacant` applies `[D 8.6.1]`.

### 12.8 Names

`names.json` holds about 40 first names and 30 family names with a mixed, spacefaring feel `[GB]`, for example: Ines Okafor, Tomas Vey, Rook Halloran, Mira Sato, Dov Petrenko, Ama Lindqvist, Kasimir Oduya, Lena Brask, Jory Castellane, Wen Achterberg. No duplicates within a run.

---

## 13. Tuning

All `[GB]` numbers from sections 3 to 10 live in `res://data/tuning.json`, grouped by section: `time`, `layout`, `patience`, `treat`, `opinion`, `relations`, `reputation`, `pulse`, `decisions`, `rumours`, `signals`, `drinks`, `background`, `generation`, `season`. One more flag belongs there:

- `show_secret_opinion_reasons`: `true` by default, per `[D 7.1]`. See section 17, item 1.

---

## 14. Interface, controls and technical notes

Everything in this section is `[GB]` unless tagged. Block 14 ("Run and presentation", including screens) is in the next update; the greybox needs only working placeholder screens. Visuals are grey shapes and text: no art, no animation beyond simple movement and bubbles.

### 14.1 Main screen: the bar

- **Bar view (centre):** top-down 2D. Grey rectangles for the counter, three tables, the booth and the door. Guests are circles with a name label, a profession label, a mood icon (section 6.1) and an injury mark if any.
- **Indicators over a guest:** "wants a drink" and "wants to talk" icons.
- **HUD (top):** day `N / 28`, clock, "Open" or "Closed", speed buttons (Pause, 1×, 2×, 4×), reputation.
- **Guest card (right):** the counter guest only `[D 3.2]`; contents in section 6.3. Empty when the counter seat is empty.
- **Snippet log (bottom):** every snippet with its game time `[D 3.6]`. Newest on top.

### 14.2 Interactions

| Action | Control |
|---|---|
| Seat or reseat a guest | Drag the guest from the door or from a seat to a free seat |
| Pour | Click a guest → "Pour" → pick a drink from the menu list |
| Refuse | Click a guest → "Refuse" `[D 4.10]` |
| Treat on someone's behalf | Click a guest → "Send a drink from…" → pick the claimed sender among guests in the bar → pick a drink `[D 4.1]` |
| Read a conversation | It opens by itself when a guest with something to say takes the counter seat |

There are no other actions. In particular: no "ask", no dialogue choices, no "tell", no "throw out", no fight intervention (section 4.6).

### 14.3 Conversation window `[D 1.4]` `[D 2.1]` `[D 2.6]`

- Opens as a modal and pauses the game.
- Shows the speaker's name and the text, paged; the only button is "Continue". No choices.
- On close, the clock jumps forward by the scene's `time_cost`. Everything that would have happened in that time (pulses, timers, patience) is simulated in one step.
- A knock on the door while the bar is closed opens the same window `[D 1.5]`.

### 14.4 Other screens

- **New game:** a seed field (empty means random) and "Start".
- **Epilogue:** section 10.
- **No save or load** in the greybox.

### 14.5 Developer overlay

Toggled with F3. Not part of the design; a test tool, like CK3's debug mode for hidden AI values.

- Every person's base and effective scales, active drink shifts, all traits including unknown ones.
- All relations and obligations, including secret ones.
- Active plots: roles, current step, timers, decision chances on the last pulse.
- Scheduled delayed events.
- Pulse log: candidates, weights, picks.
- Buttons: skip 1 hour, skip 1 day, force-start a chosen plot, reveal all facts.

### 14.6 Technical notes

- Godot 4.7.2, GDScript, 2D.
- Suggested autoloads: `Data` (loads JSON), `Clock` (time and speed), `World` (people, traits, scales, opinion, relations, obligations, reputation), `StoryEngine` (pulse, plots, decisions, requests, delayed events, generic events), `Knowledge` (facts known to the player, rumours).
- Suggested scenes: `Main`, `BarView`, `GuestCard`, `SnippetLog`, `ConversationWindow`, `Epilogue`, `DebugOverlay`, `NewGame`.
- One seeded random number generator for the whole simulation, so a seed replays the same season given the same player actions.
- Simulation logic must not depend on rendering, so it can run headless.
- **Autoplay check:** a command-line mode that runs a full 28-day season with an automatic player (seats arrivals at random free seats, pours a random drink when asked, never refuses) and prints a summary: plots started, outcomes reached, departures, deaths, errors. Used for the acceptance check in section 18.

---

## 15. Next update: not in the greybox

The team moved all of this to the next update, which starts after the greybox. Do not build any of it.

| Card | Topic | Note |
|---|---|---|
| `[D 2.4]` | Trust | Not clear yet whether it is needed. |
| `[D 4.2]` | Telling information ("what to tell" lever) | Feels too direct. |
| `[D 4.6]` | Mail through the bar, notes | |
| `[D 4.9]` | Giving someone a job at the bar | |
| `[D 4.12]` | Hosting an evening | |
| `[D 4.13]` | The bar as an exchange | Too complex for the base version. |
| `[D 5.1]` | Bartender's code | |
| `[D 5.3]` | Reputation with different circles | Greybox has one reputation. |
| `[D 5.4]` part | Choosing a side | Greybox bar is neutral. |
| `[D 5.5]` part | Ongoing relations with factions | Greybox has factions only inside plots. |
| `[D 6.2]` | Beliefs and values | |
| `[D 6.3]` | Origin as a trait, prejudice | |
| `[D 6.8.1]` | Skills, as in CK3 | Skills will shape how events play out and the chance that actions succeed. |
| `[D 6.5]` | Mood, stress and breakdowns | Not clear yet how to implement it. |
| `[D 6.9]` | Past and secrets | |
| `[D 6.11]` | Memory tags | Wanted for revenge and rematches; too complex for the first iteration. |
| `[D 6.12]` part | Aging | |
| `[D 7.3]` | Companies | Options are recorded on the card. |
| `[D 10.1.1]` | Drink quality, signature cocktails, a growing menu, a menu per run | |
| `[D 10.4]` | Intoxication | Greybox keeps only the `drinks_tonight` counter (section 9.3). |
| Block 9 | The bar as a place: zones, atmosphere, room chemistry, contagious mood, regulars, house rules, opening hours | Greybox layout and hours are placeholders. |
| Block 11 | The station: districts, schedule, work and shortages, factions, power and law, crime, ships, crises | |
| Block 12 | The owner: as a character, skills, own life, goals, the owner as a target, successor | |
| Block 13 | Bar economy: money, supplies and stock, staff, gambling table, side business, damage and repairs, competitors | Greybox has no money. |
| Block 14 | Run and presentation: scenarios and starts, end of run, epilogue, screens, modding | Greybox screens and epilogue are placeholders. |

Also deferred inside decided cards: more than one role per person `[D 8.6]`; passing an empty role to someone else `[D 8.6.1]`; a zoomed-out view of the station `[D 1.5]`.

---

## 16. Cut: never build

| Card | What is cut |
|---|---|
| `[D 2.3]` | Crowds and a rush-hour versus quiet-hour trade-off |
| `[D 2.7]` | Owner's answers and advice; any dialogue choice |
| `[D 3.3]` | A separate relationship map screen |
| `[D 3.5]` | A witness mechanic |
| `[D 4.3]` | Spreading a rumour on purpose |
| `[D 4.4]` | Blackmail with secrets |
| `[D 4.7]` | Alibis as a mechanic (fine as plot content) |
| `[D 4.8]` | Hiding someone or handing them to security (fine as plot content) |
| `[D 4.10]` | Kick-out, ban list, face control (only refusing to pour exists) |
| `[D 4.14]` | "Dirty tricks" as a separate lever |
| `[D 5.2]` | "What people think of the owner" as a separate value |
| `[D 5.6]` | Security interest as a parameter |
| `[D 6.6]` | Addiction |
| `[D 6.10]` | CK3-style schemes; intrigues started by people without a role |
| `[D 7.2]` | Relations appearing from opinion thresholds |
| `[D 7.4]` | Small debts from treats |
| `[D 8.4]` | Storyteller-director |
| `[D 8.7]` | Station chronicle |
| `[D 8.8]` | Memory wall, items left in the bar, toasts for the departed |
| `[D 10.5]` | Drinks on the house |

---

## 17. Known tensions and open questions

These are places where decided cards pull against each other or where the greybox had to fill a gap. Each is resolved in the greybox as stated, and should go back to the team.

1. **Opinion visibility versus secrets.** `[D 7.1]` shows opinion with all reasons, but `[D 3.3]` says secret relations open later. A secret lover adds +60, so the number gives the secret away. The team chose to show everything for now ("we can always hide it later"). The greybox follows `[D 7.1]` behind the flag `show_secret_opinion_reasons = true`.
2. **"What to tell" lever.** `[D 1.2]` lists "what to tell" among the owner's levers, but `[D 4.2]` moved it to the next update. The greybox has no "tell" action.
3. **Second floor without trust.** `[D 2.5]` opens secrets by trust, drunkenness or despair. Trust is in the next update `[D 2.4]`, so only drinks and despair open it.
4. **"How much they drank" without intoxication.** `[D 2.5]`, `[D 6.10]` and `[D 8.1]` rely on it, but intoxication is in the next update `[D 10.4]`. The greybox keeps only the `drinks_tonight` counter.
5. **The owner is not a character.** Block 12 is in the next update, so the owner's death `[D 1.8]` cannot happen and debts to the owner `[D 4.5]` do not exist. Debtor behaviour applies between residents.
6. **"Who they came with" without companies.** `[D 1.3]` shows who someone came with; companies are in the next update `[D 7.3]`. The greybox lets close people with overlapping routines arrive together.
7. **Compatibility.** `[D 6.1]` forbids opposite trait pairs, so CK3's "opposite traits dislike each other" does not port. The greybox compares scales instead (section 7.1). Not a design decision.
8. **Reputation effects.** `[D 3.5]` says loud events change reputation, but nothing decided says what reputation does. The greybox scales regulars' visit chance.
9. **Mood without stress.** `[D 1.3]` shows mood on the face; stress is in the next update `[D 6.5]`. The greybox derives mood from patience and plot deadlines.
10. **Layout, opening hours, season length, epilogue.** Blocks 9 and 14 are in the next update; the greybox values are placeholders.
11. **No money.** Block 13 is in the next update; drinks are free.
12. **Game language.** Not decided; greybox strings are English, all in data files.

---

## 18. Acceptance criteria

The greybox is done when all of these hold:

1. A new game with a fixed seed generates 14 residents with professions, two personality traits each, base scales, family pairs, old friendships, a rivalry and 2–3 obligations.
2. Residents arrive by their profession's routine; arriving guests wait at the door until the player seats them; nobody ever sits outside the 11 seats.
3. The player can seat, reseat, pour, refuse and send a drink "from" someone; there are no other actions.
4. A served drink changes the guest's effective scales for 90 game minutes, visible in the developer overlay.
5. The counter guest's card shows known traits, known relations, opinions with numbers and reasons, the active request and `drinks_tonight`; unknown traits show as "?".
6. A guest with something to say at the counter opens a conversation that pauses the game; after it, the clock jumps by the scene's time cost.
7. Snippets appear in bubbles and in the log on their own; rumours pass between people sitting together and reveal facts to the player.
8. The pulse starts plots only when all roles can be cast, respects weights, cooldowns and limits, and never gives one person two roles.
9. Plot decisions roll only while their owner is in the bar, and drinks and neighbours visibly change the chance in the developer overlay.
10. Requests resolve when the player arranges the condition by seating, and expire by their timer otherwise.
11. Timers advance while the bar is closed; plots reach outcomes without the player's presence.
12. Every outcome applies its effects; if the teller is alive and on the station, they come back within 1–3 days and tell the ending at the counter; otherwise a rumour spreads.
13. Delayed events fire later through the pulse and still check their conditions.
14. If a role holder dies or leaves, the plot ends with its "someone is gone" outcome.
15. Background departures remove only people without a role, and new people arrive in their place.
16. Fights happen only as consequences, the owner cannot stop them, and they lower reputation.
17. While the bar is closed, time fast-forwards and stops for a knock when its condition holds.
18. The season ends on day 28 and shows the epilogue.
19. The autoplay check (section 14.6) runs a full season for 20 different seeds without errors, and across those runs every sample plot reaches at least one outcome.
20. Nothing from sections 15 and 16 exists in the build.
