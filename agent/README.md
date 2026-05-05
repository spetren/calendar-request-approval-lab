# Calendar Management Agent

A Copilot Studio agent that turns natural-language calendar requests into rows in `CalendarRequests`.

## Files

- [instructions.md](instructions.md) — system instructions for the agent
- [topics.md](topics.md) — topic outlines (trigger phrases, slots, action wiring)

## Wiring

1. Create a new agent in Copilot Studio.
2. Paste the contents of `instructions.md` into the agent's *Instructions* field.
3. Add a topic per `topics.md`. Each topic ends by calling the `CreateCalendarRequest` Power Automate flow as an **Action**.
4. Map topic slots → flow inputs.
5. Test in the test pane, then publish.
