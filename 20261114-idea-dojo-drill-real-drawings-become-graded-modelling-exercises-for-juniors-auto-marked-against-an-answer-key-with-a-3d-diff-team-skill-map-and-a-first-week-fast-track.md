# 20261114 Idea: Dojo Drill — the team's own finished projects become graded practice exercises for junior modellers; the tool marks their model against the answer key, shows the difference in 3D, and builds a skill map so the lead knows who can do what

## Entry 2

```
date-time:   2026-11-14 (scheduled run, UTC)
idea name:   Dojo Drill (training-by-doing: exercises cut from your own approved models and CH drawings, auto-marked)
potential:   High for any BIM team that hires juniors or has attrition (common in Indian AEC); medium for freelancers.
             The pain is not "where is a Revit course". It is: the senior stops modelling for weeks to check a
             junior's work by eye, and the junior still repeats the same mistakes (wrong host, wrong type, missing
             parameters, loose steel, wrong level) because nobody has time to mark every model.
             Time saved: NOT measured here. Test, don't quote: time a senior's weekly review of one junior before and
             after; the claim to check is "senior review time per junior falls, and first-pass errors fall".
             Market check (2026-11-14, one search, not exhaustive): the results were courses (Novatr, ImaginIT,
             3D Training, BIMpure's "Claude for BIM" live course), and a vendor blog arguing AI coaching beats
             courses because it answers at the moment of need. None described exercises cut from the firm's OWN
             projects and auto-marked against that project's model. Not claimed as unique worldwide; the claim that
             fits is: free, in the browser, no upload, built from the models you already have.
             Not the same as other ideas here: Toolsmith (a team tool from an AI recipe), Warning Doctor (health of
             a live Revit model), Deja Model (reuse of old elements), Model Pulse (who did what, activity). Dojo Drill
             is the one that teaches and marks.
             Trend: LinkedIn and trade press this season is about AI inside Revit, MCP, and above all how to train a
             team to use AI "without breaking the model" (sources at the end). Juniors will prompt first and check
             later. A marker that says "your model differs from the answer here" is the safety net for that.
             The wow: a junior opens "Drill 07: Podium level, columns and beams from this CH drawing", models it in
             15 minutes, presses Mark. Their model glows: green where it matches, amber where it is close ("beam
             300x600, answer key 300x650"), red where it is missing or extra. A score, a list of mistakes with the
             rule behind each, and a "try again" that only re-marks what changed. The senior only sees the score.
```

### Description

**The pain.**
- Juniors learn Revit tools from courses but not the firm's way: type naming, level and host rules, shared
  parameters, IFC class mapping, the CH drawing convention. This is the part that breaks schedules and BOQs later.
- Review is manual: the senior opens the junior's model and looks. It is slow, uneven, and not recorded.
- The same mistake returns next month with the next junior.
- The lead cannot answer "who on the team can do rebar? who can do stairs?" except by memory.

