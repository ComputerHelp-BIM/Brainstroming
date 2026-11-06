# 20261106 Idea: Co-Model — several modellers edit the same model at the same time in the browser (no central server, no per-seat cloud licence): coloured cursors, soft claims on levels, per-field merge, per-person undo, live QA on the merged model, and an AI teammate that joins as one more cursor

## Entry 1

```
date-time:   2026-11-06 (scheduled run, third entry of the day)
idea name:   Co-Model — "Figma for BIM", but the model never leaves the team's devices.
             Three modellers open one link. Priya is on Level 3 beams, Imran on the core walls, Sana on the
             footings. They see each other's coloured cursors and selection outlines. Priya drags a beam; on
             Imran's screen it moves while she drags. Both change a column at once: Imran its grade, Priya its
             width. Both edits stay (different fields). Both type a different mark on the same column: the
             later one wins, the earlier one shows as a small chip "Imran had C-12; restore". Live QA keeps
             running on the merged model and says "Sana's footing move put F4 under column C-9 off-centre".
             A fourth cursor, "Claude", is asked to fix the QA errors on Level 4 and is watched doing it, one
             undoable edit at a time. End session: the host saves one .c2b, and every edit in it carries
             who made it.
potential:   High, because it removes a very ordinary daily wait: "I cannot open the file, someone else has it."
             Every BIM team that is two or more people on one structural model splits the work by hand
             (one model per level, one per discipline, worksets), then spends time merging, re-linking and
             chasing who changed what. The usual answers are paid: Revit worksharing needs Revit seats on
             each machine and, for people in different places, a cloud-hosted central model on a paid
             collaboration service. Cloud co-editing tools first need the model uploaded and an account for
             each person. Large clients and some firms will not allow that.
             "Real-time collaboration like Figma" is an ongoing theme in AEC posts and talks; so is "who holds
             my model". Co-Model answers both. (Stated as a general theme; no post counts were measured.)
             Time today (estimated from how teams work, not timed): model splitting and re-merging per
             milestone, half a day for a lead; "wait, which file is latest" and duplicated work, about 1 hour
             per person per week. Estimate with Co-Model: the merge step disappears; 1 to 3 hours a week saved
             per modeller in a team of 3 to 6, more for the lead. These are estimates to be replaced by
             measurements in the pilot (see Future update).
             Fit with cad2bim: the edit layer is already designed around small records keyed by GlobalId
             (docs/design/native-editing.md: elements.jsonl sorted by GlobalId, one transaction per edit,
             one undo step). Small keyed records plus transactions are the right shape for sharing edits.
```

### Description

**The pain.**
- Two people cannot edit one IFC. So teams cut the model into pieces (per level, per block, per discipline) or take
  turns with a file on a shared drive. Cutting costs a merge later: duplicated elements on the seams, a column
  moved in one piece and not in the other, marks that clash.
- Revit's answer, worksharing, is good but heavy: each person needs Revit, the central model must be reachable,
  elements are *borrowed* in a way people learn the hard way, and sync-with-central takes minutes on a big model.
  Working across offices usually means a paid cloud service and an account per person.
- For the small or mixed team (a lead on Revit, two modellers on cad2bim, a consultant who has only a browser)
  there is no easy "all of us in the same model, now" step.
- The cost is not only time. It is the quiet errors: a change nobody told you about, a duplicate that survived
  a merge, a QA run on a file that was already out of date.

**What exists today (to be re-verified before any "nothing like it" claim).**
- In this branch: Room Link (2026-11-02) is a live **review** room: shared camera, pointers, pinned comments to BCF.
  It does not let guests *change* the model. Rev Merge (2026-11-02) does a three-way merge of a new architect
  DXF into a hand-fixed model, one time, offline. Sheet Sync (2026-10-17) edits many elements from Excel, alone.
  Model Pulse (2026-10-27) records who did what from Revit activity. Change Radar (2026-10-14) diffs two revisions.
  A search of the folder for "multiplayer", "co-edit" and "CRDT" finds nothing. **No idea here is live
  co-editing of one model.**
- In cad2bim: the editing design (`docs/design/native-editing.md`), the project file (`docs/design/project-file.md`),
  versions (`docs/design/project-versioning.md`) and multi-project (`docs/design/multi-project.md`).
