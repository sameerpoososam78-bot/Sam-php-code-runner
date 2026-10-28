# Sam-php-code-runner
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>PHP Runner</title>
<style>
  :root {
    color-scheme: light dark;
    --bg: #f3f4f9;
    --panel: #ffffff;
    --line: #dfe2ee;
    --text: #1d2433;
    --muted: #6a7188;
    --accent: #5256c4;
    --accent-ink: #ffffff;
    --gutter: #f0f1f8;
    --ok: #1c7d52;
    --bad: #b8322a;
    --out-bg: #fbfbfe;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #12141c;
      --panel: #1a1d29;
      --line: #2b3042;
      --text: #e7e9f2;
      --muted: #9097b0;
      --accent: #8d91ee;
      --accent-ink: #12141c;
      --gutter: #161925;
      --ok: #55c795;
      --bad: #ff8a80;
      --out-bg: #151822;
    }
  }
  * { box-sizing: border-box; }
  html, body { height: 100%; }
  body {
    margin: 0;
    background: var(--bg);
    color: var(--text);
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    display: flex;
    flex-direction: column;
  }
  header {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
    padding: 12px 16px;
    border-bottom: 1px solid var(--line);
    background: var(--panel);
  }
  h1 { font-size: 1.05rem; margin: 0; font-weight: 700; letter-spacing: .2px; }
  .status {
    font-size: .8rem;
    color: var(--muted);
    display: inline-flex;
    align-items: center;
    gap: 6px;
  }
  .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--muted); }
  .status.ready .dot { background: var(--ok); }
  .status.error .dot { background: var(--bad); }
  .status.busy .dot { animation: pulse 1s ease-in-out infinite; background: var(--accent); }
  @keyframes pulse { 50% { opacity: .25; } }
  @media (prefers-reduced-motion: reduce) { .status.busy .dot { animation: none; } }
  .spacer { flex: 1; }
  select, button {
    font: inherit;
    font-size: .88rem;
    color: var(--text);
    background: var(--panel);
    border: 1px solid var(--line);
    border-radius: 8px;
    padding: 7px 12px;
    cursor: pointer;
  }
  button:hover, select:hover { border-color: var(--accent); }
  button.run {
    background: var(--accent);
    color: var(--accent-ink);
    border-color: var(--accent);
    font-weight: 700;
    padding: 7px 18px;
  }
  button.run:disabled { opacity: .55; cursor: wait; }
  :focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }

  main {
    flex: 1;
    min-height: 0;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    padding: 12px;
  }
  @media (max-width: 860px) {
    main { grid-template-columns: 1fr; grid-template-rows: 1.2fr 1fr; }
  }
  .pane {
    min-height: 0;
    display: flex;
    flex-direction: column;
    background: var(--panel);
    border: 1px solid var(--line);
    border-radius: 10px;
    overflow: hidden;
  }
  .pane-head {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 8px 12px;
    border-bottom: 1px solid var(--line);
    font-size: .82rem;
    color: var(--muted);
  }
  .pane-head button { padding: 3px 10px; font-size: .78rem; }

  .editor { flex: 1; min-height: 0; display: flex; }
  #gutter {
    width: 46px;
    flex: none;
    overflow: hidden;
    background: var(--gutter);
    color: var(--muted);
    text-align: right;
    padding: 12px 8px 12px 0;
    border-right: 1px solid var(--line);
    user-select: none;
  }
  #code, #gutter, #output {
    font-family: ui-monospace, "Cascadia Code", "JetBrains Mono", Consolas, Menlo, monospace;
    font-size: 14px;
    line-height: 1.55;
  }
  #code {
    flex: 1;
    min-width: 0;
    resize: none;
    border: 0;
    outline: 0;
    padding: 12px;
    background: transparent;
    color: var(--text);
    white-space: pre;
    overflow: auto;
    tab-size: 4;
  }
  #output {
    flex: 1;
    min-height: 0;
    margin: 0;
    padding: 12px;
    overflow: auto;
    white-space: pre-wrap;
    word-break: break-word;
    background: var(--out-bg);
  }
  #output .err { color: var(--bad); }
  #output .note { color: var(--muted); font-style: italic; }
  footer {
    padding: 0 16px 10px;
    font-size: .75rem;
    color: var(--muted);
  }
  kbd {
    font-family: inherit;
    border: 1px solid var(--line);
    border-radius: 4px;
    padding: 0 5px;
    background: var(--panel);
  }
