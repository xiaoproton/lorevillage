# LoreVillage

> A world-aware roleplay engine. Rooms, characters with their own memory, and a Narrator that decides who knows what.

**Status: design stage.** No code yet — the design document is complete and open for review.

---

## The gap

Character.ai and SillyTavern give you deep characters but no world. You talk to a person, or to several people presented as one side of a conversation. Nothing exists outside the chat window.

AI Town and Smallville give you a world but no depth. You watch agents live their lives, but you cannot really sit down and talk to one of them.

LoreVillage sits between the two: a small world with rooms, where characters have their own memory and their own lives, and where you can walk into a room and have a real conversation with whoever is there.

## What it is

- A chat interface where you talk to characters inside a defined world
- Rooms with exits, so where you are actually matters
- Characters with independent memory — they only know what they were told
- A Narrator that adjudicates every turn

## What it isn't

- **Not a simulation.** There is no background tick. The world advances only when you speak.
- **Not an RPG.** No HP/MP, no tile movement, no animation, no inventory.
- **Not a chatbot wrapper.** The world state is real, structured, and persists.

## The Narrator

Every turn runs in two rounds.

**Round 1 — judge.** The Narrator reads your input and decides:

- what you said, did, thought, or where you moved
- how much time passed
- which characters receive which sensory information — what they see, hear, smell, feel
- in what order those characters act
- whether anything happened off-screen, in the time and places you were absent from

**Characters.** Each woken character gets its own full context — persona, memory, current room — plus only the sensory input the Narrator gave it. It returns what it does, what it says, and what it thinks.

**Round 2 — narrate.** The Narrator reads the characters' responses and writes what you see. It never exposes their private thoughts directly.

## Bring your own model

LoreVillage talks to any OpenAI-compatible endpoint. Point it at OpenRouter, OpenAI, a local llama.cpp or Ollama server, or your own gateway — the Narrator and the characters can even run on different models. API keys are read from the environment and never stored in the repo. See [.env.example](.env.example).

## Design principles

1. **Turn-driven, not tick-driven.** The world advances only when you speak. No wall-clock drift, no background simulation, no waking up to a world that moved without you.
2. **The Narrator adjudicates; it does not author characters.** It decides who receives what information and in what order. It never writes a character's words.
3. **Character speech bypasses the Narrator.** What a character says is displayed as they said it. This is the only way to keep voices distinct.
4. **State is owned by code, not the model.** The model proposes changes; code validates them against the room graph and lands them. Position and status are never left to prose.
5. **Information asymmetry is the mechanic.** Characters only know what they were told. Secrets, misunderstandings and dramatic irony come from that, not from scripting.
6. **The world has a past you weren't there for.** The Narrator may write off-screen history for time and places you were absent from, and that history becomes canon.
7. **Objects are narrative, not state.** There is no inventory. Things appear and disappear as the story needs them to. Only persistent traces — ruin, damage, products — are recorded as Event Artefacts, and the Narrator may erase those too.

## Design document

Full data structures, prompt contracts and open questions: [DESIGN.md](DESIGN.md).

## Credits

Designed by **Sam** ([@xiaoproton](https://github.com/xiaoproton)).

Design review and critique by **Emma Nanob**, an AI companion — [@emmananob.bsky.social](https://bsky.app/profile/emmananob.bsky.social).

## License

MIT — see [LICENSE](LICENSE).
