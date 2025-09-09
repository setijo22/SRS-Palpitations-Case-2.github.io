# SRS-Palpitations-Case-2.github.io

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>SRS Palpitations</title>
  <style>
    :root {
      --bg: #0f172a;
      --card: #111827;
      --text: #e5e7eb;
      --muted: #9ca3af;
      --accent: #22d3ee;
      --border: #1f2937;
      --focus: #38bdf8;
    }
    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(1200px 800px at 80% -10%, rgba(56,189,248,0.2), transparent 60%),
                  radial-gradient(1200px 800px at -10% 110%, rgba(34,211,238,0.18), transparent 60%),
                  var(--bg);
      color: var(--text);
      display: grid;
      place-items: center;
      padding: 24px;
    }
    .card {
      width: 100%;
      max-width: 720px;
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border: 1px solid var(--border);
      border-radius: 18px;
      padding: 24px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.35);
      backdrop-filter: blur(6px);
    }
    h1 {
      margin: 0 0 10px;
      font-size: 22px;
      letter-spacing: 0.2px;
    }
    p.hint {
      margin: 0 0 18px;
      color: var(--muted);
      font-size: 14px;
    }
    form {
      display: flex;
      gap: 10px;
      margin-bottom: 16px;
    }
    input[type="text"] {
      flex: 1;
      padding: 12px 14px;
      border-radius: 12px;
      border: 1px solid var(--border);
      background: #0b1220;
      color: var(--text);
      outline: none;
      font-size: 16px;
    }
    input[type="text"]:focus {
      border-color: var(--focus);
      box-shadow: 0 0 0 3px rgba(56,189,248,0.25);
    }
    button {
      padding: 12px 16px;
      border-radius: 12px;
      border: 1px solid var(--border);
      background: linear-gradient(180deg, #121a2b, #0f1626);
      color: var(--text);
      font-weight: 600;
      cursor: pointer;
      transition: transform 0.03s ease-in-out, box-shadow 0.15s ease;
    }
    button:hover {
      box-shadow: 0 8px 20px rgba(34,211,238,0.2);
    }
    button:active { transform: translateY(1px); }
    .result { min-height: 44px; display: grid; align-items: center; gap: 10px; }
    .content-card {
      display: grid;
      grid-template-columns: auto 1fr;
      align-items: center;
      gap: 10px;
      padding: 12px 14px;
      border-radius: 12px;
      border: 1px solid var(--border);
      background: #0b1220;
      box-shadow: inset 0 0 0 1px rgba(34,211,238,0.18);
      max-width: 100%;
    }
    .content-card a { color: var(--accent); text-decoration: none; word-break: break-all; }
    .content-card a:hover { text-decoration: underline; }
    .tag {
      font-size: 12px;
      color: var(--muted);
      border: 1px solid var(--border);
      padding: 4px 8px;
      border-radius: 999px;
      align-self: start;
    }
    .text-block { white-space: pre-wrap; line-height: 1.35; }
    img.preview {
      max-width: 100%;
      height: auto;
      border-radius: 10px;
      border: 1px solid var(--border);
    }
    .small { font-size: 12px; color: var(--muted); margin-top: 6px; line-height: 1.4; }
  </style>
</head>
<body>
  <main class="card" role="main" aria-labelledby="title">
    <h1 id="title">Case 2 - EMERGENCY EMERGENCY</h1>
    <p class="hint">Type a keyword (e.g., <code>ecg</code>, <code>bp</code>). If a relevant keyword is given, results will be displayed; otherwise nothing shows. Ensure to use to exact phrase given in the google docs to ensure results show.</p>

    <form id="lookup-form" autocomplete="off">
      <input id="keyword" type="text" placeholder="Enter keyword…" aria-label="Keyword" required />
      <button type="submit" aria-label="Show content">Show</button>
    </form>

    <div id="result" class="result" aria-live="polite"></div>
    <p class="small">Case-insensitive • Exact keyword match • Supports <b>image</b>, <b>link</b> or <b>text</b>.</p>
  </main>

  <script>
    // === Configure your keyword → content mappings here ===
    // type: "image" | "link" | "text"
    const KEY_MAP = {
      "ecg": { type: "image", value: "https://images-prod.healthline.com/hlcmsresource/images/Image-Galleries/torsades-de-pointes/1036-Torsades_de_Pointes-642x361-slide_1.jpg" },
      "bp":  { type: "text",  value: "Blood Pressure: 80/50 mmHg" },
      "hr": { type: "text", value: "160 bpm" },
      "rr": { type: "text", value: "30 breaths/min" },
      "spo2": { type: "text", value: "90% on room air" }
      // Add more entries. If a keyword is not present (e.g., "cxr"), nothing will display.
    };

    const form = document.getElementById("lookup-form");
    const input = document.getElementById("keyword");
    const result = document.getElementById("result");

    function sanitizeText(str) {
      // Minimal escape to avoid injecting HTML if text is used
      return str.replace(/[&<>]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;'}[c]));
    }

    function renderContent(entry, key) {
      const wrapper = document.createElement("div");
      wrapper.className = "content-card";

      const tag = document.createElement("span");
      tag.className = "tag";
      tag.textContent = key;
      wrapper.appendChild(tag);

      if (entry.type === "image") {
        const img = document.createElement("img");
        img.src = entry.value;
        img.alt = key;
        img.loading = "lazy";
        img.className = "preview";
        wrapper.appendChild(img);

      } else if (entry.type === "text") {
        const textEl = document.createElement("div");
        textEl.className = "text-block";
        textEl.textContent = entry.value; // safe text insertion
        wrapper.appendChild(textEl);

      } else if (entry.type === "link") {
        const anchor = document.createElement("a");
        anchor.href = entry.value;
        anchor.target = "_blank";
        anchor.rel = "noopener noreferrer";
        anchor.textContent = entry.value;
        wrapper.appendChild(anchor);
      }

      return wrapper;
    }

    function handleLookup(e) {
      e.preventDefault();
      const raw = input.value.trim();
      const key = raw.toLowerCase();
      result.innerHTML = ""; // clear previous

      if (key && Object.prototype.hasOwnProperty.call(KEY_MAP, key)) {
        const entry = KEY_MAP[key];
        result.appendChild(renderContent(entry, key));
      }
      // else: show nothing
    }

    form.addEventListener("submit", handleLookup);

    input.addEventListener("input", () => {
      if (!input.value.trim()) result.innerHTML = "";
    });

    window.addEventListener("DOMContentLoaded", () => input.focus());
  </script>
</body>
</html>