**What exists (checked 2026-11-14; one search, not exhaustive).**
- Courses and live classes ([Novatr](https://www.novatr.com/blog/top-bim-modeler-skills),
  [BIMpure](https://www.bimpure.com/blog/announcement-claude-for-bim-live-course),
  [3D Training](https://3dtraining.com/corporate-aec)); general Revit lessons, not the firm's own standards.
- AI coaching pitched as faster onboarding ([coaching article](https://djinja-c.pages.dev/blog-future-bim-ai-coaching.html)),
  vendor claims, not measured here.
- cad2bim already has the pieces: an IFC reader, element ids and types, QA rules and IDS-style checks, the
  CH drawing → 3D pipeline (which gives an answer key from a drawing), the viewer's highlight, and BCF.

**The idea in one line.** Take a finished, approved piece of a real project; hide the answer; let the junior build it;
compare their model with the answer, element by element, and teach from the differences.

### How it works (user view)

1. **Make a drill (lead, once, 5 minutes).** Pick a floor, a zone or a few element types from an approved model (IFC
   or .c2b). The tool strips it to a starter pack: the CH DXF or PDF underlay, grids and levels, empty or partly
   filled. The approved model is the answer key and stays on the lead's device unless they share it.
2. **Set the marking rules.** Defaults: match by position (within a tolerance, set by the lead, e.g. 10 mm) and IFC
   class; then check type/size, level, host, material, shared parameters, mark text and (optionally) rebar. Each rule is
   a line the lead can switch on or off, weighted.
3. **The junior models.** In Revit or in cad2bim, as usual. They export an IFC (or save the .c2b) and drop it on Dojo
   Drill. Nothing is uploaded.
4. **Mark.** The tool pairs elements junior-vs-key (nearest by position and class; ties by size), then:
   - **Green**: match. **Amber**: right place, wrong size/type/level/parameter. **Red**: missing, or extra.
   - Each non-green element has a plain reason and the rule behind it ("Beams are named B-WxD; yours is 'Beam 1'").
   - A score per rule group (geometry, types, parameters, naming) and an overall score.
5. **3D diff.** Overlay: key as a faint ghost, the junior's model solid, the differences coloured. Click an amber
   beam to see both side by side.
6. **Retry loop.** The junior fixes and presses Mark again; only the changed elements are re-marked. Attempts and
   time are kept (on the device or shared, the lead's choice).
7. **Skill map.** After a set of drills the lead sees a grid: person × skill (columns, beams, slabs, stairs,
   openings, rebar, parameters, naming, drawings → 3D). Colour is the average score over the last few drills, with the
   number of attempts shown so one lucky score is not read as a skill.
8. **First-week path.** A default sequence of drills (grids and levels → columns → beams → slabs → walls and
   openings → types and parameters → export checks). The lead can reorder it, add their own, or lock the next drill
   until the previous one passes.

### Why a BIM team would use it

- Senior review time goes to the hard questions, not to "this is on the wrong level".
- The firm's way of working becomes something a junior can practise, not something they absorb over months.
- Every mistake is tied to a rule the lead wrote, so the rule is explained once, by the tool, every time.
- The lead sees who is ready for a live project, with evidence. This also helps when staffing and when quoting.
- It keeps AI-assisted juniors honest: the marking does not care how the model was made.

### Implementation

Runs in the browser on files on the device; no upload (the repo's rule). Read-only on every input.

1. **Reading.** Use the existing IFC reader for both models. For a drill made from a drawing, run the existing
   CH → 3D pipeline to produce the answer key, so any approved CH drawing is a drill source.
2. **Starter pack.** Keep grids, levels, the underlay and any elements the lead marks as "given". Remove the rest
   from the copy given to the junior. The answer key is a separate file; it is never inside the starter pack.
3. **Pairing.** Per IFC class, build a spatial index (position of placement and bounding box). Pair by nearest centre
   within tolerance; resolve ties with size and type name; leftovers are "missing" (key only) or "extra" (junior
   only). Report the pairing so a person can see it was fair; allow the lead to re-pair one element by hand.
4. **Rules.** A small declarative list, in the same style as the QA/IDS rules the app already has: the lead's
   rules are data, not code. Rule result per element: pass, near (within a second tolerance), fail.
5. **Score.** Weighted share of elements passing each rule, per group and overall. The formula is shown on screen.
   A pass mark is set by the lead per drill.
6. **Diff view.** Reuse the viewer's ghost, override-colour and highlight; reuse BCF export so a junior's mistake list
   can be handed to the senior as a normal issue list.
7. **Skill map.** A small local table: person, drill, attempt, date, score by group, time. Export to CSV. Shown only
   to people the lead chooses; see "Honest limits".
8. **Test plan.** Take one real approved floor. Have two people model it (one experienced, one new). The experienced
   one should score high, the new one lower, and every red or amber item must be a real difference a senior agrees
   with. Count false marks (marked wrong but fine, or marked right but wrong). Do not publish a score or time figure
   until this has been done on at least three floors.

### Future update

- **Hint ladder.** After a failed attempt, show a hint first ("check the level of the beams"), then the rule, then
  the answer — so the junior does the thinking.
- **AI explainer with the junior's own key.** An optional "why is this wrong?" that gets only the rule and the two
  element descriptions, never the model, and only when the person asks (the app's existing own-key chat).
- **Revit-side button.** Mark from inside Revit through the add-in, without exporting.
- **Rebar drills.** Compare cage layout, bar marks and BBS totals with the key, within a tolerance.
- **Live-project mode.** Use a senior's finished floor as the key for the next floor (typical-floor checking), so the
  same marker becomes a peer check.
- **Drill library.** Share anonymised drills between firms, with consent and a stripped answer key.
- **Certificates.** A passed path gives a PDF with the date and scores; the firm decides what it is worth.
- **Hiring test.** A timed drill for candidates, marked the same way, so interviews compare like with like.

### Reel idea for LinkedIn / Instagram

15 seconds: split screen. Left, a junior's Revit window; right, the answer key as a ghost. Press Mark: green, amber,
red, and a score ticking up through three tries. Caption: "Your own projects are your best BIM training course."
(Post only real numbers from a real team's drill.)

### Honest limits

- Marking is by position, class and data. It cannot judge a good design from a merely matching one; it teaches the
  firm's standard, not engineering judgement.
- Pairing can be wrong in dense or irregular layouts; the lead can re-pair, and the tolerance is theirs.
- A skill map of people is sensitive. It should be private to the lead by default, and the tool must say what is
  stored. It must not be used as the only basis for decisions about someone's job.
- The approved model may have its own errors; the key is only as good as that model, and the lead can fix it.
- The time-saved figure is an estimate to test. The market check was one search, mostly vendor pages, and I could not
  measure LinkedIn or Instagram directly.

### Sources (checked 2026-11-14)

- AI and training trend: [Dan Cumberland Labs, training a BIM team to prompt without breaking the model](https://dancumberlandlabs.com/blog/bim-2-0/),
  [ImaginIT, applied learning vs AI and YouTube](https://resources.imaginit.com/elevate-your-people-processes/the-myths-of-learning-ai-google-youtube-and-why-applied-learning-still-wins),
  [ImaginIT, putting AI to work in BIM](https://resources.imaginit.com/building-solutions-blog/you-dont-need-to-be-an-ai-expert-to-put-ai-to-work-in-bim),
  [BIMpure, Claude for BIM course](https://www.bimpure.com/blog/announcement-claude-for-bim-live-course).
  These are vendor and trade pages, not LinkedIn counts.
- Existing courses and skills lists: [Novatr](https://www.novatr.com/blog/most-in-demand-bim-skills-and-how-to-learn-them).