</style>
</head>
<body>

<header>
  <h1>PHP Runner</h1>
  <span id="status" class="status busy"><span class="dot"></span><span id="statusText">Loading PHP…</span></span>
  <span class="spacer"></span>
  <label class="sr" for="examples" style="position:absolute;left:-9999px">Examples</label>
  <select id="examples" aria-label="Load an example">
    <option value="">Examples</option>
    <option value="hello">Hello, world</option>
    <option value="loops">Loops and arrays</option>
    <option value="functions">Functions</option>
    <option value="classes">Classes</option>
    <option value="json">JSON and dates</option>
  </select>
  <button id="download" title="Save your code as a .php file">Download .php</button>
  <button id="run" class="run" disabled>Run</button>
</header>

<main>
  <section class="pane" aria-label="Code editor">
    <div class="pane-head"><span>index.php</span></div>
    <div class="editor">
      <div id="gutter" aria-hidden="true"></div>
      <textarea id="code" spellcheck="false" autocapitalize="off" autocomplete="off" aria-label="PHP code"></textarea>
    </div>
  </section>

  <section class="pane" aria-label="Output">
    <div class="pane-head">
      <span>Output <span id="timing"></span></span>
      <button id="clear">Clear</button>
    </div>
    <pre id="output" aria-live="polite"></pre>
  </section>
</main>

<footer>Press <kbd>Ctrl</kbd> + <kbd>Enter</kbd> (or <kbd>Cmd</kbd> + <kbd>Enter</kbd>) to run. PHP runs inside your browser, so no server or XAMPP is needed. Needs internet on first load.</footer>

<script type="module">
import { PhpWeb } from 'https://cdn.jsdelivr.net/npm/php-wasm/PhpWeb.mjs';

const $ = (id) => document.getElementById(id);
const codeEl = $('code'), gutter = $('gutter'), out = $('output');
const runBtn = $('run'), statusEl = $('status'), statusText = $('statusText'), timing = $('timing');

const EXAMPLES = {
  hello: `<?php
echo "Hello, world!\\n";
echo "PHP version: " . PHP_VERSION . "\\n";
`,
  loops: `<?php
$fruits = ["apple", "banana", "cherry"];

foreach ($fruits as $i => $fruit) {
    echo ($i + 1) . ". " . ucfirst($fruit) . "\\n";
}

echo "\\nSquares:\\n";
for ($n = 1; $n <= 5; $n++) {
    echo "$n x $n = " . $n * $n . "\\n";
}

$marks = ["Asha" => 82, "Ravi" => 74, "Meena" => 91];
arsort($marks);
echo "\\nRanking:\\n";
foreach ($marks as $name => $mark) {
    echo str_pad($name, 8) . $mark . "\\n";
}
`,
  functions: `<?php
function factorial(int $n): int {
    return $n <= 1 ? 1 : $n * factorial($n - 1);
}

function isPrime(int $n): bool {
    if ($n < 2) return false;
    for ($i = 2; $i * $i <= $n; $i++) {
        if ($n % $i === 0) return false;
    }
    return true;
}

echo "10! = " . factorial(10) . "\\n";
echo "Primes up to 30: " . implode(", ", array_filter(range(1, 30), 'isPrime')) . "\\n";
`,
  classes: `<?php
class Student {
    public function __construct(
        private string $name,
        private array $marks = []
    ) {}

    public function addMark(int $mark): static {
        $this->marks[] = $mark;
        return $this;
    }

    public function average(): float {
        return count($this->marks) ? array_sum($this->marks) / count($this->marks) : 0;
    }

    public function __toString(): string {
        return sprintf("%s: average %.1f", $this->name, $this->average());
    }
}

$s = (new Student("Sameer"))->addMark(80)->addMark(92)->addMark(77);
echo $s . "\\n";
`,
  json: `<?php
$data = [
    "name" => "Sameer",
    "skills" => ["Java", "Python", "PHP"],
    "year" => 2026,
];

$json = json_encode($data, JSON_PRETTY_PRINT);
echo $json . "\\n\\n";

$back = json_decode($json, true);
echo "Skills: " . implode(" / ", $back["skills"]) . "\\n";
echo "Date: " . date("Y-m-d H:i:s") . "\\n";
`
};

