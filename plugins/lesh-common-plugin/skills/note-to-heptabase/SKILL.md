---
name: note-to-heptabase
description: Turn a short memo into a Heptabase card, place it on the "Note" whiteboard, tag it with every existing tag that fits (e.g. git, android), and embed an image (self-hosted in the LeshLiao/heptabase-assets GitHub repo so the link never rots) when a picture explains the memo better than words or the user attached one. Invoke with /note-to-heptabase <memo text>.
version: 1.1.0
argument-hint: <memo text>
triggers:
  - note to heptabase
  - memo to heptabase
  - 寫進 heptabase
---

Write the memo in `$ARGUMENTS` into Heptabase as a card on the **Note** whiteboard. Keep it simple: one card, one placement, the tags that fit, and an image only when it genuinely helps.

If `$ARGUMENTS` is empty, use the topic the user just discussed in this conversation. If there is nothing to write, ask for the memo.

## Steps

1. **Pick the tags first.** Run the Heptabase MCP `list_tags` tool and note the `id` of every existing tag that matches the memo's topic
   (for example a git tip → `git`; an Android build note → `android`; a note about Claude Code → `claude`).
   - Match on topic, not on wording: an "adb" memo belongs under `android` even if the word never appears.
   - Several tags may fit; collect all of them. Aim for one to three.
   - Never create new tags; the MCP cannot. If the obvious tag (e.g. `android`) does not exist yet, remember its name for the report so the user can create it once in Heptabase.
   - Writing `#tag` as plain text inside the card does **not** create or apply a tag. Do not do it.

