# 20261123 Idea: Bhasha Sheet — every site-facing output (BBS, bar-bending cards, pour cards, fixing manual, toolbox talk, snag list) in the language the crew actually reads, with picture bar shapes, tap-to-listen audio and a "did the foreman understand?" check

## Entry 1

```
date-time:   2026-11-23 05:30 UTC (scheduled run)
idea name:   Bhasha Sheet (Site Documents in the Crew's Own Language, Picture-First, Read-Aloud, Offline)
potential:   High. The BIM team's work ends on a sheet in English that bar benders, carpenters, masons and
             gang leaders often cannot read comfortably. The usual fix is a verbal brief from the site
             engineer, which is where wrong cut lengths, missed laps and "we did not know" come from.
             Estimated saving (an estimate, not measured): 30 to 60 minutes per day per site engineer on
             briefing, and fewer wrong-bar and wrong-lap reworks. It is cheap to build on files cad2bim
             already makes (BBS, pour card, fixing manual), so it extends the toolkit instead of adding a
             new product. Wow factor: a bar-bending card that speaks.
```

### Description

**The pain.**
- BBS and drawings are in English with codes ("T12 @ 150 c/c, 40 cover, Ld 47d"). Gang leaders read numbers
  and shapes well, labels and notes poorly. A site engineer re-explains every pour by voice, every day.
- Existing ideas here make good site outputs: Cage Manual (fixing steps), Bend Bridge (bar bending files),
  Site QR Pass (pour card), Edge Guard (toolbox talks), Snag Sweep (trade fix lists). All of them are
  English first. The last metre to the person doing the work is not covered.
- Plain machine translation breaks construction words. Site words are not dictionary words: "shuttering"
  and "centering", "cover block", "chair", "lap", "stirrup" (ring), "binding wire", "cold joint" mean
  different things in different states, or are said in English by the crew itself.
