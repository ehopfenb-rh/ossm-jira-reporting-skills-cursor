# HTML Template for Product Pages Update

Use this HTML structure when generating the update. Google Drive converts this to native Google Docs formatting (headings, bullets, bold) on import.

## Template

```html
<html>
<head><style>
body { font-family: Arial; font-size: 11pt; }
h3 { font-family: Arial; font-size: 11pt; }
ul { margin-top: 2px; margin-bottom: 2px; }
li { font-family: Arial; font-size: 11pt; margin-bottom: 4px; }
p { font-family: Arial; font-size: 11pt; margin-bottom: 4px; }
</style></head>
<body>

<h3>[DATE e.g. 6/4/26]</h3>

<p><b>Summary:</b> [2 sentences max. Sentence 1 = minor release status. Sentence 2 = Z-Stream or notable update.]</p>

<p><b>Minor Releases:</b></p>
<ul>
  <li><b>[Version]:</b> [Target date and status.]
    <ul>
      <li><b>[Sub-topic]:</b> [Details.]</li>
      <li><b>[Sub-topic]:</b> [Details.]</li>
    </ul>
  </li>
</ul>

<p><b>Z-Streams:</b></p>
<ul>
  <li><b>[Stream]:</b> [Status and details.]
    <ul>
      <li><b>[Sub-topic]:</b> [Details.]</li>
    </ul>
  </li>
  <li><b>[Stream]:</b> [Status and details.]
    <ul>
      <li><b>[Sub-topic]:</b> [Details.]</li>
    </ul>
  </li>
</ul>

<p><b>Strategic Program &amp; Integration Updates:</b></p>
<ul>
  <li><b>[Topic]:</b> [Details.]</li>
  <li><b>[Topic]:</b> [Details.]</li>
</ul>

<!-- Include Action & Mitigation Plan only when there are active risks -->
<p><b>Action &amp; Mitigation Plan:</b></p>
<p>Target Release Date: [date] ([AT RISK / ON TRACK / OFF TRACK])</p>
<p>Owner: [name]</p>
<p>Status Update: [details]</p>
<p>Component Status:</p>
<ul>
  <li>[Component]: [Status] ([date])</li>
</ul>

</body>
</html>
```

## Formatting Rules

- **Font:** Arial 11pt for all body text
- **Date header:** Use `<h3>` — Google Docs converts this to a sub-heading that matches existing entries
- **Section headers** (Summary, Minor Releases, Z-Streams, etc.): Bold text in `<p>` tags, NOT headings
- **Bullet items:** Use `<ul>/<li>` with bold label followed by colon and details
- **Nested bullets:** Use nested `<ul>` inside `<li>` for sub-items
- **HTML entities:** Use `&amp;` for &, `&ldquo;`/`&rdquo;` for smart quotes
- **No extra spacing:** Keep margins tight so the output matches the existing doc style
