---
name: note-to-heptabase
description: Turn a short memo into a Heptabase card and place it on the "Note" whiteboard, tagging it only when an existing tag clearly fits. Invoke with /note-to-heptabase <memo text>.
version: 1.0.0
argument-hint: <memo text>
triggers:
  - note to heptabase
  - memo to heptabase
  - 寫進 heptabase
---

Write the memo in `$ARGUMENTS` into Heptabase as a card on the **Note** whiteboard. Keep it simple: one card, one placement, at most one tag.

If `$ARGUMENTS` is empty, use the topic the user just discussed in this conversation. If there is nothing to write, ask for the memo.

## Steps

1. **Create the card** with the Heptabase MCP `create_object` tool:
   - `objectType`: `card`
   - `content`: Hepta Markdown in the user's language (Traditional Chinese unless the memo is in English).
     Start with a short `# Title` (it becomes the card title), then the memo body.
     Light formatting only: bullets, a code block if there is code. Do not pad or invent content.
   - Save the returned `objectId`.

2. **Place it on the Note whiteboard** with the Heptabase MCP `place_whiteboard_objects` tool:
   - `whiteboardId`: `79f483a6-a66e-4e4f-bb44-1c985741ea0e` (whiteboard "Note")
   - `objects`: `[{"id": "<objectId>", "objectType": "card"}]`
   - `destination`: `{"type": "auto"}`
   - Call this exactly once. Retrying creates duplicate instances.
   - If the ID fails, run the Heptabase MCP `list_whiteboards` tool and pick the whiteboard named "Note".

3. **Tag only if needed.** Run the Heptabase MCP `list_tags` tool. If an existing tag obviously matches the memo
   (for example a git tip → tag `git`), add it with the Heptabase MCP `update_database_card_membership` tool:
   - `operation`: `add`, `tagId`: `<tag id>`, `cardIds`: `["<objectId>"]`
   - Never create new tags. If no tag clearly fits, skip this step.

4. **Report** in one or two lines: the card title, that it is on the Note whiteboard, and which tag (if any) was added.

## Notes

- If the Heptabase MCP is not connected, tell the user to run `/mcp` and authenticate. Open any auth URL with `open -a "Microsoft Edge" "<url>"`.
- Do not edit or delete existing cards. This skill only adds.
