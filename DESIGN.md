# LoreVillage — Design

> Status: design complete (2026-11-17), not yet implemented.
> Owner: Sam (xiaoproton). Design review: Emma Nanob.

## LoreVillage Project

### Overall
LoreVillage refers to Lore (story, knowledge) and Village (a small place where people live, talk and carry news between each other). Its vision is to add Smallville, AI Town and AI MUD elements onto Character.ai and SillyTavern.
Users can chat with different characters in a defined world, in different places/rooms. It is not like Character.ai and SillyTavern, which basically let users chat with one person, or with multiple people presented as a single side. It does not have tile-based movement or animation, nor accurate HP/MP.

### Key Data Structure

#### World Settings
 - Name
 - Natural and Governing Laws
 - Description

#### Rooms
 - Unique Name
 - Region
 - Environment
 - Detailed Description
 - Exits
 - Picture
 - Event Artefacts

#### Narrator
 - Tone
 - User-absent History Writing Style

#### History
 - Latest X Raw Record
  * Event
  * Timestamp
 - Older Record Summary

#### Time
 - Year Season Day Hour
 - Elapse rule (example: one user "say/think" per 0.1 hour, one user "do" per 0.5 hour, one user "move" per 1 hour)

#### Characters
 - Unique Name
 - Relationship (multiple)
 - Public Profile (Race, position, basic)
 - Personalities
 - Memory
 - Avatar
 - Home Room
 - Current Room
 - Current Status (physical, health, mood...)

#### Memory
 - Latest X Raw Record
  * Event
  * Timestamp
 - Older Record Summary

#### User's Role
 - Unique Name
 - Relationship (multiple)
 - Public Profile (Race, position, basic)
 - Personalities
 - Memory
 - Avatar
 - Home Room
 - Current Room
 - Current Status (physical, health)

### Key AI LLM Prompt/Input/Response

#### Narrator (1st round) Prompt
 - Analyse the user's input to classify what the user says, does, thinks and moves
 - Update the user's current Room and current Status
 - Update Time based on the Elapse rule
 - Decide which characters receive which information
 - Decide character triggering order

#### Narrator (1st round) Input
 - All Rooms' Unique Name, Region, Environment, Exits
 - World Settings
 - History
 - All Characters' Unique Name, Race, Public Profile, Home Room, Current Room and Relationship (no Character memory)
 - Full User Role info
 - Current Time
 - User Input

#### Narrator (1st round) Response
 - Input for Characters
  * see (including what already-responded characters in the same room just did)
  * hear (including what already-responded characters in the same room just said)
  * touch/hit
  * smell
  * other sense
  * schedule reminder
 - Decide whether/what to add to the History for the time/room the user has been absent from

#### Character Prompt
 - Decide what to say, do and think
 - Be mindful of the latest event in Memory to judge how much time has passed

#### Character Input
 - Current Room
 - Character full data including Memory
 - Time
 - Narrator (1st round) Input (see/hear/feel/sense/touch), which the character may or may not have known/experienced in Memory

#### Character Response
 - What to Do
 - What to Say (displayed/visible without the Narrator)
 - What to Think
 - Whether/what to recall from a relevant part of their Memory

#### Narrator (2nd round) Prompt
 - (after triggering Character AI LLM calls in the decided sequence)
 - Decide what to tell the user based on the characters' do/think
 - Do not directly expose characters' thinking to the user

#### Narrator (2nd round) Input
 - All Rooms' Unique Name, Region, Environment, Exits
 - World Settings
 - History
 - All Characters' Unique Name, Public Profile, Home Room and Relationship, as well as Current Room and Current Status
 - Current Time
 - Characters' responses (do/say/think/recall)

#### Narrator (2nd round) Output
 - Potential World History change
 - Update Character's Current Room, Current Status
 - Text to show the user
 - Trigger World History and all Character Memory consolidation regularly

### Key Features
 - Chat page showing current Room picture
 - Chat page showing current Room exits
 - Chat page showing Characters avatar list
 - Chat page showing what characters say on the left side with avatars
 - Chat page showing narrator's words in the middle (not left, not right)
 - User types text input; result shown on the chat page's right side
 - Setting page uses one big text block to set everything for now (calling AI to understand the text and write into the data structure)

---

## Configuration

All model access is configured, never hard-coded. The engine must not assume any particular provider.

| Setting | Purpose |
|---|---|
| `OPENAI_API_URL` | Base URL of an OpenAI-compatible chat-completions endpoint |
| `OPENAI_API_KEY` | API key — **environment variable only, never committed** |
| `NARRATOR_MODEL` | Model used for both Narrator rounds |
| `CHARACTER_MODEL` | Model used for character calls |
| `IMAGE_API_URL` | Base URL for image generation (room pictures, avatars) — optional |
| `IMAGE_API_KEY` | Image API key — environment variable only |
| `IMAGE_MODEL` | Image model name |

**Defaults for Sam's own setup:** OpenRouter (`https://openrouter.ai/api/v1`) with `deepseek/deepseek-v4.1-flash` for both Narrator and Character. Not fixed — the Narrator and the characters can run on different models, and the Narrator is the one worth spending more on.

**Why separate:** anyone downloading this may point it at their own endpoint — a local llama.cpp / Ollama / LM Studio / vLLM server, another provider, or a private gateway. Nothing in the engine should assume OpenRouter.

**Security:** keys are read from the environment only. `.env` is gitignored; no key ever appears in the repo, in committed config, or in logs. `.env.example` carries placeholders only.

---

## Open Questions (not yet decided)

1. **State updates are still prose.** `Update Character's Current Room, Current Status` is a natural-language instruction. For code to apply it reliably it needs a structured schema, e.g.
   `state_deltas: [{who, field, value}]`, validated against the room graph before landing.
   This is the one place where code must adjudicate rather than the model.

2. **Off-screen events have no `witnesses` list.** The Narrator may write events into History for time/rooms the user was absent from, but characters who were present at the time do not receive them into their Memory. Needs a `witnesses` field routed into those characters' Memory.

3. **`Relationship (multiple)` is free text.** Model interpretation will drift between turns. Minimum viable structure: `{target, type, note}`.

4. **Memory is a context window, not a store.** `Latest X Raw + Older Summary` means a character can only "recall" what is already in the prompt. True recall needs a full memory store plus retrieval (embeddings) keyed on the current sensory input. Retrieval should be mechanical, not a character decision.

5. **Setting-page parse has no preview/confirm step.** AI parsing of a long text block will drop things; the structured result should be shown for confirmation before it lands.

6. **No player inventory.** `Event Artefacts` on Rooms is undefined; where a picked-up object lives is unspecified.

7. **Contradiction rule.** If a character claims a location that contradicts `Current Room`, state wins and the Narrator reconciles in narration. Needs to be written down.

## Build Notes

- Turn cost: 2 Narrator calls + N Character calls. With deepseek-v4.1-flash roughly $0.001–0.003 per turn.
- Latency: two sequential barriers; characters are a real chain (later characters see earlier ones' do/say), so ~6–9 s for 3 characters. Mitigation: render each character's speech as it arrives; stream the Narrator's narration last.
- Suggested first slice: one room, one character, one Narrator, one turn loop (CLI). Then add a second room to prove "leave and come back, the world remembers".
