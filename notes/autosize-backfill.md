# Runbook: Backfill iframe auto-height into existing FLCC note pages

Claude — this is a saved workflow. When I hand it to you, start the "autosize backfill."

## Context
Note pages are self-contained HTML on GitHub Pages (repo: github.com/byungmooncho/byungmooncho.github.io, served at byungmooncho.github.io), embedded into Brightspace (D2L) topics via a small iframe wrapper. Course folders: PHY105, PHY118, PHY151, each with Module_0N subfolders (plus a handouts folder). We built an auto-height system so the embedded iframe fits its content with no scrollbar. Goal: apply it to ALL existing note pages.

## Auto-height system — use these EXACT snippets

Reporter — in each NOTE page, just before the closing body tag:

```
<script>
function reportHeight(){ parent.postMessage({ flccNoteHeight: document.documentElement.scrollHeight }, "*"); }
window.addEventListener("load", reportHeight);
window.addEventListener("resize", reportHeight);
new ResizeObserver(reportHeight).observe(document.body);
</script>
```

Listener — in each Brightspace wrapper (D2L topic, Edit HTML, source view). The wrapper iframe must have id="flcc-note":

```
<script>
window.addEventListener("message", function (e) {
if (e.origin !== "https://byungmooncho.github.io") return;
if (e.data && e.data.flccNoteHeight) {
document.getElementById("flcc-note").style.height = e.data.flccNoteHeight + "px";
}
});
</script>
```

## Two layers — do both

1. Repo (note pages): add the reporter before the closing body tag in every course .html missing it. One sweep via script (clone, insert, commit, push). Idempotent: skip files already containing "flccNoteHeight". Needs a terminal + GitHub push access — either give me a script to run, or connect the Claude desktop app and run it on the local clone.

2. 2. Brightspace (wrappers): add the listener to each D2L topic's wrapper via Edit HTML, source view (paste in SOURCE view so the script survives the WYSIWYG sanitizer). No bulk tool — drive it through Claude in Chrome, topic by topic. First enumerate which topics are wrapper-embeds and skip the PDF-document topics.
  
   3. ## Caveats / rules
   4. - Some wrappers have manually-set iframe heights — do not preserve them; the listener overrides them as a harmless fallback.
      - - Live course: do not change any topic's publish/visibility state; leave Hidden ones Hidden.
        - - Confirm scope first: which courses/modules, and repo-only vs both layers this session.
          - - Optional upgrade: replace the inline listener with a shared file loaded via a script src tag pointing to byungmooncho.github.io/assets/frame-autosize.js in each wrapper, so future tweaks are one-file edits.
           
            - ## Start by asking me
            - - Which course(s)/module(s) this run covers.
              - - Repo push method (script vs desktop app).
                - - Whether to do the D2L wrappers now or just the repo.
                  - 