- I saw regional-language and Hinglish content doing well on LinkedIn and Instagram (site vlogs, "civil
  engineer in Marathi/Hindi" reels). Read this as a trend I noticed in feeds, not a measured statistic.

**What exists today (one thin check, to be re-verified before any "nothing like it" claim).**
- General translators (Google Translate, Bing, etc.) translate text but know nothing about a bar shape, a
  pour or an IFC element, and mangle site words. I did not test their current output on a BBS.
- Paid construction-management apps (Fieldwire, Procore, PlanRadar and similar) have interface language
  options. I did not verify what they offer for documents, or their prices. They do not turn a BBS into a
  picture card.
- Inside this repo, nothing: the few mentions of regional language are one-line notes inside other ideas
  (Chat-to-Model, Cage Manual, Edge Guard). No idea owns it.

**The idea.** A **Bhasha** output option on every site-facing export, in the browser, on the device:
1. **Pick language and style** once per project: Hindi, Marathi, Gujarati, Tamil, Telugu, Kannada, Bengali,
   Odia, Punjabi, plus "Hinglish" (English words in Roman script, the way many sites actually talk).
2. **A site glossary, not free translation.** A small, team-owned table of construction terms per language
   (term, local word, "keep English" flag, picture). It ships with a starter list; the team's senior
   foremen fix it once and it is reused on every project. A word not in the glossary is left in English
   and listed for review. Nothing is guessed silently.
3. **Picture first.** Each bar on the card is its BBS shape drawn to scale with the cut length big, then
   one line in the local language ("12 mm, 8 numbers, 2 bends, 2.40 m"). Numbers stay in numerals the crew
   uses (Western or local digits, a project setting). Text is the second layer, not the first.
4. **Tap to listen.** On the phone version each line has a play button: a short voice line read by the
   device's own speech engine ("Beam B14, twelve millimetre, eight bars, two point four metres"). Audio is
   made on the device; no recording of anyone is sent anywhere.
5. **Understood?** A three-tap quiz at the end of a pour card (cover? lap length? which bar first?) with
   answers from the model. The engineer sees a gang's score. It is a briefing aid, not an exam, and it can
   be switched off.
6. **Round-trip safety.** Every translated sheet prints the English line in small type beside the local
   line, and a footer "translated with glossary v3, reviewed by: ___ on ___". Numbers are never
   translated; they are copied from the model and checked: the sheet's numbers must equal the English
   sheet's numbers, or the export is blocked.

**The wow 1: the card that speaks.** Scan the QR on a column (Site QR Pass), hold the phone to your ear, and
hear the fixing steps in Marathi with the picture of each bar on screen. A short video of this is shareable
content in itself (the Model Reel link).

**The wow 2: the foreman fixes the glossary, and everyone gets it.** Tap a wrong word on the sheet, type
the right one; it is saved to the team glossary with the foreman's name and applies to every sheet and
every project after. Over a year the glossary becomes the team's own asset.

**The wow 3: one change, six sheets.** When Rev C changes a bar, every language version regenerates from
the same model data, so there is never a stale Hindi sheet beside a new English one.

**Why it is practical every day.**
- BIM modeller or detailer: no new modelling work; one dropdown at export.
- Site engineer: hands the gang a card they can read and hear; briefing time falls.
- Contractor and client: fewer wrong-bar claims; a clear record of what the gang was given, in which
  language, with whom it was reviewed.

### Implementation

**What already exists in cad2bim (to build on, not replace).** BBS and bar shapes (`docs/design/bbs.md`),
the BBS and BOQ exports, the viewer and the PDF/sheet output path, the AI chat. Nothing leaves the device:
translation is by glossary lookup, not by a web service.

**Pipeline.**
1. **Source of truth is structured data, not text.** Take the BBS rows and pour-card fields as data (bar
   mark, diameter, shape code, dimensions, count, cut length, weight, element, level). Never translate a
   rendered sheet.
2. **Template per output** (BBS card, pour card, fixing steps, toolbox talk, snag list): fixed sentence
   frames per language with slots for numbers and element names ("{count} bars of {dia} mm, cut length
   {len} m"). Frames are written and reviewed by a human once. This is why it is safe: there is no free
   machine translation in the sheet.
3. **Glossary table** `glossary.json` (term id, per-language word, keep-English flag, optional image,
   reviewer, date). Versioned like the drawings (the project-versioning design). Import and export as a
   CSV so a foreman can fix it in Excel.
4. **Rendering.** Draw bar shapes from the existing BBS shape code to SVG; lay out the card as a printable
   page (A5 or A4, two cards per page) and as a phone page.
5. **Fonts.** Indic scripts need fonts. The repo's licence rule allows Apache-2.0 and the Hershey fonts'
   own licence and says SIL OFL is **not** on the list, and the common Indic fonts (the Noto family) are
   OFL. **Open question for the owner:** find Apache-2.0 Indic fonts, or ask for an OFL exception in
   `docs/decisions.md`. Until this is settled the build is blocked for scripts other than Latin
   (Hinglish works first).
6. **Audio.** Use the browser's built-in speech synthesis (`speechSynthesis`) when a voice for the language
   is installed on the phone; otherwise hide the play button. No audio files are bundled. State what was
   measured: voice availability differs by phone, so test on 3 to 4 common Android phones before promising it.
7. **Number check.** After rendering, extract every number from the translated sheet and compare it with
   the English sheet; any mismatch blocks the export and names the line.
8. **Understood quiz.** Questions are made from the model (cover, lap, bar count); answers are stored on
   the device only; the engineer's view is a table on the same device.

**Suggested build order.**
- **Phase 1 (about 1 week):** Hinglish and Hindi bar-bending card from the BBS (PDF), with the glossary,
  the number check and the English line beside. Needs no Indic font if Hinglish is in Roman script.
- **Phase 2:** Marathi and Gujarati, pour card and fixing steps, phone page, the glossary editor, CSV.
- **Phase 3:** Tap-to-listen, the quiz, Tamil, Telugu, Kannada, Bengali; the toolbox talk and snag list.

**Risks.**
- A wrong or rude word on a sheet. Mitigation: glossary-only text, a reviewer line on every sheet, and the
  English line printed beside.
- Fonts (see above) and speech voices are the real blockers, not the code.
- Dialect differences between sites. Mitigation: the glossary is per team and per project, not global.

### Future update

- **Photo of the wrong sheet:** a foreman photographs a sheet that looks wrong; the app reads the mark and
  shows the right card (links with Photo Placer).
- **Voice question:** "ye beam ka lap kitna hai?" answered from the model with a number and 3D proof (links
  with Ask Watch).
- **Pictogram-only mode** for crews who read little: shapes, colours and numbers, no text.
- **Client and authority versions:** the same engine for a Hindi or Marathi summary for a society or a
  local office.
- **Glossary exchange:** the team exports its reviewed glossary so other teams can start from it.
- **Open for the owner to decide:** the font question, the order of languages, and whether the quiz is
  shown to the contractor or kept private to the engineer.