- In the market (not checked product by product): Revit worksharing and cloud worksharing on Autodesk's
  collaboration service (paid, per seat); Trimble Connect and Revizto (shared views and issues, paid tiers);
  Speckle (open source, a connector and a server hold the data, merges happen by versions, not live keystroke
  level); Figma and Miro (the interaction we want, not for IFC). A free, in-browser, device-to-device,
  live co-editing of IFC records with claims and per-person undo is not something I found. Unverified until
  a proper search is done.

**The idea.** A **Co-Model room**, opened from the project the way Room Link is opened. The host holds the model;
guests join by link. Everyone edits.

1. **Join in seconds.** A guest gets a snapshot (the project's records, not the IFC again) plus the edit log since
   it, over a direct device-to-device connection (WebRTC, the same plumbing as Room Link; a small signalling
   service only introduces the devices, it never sees the model). The key to the room is in the link's fragment,
   so it is never sent to a server. Late joiners and people who drop off and come back are caught up from the log.
2. **Presence you can see.** Coloured cursor with a name, coloured outline on whatever each person has selected,
   a level strip with small avatars ("Sana is on Footings"), a "who is editing this?" line in Properties.
   *Follow* a person to see what they see.
3. **Soft claims, not hard locks.** Selecting elements, or pressing *Claim level / category / area*, tints them in
   your colour. Others can still edit them, but they get a small prompt: *"Priya is working on Level 3 beams.
   Edit anyway / Ask / Skip."* The prompt is the whole point: the usual harm is surprise, not simultaneity.
   (Revit's hard borrowing stays a choice for those who want it: *Strict claims* in room settings.)
4. **Merge by field, not by element.** Each edit is an operation on a GlobalId and a field (width, grade, mark, level,
   position, delete, add). Two people editing different fields of one element both win. Same field: the later
   edit wins by a shared clock, the earlier is kept as a *conflict chip* with a one-click restore. A delete
   against an edit is shown as "deleted by X; edited by Y: keep / delete". No silent loss, nothing locks the room.
5. **Per-person undo.** Ctrl + Z undoes **your** last edit, not the last edit in the room. If someone else has
   since changed the same field, you are told what undo would overwrite and asked.
6. **Live QA on the merged model.** The existing QA checks (`packages/engine/src/qa`) run on the merged state with
   a short delay. A new issue shows who's edit introduced it ("after Sana's move at 14:32"), so a clash made by
   two correct edits is found when it happens, not at the weekly merge.
7. **An AI teammate.** The agent helper (`packages/mcp`) can join as a participant with its own cursor, named
   *Claude*. Ask in the room chat: "Fix the QA errors on Level 4." It claims what it touches, makes edits one
   transaction at a time, and every edit is in the log with its name on it and can be undone by anyone. Nothing
   it does is hidden or in bulk. **This is the part to film for LinkedIn.**
8. **End the session.** The host saves one `.c2b` and an *attribution list*: every edit with person and time.
   That list feeds Model Pulse's "who built what" and Change Radar's "what changed since Monday". The session's
   log can be replayed as a time-lapse (two minutes of a team building a floor).

**Why it works with cad2bim's own design.** Edits are small records keyed by GlobalId with one transaction per
user action. That is close to what an operation log needs. The room does not need a CRDT library for the
whole IFC; it needs ordered operations on a few record types with field-level last-writer-wins and a short list
of rules for the awkward cases (below). The IFC is written from the merged records on save, as it is today.

**Awkward cases, decided up front.**
- *Host-dependent edits* (a door in a wall that another person deletes): the delete wins, the door is shown as
  an orphan with a chip, as Revit does with hosted elements.
- *Joined and aligned elements* (a beam trimmed to a column that is then moved): the constraint is re-evaluated on
  the merged state; if it no longer holds, a QA issue is raised instead of silently stretching anything.
- *Numbering* (marks): marks created in the room are drawn from per-person blocks (Priya C-100 to C-199) so
  two people adding columns never get the same next mark.
- *Large imports* (a whole DXF → 3D run) are one named operation by one person, with a progress line for others,
  never a hundred thousand tiny edits.
- *Leaving and returning offline:* local edits are kept; on return they are merged with the Rev Merge three-way
  logic, conflicts shown as chips.

### Implementation

1. **Operation log (engine, new module `collab/`).** Edit = `{id, author, clock, gid, field, value, prev}` plus
   structural ops `add` / `delete`. A hybrid logical clock gives a total order without a server. Reducer applies
   ops to the element records; unit tests: any order of delivery gives the same state (property tests), conflict
   chips are created exactly when two different authors changed the same field within the same causal window.
2. **Transport.** Reuse Room Link's connection layer. Mesh for up to about 8 people, host as relay beyond that.
   Operations go as small messages (bytes each); snapshots are compressed records, sent in chunks.
   Room key in the URL fragment; messages encrypted with it. No model data on any server.
3. **Presence.** Cursor position in world coordinates and view, selection ids, claims; throttled to 10 to 20 Hz;
   drawn by the existing viewer as an overlay layer. Pure data, easy to turn off.
4. **UI (apps/web, design-system components, tokens only).** Room button in the ribbon, people strip, conflict chip
   in Properties and Changes panel, claim prompt, room chat, a per-room settings sheet (soft / strict claims,
   who may import, who may delete).
5. **Undo.** A per-author stack over the op log. Undo creates an inverse op with a new clock, so it travels like
   any other edit.
6. **QA.** Wire the existing checks to the merged state, debounce 500 ms, stamp each new issue with the
   operation ids that introduced it (diff the issue set before and after each batch).
7. **AI teammate.** Add a room participant type to the MCP helper: it receives presence and the log, calls the
   same edit commands as a person through the same op path; a per-room allow list for what it may do; every op
   carries the agent's name. Reuses `docs/agent/protocol.md`.
8. **Save and attribution.** Write the merged records, the IFC and a `collab/attribution.jsonl` into the `.c2b`.
9. **Build order.** (a) Op log and merge with tests, no network. (b) Two tabs on one machine over a broadcast
   channel (a demo that needs no network and proves the merge). (c) Device to device, presence, claims.
   (d) Per-person undo, conflict chips. (e) Live QA. (f) AI teammate. (g) Time-lapse replay.
10. **Honest dependency.** Value grows with what can be edited: the first release can share the edits cad2bim
    already supports (properties, marks, grades, move, copy, delete, pin, align) and QA fixes. Create tools
    (columns, beams, walls, slabs, footings drawn in cad2bim) are on the roadmap as *Planned*; each tool that
    ships becomes shareable when it is written as an operation. Co-Model does not wait for all of them.
11. **Cost and privacy.** No server holds the model. The only service is the small introduction (signalling)
    endpoint, which fits the existing Vercel functions; it sees room ids and connection offers, not content.
    A TURN relay may be needed for strict office networks; it relays encrypted bytes it cannot read.
12. **Pilot measurement.** Instrument (locally, shown to the team, not sent anywhere): number of conflicts, how
    many were resolved by chip, time between an edit and its appearance on other screens, QA issues caught live.
    Ask two teams to do a real day of work in a room and compare with their usual split-and-merge day.

### Future update

- **Revit in the room:** the Revit add-in joins as a participant through the bridge (`docs/bridge/protocol.md`),
  so a Revit user and two browser users edit the same structural model, with the add-in applying ops as one
  Revit transaction per batch. Conflicts are always decided by a person, as the sync design already says.
- **Branch and compare inside the room** (try an option on a private branch, show it to the room as a ghost, merge
  or drop), using Versions and history.
- **Voice in the room** with live captions that become BCF notes pinned where the speaker's cursor was
  (borrowing from Meeting Mind, 2026-10-26).
- **Role templates:** *Reviewer* (comments only), *Modeller*, *Checker* (QA fixes only), *Client* (view and mark up).
- **Reviewer sign-off per area:** a claim can be turned into *Ready for check*, which locks the area for edits
  and hands it to the checker; the checker's approval is stored in the attribution list (a quality trail without
  a separate tool).
- **Time-lapse export** of a session as a short video for the team, with names, for LinkedIn and Instagram.
- **Measure, then publish:** after the pilot, publish a short note with the measured hours saved per modeller,
  to replace the estimates above.