const DEFAULT_CODE = EXAMPLES.hello;

/* ---------- editor helpers ---------- */
function updateGutter() {
  const lines = codeEl.value.split('\n').length;
  let s = '';
  for (let i = 1; i <= lines; i++) s += i + '\n';
  gutter.textContent = s;
  gutter.scrollTop = codeEl.scrollTop;
}
codeEl.addEventListener('input', () => { updateGutter(); save(); });
codeEl.addEventListener('scroll', () => { gutter.scrollTop = codeEl.scrollTop; });
codeEl.addEventListener('keydown', (e) => {
  if (e.key === 'Tab') {
    e.preventDefault();
    const { selectionStart: s, selectionEnd: en, value } = codeEl;
    codeEl.value = value.slice(0, s) + '    ' + value.slice(en);
    codeEl.selectionStart = codeEl.selectionEnd = s + 4;
    updateGutter();
  }
});

function save() { try { localStorage.setItem('php-runner-code', codeEl.value); } catch (_) {} }
function load() { try { return localStorage.getItem('php-runner-code'); } catch (_) { return null; } }

codeEl.value = load() || DEFAULT_CODE;
updateGutter();

/* ---------- status ---------- */
function setStatus(kind, text) {
  statusEl.className = 'status ' + kind;
  statusText.textContent = text;
}

/* ---------- output ---------- */
function toText(d) {
  if (Array.isArray(d)) return d.join('');
  return typeof d === 'string' ? d : String(d ?? '');
}
function append(text, cls) {
  if (!text) return;
  if (cls) {
    const span = document.createElement('span');
    span.className = cls;
    span.textContent = text;
    out.appendChild(span);
  } else {
    out.appendChild(document.createTextNode(text));
  }
  out.scrollTop = out.scrollHeight;
}

/* ---------- run ---------- */
let busy = false;

async function warmUp() {
  try {
    const php = new PhpWeb();
    await php.run('<?php echo "";');
    setStatus('ready', 'PHP ready');
    runBtn.disabled = false;
  } catch (err) {
    setStatus('error', 'Could not load PHP');
    append('PHP could not be loaded. Check your internet connection and reload the page.\n' + (err && err.message ? err.message : err), 'err');
  }
}

async function run() {
  if (busy) return;
  busy = true;
  runBtn.disabled = true;
  out.textContent = '';
  timing.textContent = '';
  setStatus('busy', 'Running…');
  const started = performance.now();
  let produced = false;

  try {
    // A fresh PHP instance for every run keeps each run isolated.
    const php = new PhpWeb();
    php.addEventListener('output', (e) => { produced = true; append(toText(e.detail)); });
    php.addEventListener('error', (e) => { produced = true; append(toText(e.detail), 'err'); });
    await php.run(codeEl.value);
    if (!produced) append('(no output)', 'note');
    setStatus('ready', 'Finished');
  } catch (err) {
    append('\n' + (err && err.message ? err.message : String(err)), 'err');
    setStatus('error', 'Run failed');
  } finally {
    const ms = Math.round(performance.now() - started);
    timing.textContent = '· ' + ms + ' ms';
    busy = false;
    runBtn.disabled = false;
  }
}

runBtn.addEventListener('click', run);
document.addEventListener('keydown', (e) => {
  if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') { e.preventDefault(); if (!runBtn.disabled) run(); }
});

$('clear').addEventListener('click', () => { out.textContent = ''; timing.textContent = ''; });

$('examples').addEventListener('change', (e) => {
  const code = EXAMPLES[e.target.value];
  if (code) { codeEl.value = code; updateGutter(); save(); }
  e.target.value = '';
});

$('download').addEventListener('click', () => {
  const blob = new Blob([codeEl.value], { type: 'text/x-php' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'index.php';
  a.click();
  URL.revokeObjectURL(a.href);
});

warmUp();
</script>
</body>
</html>