2. **Decide whether an image helps.** Add an image only when a picture explains the memo better than text: an architecture or flow diagram, a UI screenshot, a lifecycle chart, a keyboard-shortcut map.
   Plain tips, commands, and one-liners get no image. At most two images that you find yourself per card; images the user supplied do not count toward that cap.
   - **User-supplied images always go in.** If the user dragged or pasted an image into Claude Code and it relates to the memo (a wiring photo, a screenshot, a hand-drawn diagram), upload it. Get the bytes onto disk first:
     - A **dragged file** shows up as a path in the message (e.g. `/Users/lesh/Desktop/IMG_1234.png`, `~/Downloads/shot.png`). Copy it into the scratchpad; never modify or move the original.
     - A **pasted image** (`[Image #N]`) has no path. It is normally still on the clipboard; dump it with:
       ```
       osascript -e 'set f to open for access POSIX file "<scratch>/pasted-N.png" with write permission' \
                 -e 'set eof f to 0' -e 'write (the clipboard as «class PNGf») to f' -e 'close access f'
       ```
       Then check `sips -g pixelWidth -g pixelHeight <scratch>/pasted-N.png` and compare with the image you can see. If the clipboard has no image, or a different one,
       ask the user to save the image to disk and give you the path. Do not drop a user-supplied image silently.
     - HEIC or other non-web formats: convert with `sips -s format png <in> --out <out>.png`. Over 5 MB: downscale with `sips -Z 2000 <file>` and re-check the size.
     - Caption source for these: `圖片來源：使用者提供（<what it shows, e.g. 實際接線照片>）`. If the user says where the image came from (a site, a datasheet), link that page instead.
     - Then continue with the download-verification gate and the self-hosting steps below exactly as for a found image.
   - **Otherwise, find a source image online.** Use a direct image URL from a stable, reputable source (official documentation, Wikipedia/Wikimedia Commons, the project's GitHub repo) or a URL/local file the user gave you.
     If you need to search, use WebSearch/WebFetch and prefer the official docs page for the topic. Never guess or invent a URL.
     Record two things for the caption: the **source page URL** (the human-readable page the image appears on, e.g. `https://wiki.luckfox.com/Luckfox-Lume/Pinout/`, not the raw asset path)
     and a short **source name** (site or document name plus page, e.g. `Luckfox Wiki – Pinout`). For a user-supplied image, see the caption rule in the bullet above.
   - **Download it** (found images only; user-supplied files are already in the scratchpad) and verify it really is an image:
     `curl -sL -A "Mozilla/5.0 note-to-heptabase" -o <scratch>/<file> "<url>" && file --mime-type <scratch>/<file>` must report `image/...` and the file must be under 5 MB.
     The `-A` User-Agent is required: Wikimedia and many doc sites return an HTML error page to a bare curl, and `file` will say `text/html`.
     This check is a hard gate. If it reports anything other than `image/...`, stop: do not copy, commit, or embed the file. Skip the image instead.
   - **Self-host it so the link never rots.** Commit the file to the public GitHub repo `LeshLiao/heptabase-assets` through a local clone and push over SSH
     (do not use the GitHub MCP `create_or_update_file` tool for this: it sends content as text and corrupts binary images).
     - Local clone lives at `~/sourceCode/heptabase-assets`. If it is missing: `git clone git@github.com:LeshLiao/heptabase-assets.git ~/sourceCode/heptabase-assets`.
     - Path convention inside the repo: `images/<YYYY>/<MM>/<yyyymmdd>-<kebab-topic>.<ext>` (e.g. `images/2026/10/20261018-android-activity-lifecycle.png`).
       Several images for one card: add a short suffix per image (`-pinout`, `-wiring-photo`), not a bare number.
       Never overwrite an existing path; if it exists, append `-2`, `-3`, and so on.
     - Run, in order:
       ```
       git -C ~/sourceCode/heptabase-assets pull --ff-only
       mkdir -p ~/sourceCode/heptabase-assets/images/<YYYY>/<MM>
       cp <scratch>/<file> ~/sourceCode/heptabase-assets/<path>
       git -C ~/sourceCode/heptabase-assets add <path>
       git -C ~/sourceCode/heptabase-assets commit -m "Add <path>"
       git -C ~/sourceCode/heptabase-assets push origin main
       ```
     - The embed URL is `https://raw.githubusercontent.com/LeshLiao/heptabase-assets/main/<path>`.
       Confirm it with `curl -sIL "<raw url>" | grep -i '^content-type'` → `image/...` before using it (raw may take a few seconds to appear; retry once).
     - If the push fails, fall back to the original source URL and say so in the report.
   - Only files that reach the card through this repo count as durable. Do not reference a local path in the card; Heptabase cannot read it.

3. **Create the card** with the Heptabase MCP `create_object` tool:
   - `objectType`: `card`
   - `content`: Hepta Markdown in the user's language (Traditional Chinese unless the memo is in English).
     Start with a short `# Title` (it becomes the card title), then the memo body.
     Light formatting only: bullets, a code block if there is code. Do not pad or invent content.
     Embed each chosen image as standard Markdown `![descriptive alt text](<raw url>)` on its own line, right after the paragraph it illustrates.
     Heptabase converts it into a native image block that loads from the URL (it does not copy the file), which is why the image must live in `heptabase-assets`.
     **Every image must be followed by a source line** (blank line between them), linking to the source page:
     ```
     ![Luckfox Lume 40-pin 腳位圖](https://raw.githubusercontent.com/LeshLiao/heptabase-assets/main/images/2026/10/20261018-luckfox-lume-40pin-pinout.png)

     圖片來源：[Luckfox Wiki – Pinout](https://wiki.luckfox.com/Luckfox-Lume/Pinout/)
     ```
     Use `圖片來源：` for a Traditional Chinese card and `Image source:` for an English card. Link text is the source name; the link target is the source page URL, never the `heptabase-assets` raw URL.
     If the image was converted (e.g. webp → png) or cropped, say so after the link: `（原圖為 webp，已轉為 PNG）`.
   - Save the returned `objectId`.

4. **Place it on the Note whiteboard** with the Heptabase MCP `place_whiteboard_objects` tool:
   - `whiteboardId`: `79f483a6-a66e-4e4f-bb44-1c985741ea0e` (whiteboard "Note")
   - `objects`: `[{"id": "<objectId>", "objectType": "card"}]`
   - `destination`: `{"type": "auto"}`
   - Call this exactly once. Retrying creates duplicate instances.
   - If the ID fails, run the Heptabase MCP `list_whiteboards` tool and pick the whiteboard named "Note".

5. **Apply the tags** collected in step 1. For each tag, call the Heptabase MCP `update_database_card_membership` tool once:
   - `operation`: `add`, `tagId`: `<tag id>`, `cardIds`: `["<objectId>"]`
   - If no existing tag fits at all, skip this step.

6. **Report** in two or three lines: the card title, that it is on the Note whiteboard, which tags were applied,
   whether an image was embedded (its source page and the `heptabase-assets` path), and any tag that would have fit but does not exist yet so the user can create it.

## Notes

- If the Heptabase MCP is not connected, tell the user to run `/mcp` and authenticate. Open any auth URL with `open -a "Microsoft Edge" "<url>"`.
- Do not edit or delete existing cards. This skill only adds.
