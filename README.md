<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Hangman — The Challenge</title>
<style>
:root{color-scheme:dark;--bg:#0b1020;--panel:#151d32;--panel2:#1b2741;--text:#f4f7ff;--muted:#9eabc7;--accent:#8b5cf6;--accent2:#22d3ee;--good:#34d399;--bad:#fb7185}
*{box-sizing:border-box}
body{margin:0;min-height:100vh;padding:28px 16px;display:grid;place-items:center;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",sans-serif;color:var(--text);background:radial-gradient(ellipse at top,#202a50 0%,var(--bg) 58%)}
.game{width:min(940px,100%);background:rgba(21,29,50,.94);border:1px solid #34405e;border-radius:24px;overflow:hidden;box-shadow:0 24px 80px #0007}
header{text-align:center;padding:28px 20px 20px;background:linear-gradient(110deg,#6d28d9aa,#0891b244);border-bottom:1px solid #34405e}
h1{margin:0;font-size:clamp(1.8rem,5vw,2.7rem);letter-spacing:-.04em}
header p{margin:8px 0 0;color:#d5dcf2}
main{padding:24px;display:grid;grid-template-columns: minmax(220px,.85fr) minmax(0,1.15fr);gap:24px}
.visual,.play{background:var(--panel2);border:1px solid #34405e;border-radius:18px;padding:18px}
.visual{display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:310px}
svg{width:100%;max-width:280px;height:245px;overflow:visible}
.gallows{stroke:#a8b6d6;stroke-width:7;stroke-linecap:round;fill:none}
.person{stroke:var(--accent2);stroke-width:6;stroke-linecap:round;fill:none;transition:opacity .2s}
#status{font-weight:700;margin:8px 0 0;text-align:center}
#status small{display:block;color:var(--muted);font-weight:500;margin-top:4px}
.play{display:flex;flex-direction:column;gap:18px}
.label{font-size:.76rem;text-transform:uppercase;letter-spacing:.13em;color:var(--muted);font-weight:800;text-align:center}
#phrase{display:flex;flex-wrap:wrap;justify-content:center;gap:8px 6px;min-height:76px;align-content:center}
.word{display:flex;gap:3px;margin:0 4px 5px}
.letter{width:clamp(16px,2.6vw,25px);height:38px;border-bottom:3px solid #7483a7;display:grid;place-items:center;font-size:clamp(1rem,2.8vw,1.4rem);font-weight:800;text-transform:uppercase}
.letter.revealed{border-color:var(--accent2);color:white}
.letter.space{width:10px;border:0}
.letter.punctuation{border:0;width:auto;min-width:10px}
#wrongLetters{min-height:26px;text-align:center;color:var(--bad);font-weight:700;letter-spacing:.12em;overflow-wrap:anywhere}
#keyboard{display:grid;grid-template-columns:repeat(7,minmax(0,1fr));gap:7px}
.key{border:1px solid #465477;border-radius:9px;background:#273451;color:var(--text);font-weight:800;font-size:1rem;padding:10px 0;cursor:pointer;transition:transform .12s,background .12s,border-color .12s}
.key:hover:not(:disabled){transform:translateY(-2px);background:#38496f;border-color:#8b9bc4}
.key:disabled{opacity:.32;cursor:not-allowed}
.key.correct{background:#065f46;border-color:#34d399;opacity:1}
.key.wrong{background:#7f1d35;border-color:#fb7185;opacity:.75}
#message{min-height:26px;text-align:center;font-weight:800;font-size:1rem}
#message.win{color:var(--good)}#message.lose{color:var(--bad)}
.actions{display:flex;justify-content:center;gap:10px;flex-wrap:wrap}
.action{border:0;border-radius:10px;padding:11px 18px;font-weight:800;cursor:pointer;color:white;background:linear-gradient(110deg,var(--accent),#6d28d9)}
.action.secondary{background:#303d5c;border:1px solid #506080}
.action:hover{filter:brightness(1.12)}
footer{text-align:center;padding:0 20px 20px;color:var(--muted);font-size:.8rem}
@media(max-width:680px){main{grid-template-columns:1fr;padding:14px;gap:14px}.visual{min-height:245px}svg{height:190px}.play{padding:14px}#keyboard{gap:5px}.key{padding:10px 0}}
</style>
</head>
<body>
<div class="game">
<header>
<h1>🎯 Hangman</h1>
<p>Think you can figure out the secret message?</p>
</header>
<main>
<section class="visual" aria-label="Hangman drawing and game status">
<svg viewBox="0 0 250 230" role="img" aria-label="Hangman drawing">
<path class="gallows" d="M25 210 H145 M55 210 V18 H160 V42"/>
<path class="person" id="part-head" d="M160 44 a19 19 0 1 0 0.1 0" opacity="0"/>
<path class="person" id="part-body" d="M160 82 V143" opacity="0"/>
<path class="person" id="part-left-arm" d="M160 98 L132 124" opacity="0"/>
<path class="person" id="part-right-arm" d="M160 98 L188 124" opacity="0"/>
<path class="person" id="part-left-leg" d="M160 143 L136 178" opacity="0"/>
<path class="person" id="part-right-leg" d="M160 143 L184 178" opacity="0"/>
</svg>
<div id="status">Mistakes: 0 / 6<small>Choose a letter to get started.</small></div>
</section>
<section class="play">
<div class="label">Secret message</div>
<div id="phrase" aria-label="Secret message"></div>
<div>
<div class="label">Incorrect guesses</div>
<div id="wrongLetters">None yet</div>
</div>
<div id="keyboard" aria-label="Letter keyboard"></div>
<div id="message" role="status" aria-live="polite"></div>
<div class="actions"><button class="action" id="restart">↻ Play again</button><button class="action secondary" id="reveal">Reveal message</button></div>
</section>
</main>
<footer>Tip: you can use your physical keyboard, too.</footer>
</div>
<script>
(() => {
  const secret = "I made you do this and your still gay";
  const target = secret.toLowerCase();
  const maxWrong = 6;
  const phraseEl = document.getElementById("phrase");
  const keyboardEl = document.getElementById("keyboard");
  const wrongEl = document.getElementById("wrongLetters");
  const statusEl = document.getElementById("status");
  const messageEl = document.getElementById("message");
  const restartBtn = document.getElementById("restart");
  const revealBtn = document.getElementById("reveal");
  const parts = ["part-head","part-body","part-left-arm","part-right-arm","part-left-leg","part-right-leg"];
  let guessed = new Set();
  let wrong = [];
  let finished = false;

  function renderPhrase() {
    phraseEl.replaceChildren();
    const words = secret.split(" ");
    words.forEach((word, wordIndex) => {
      const wordEl = document.createElement("span");
      wordEl.className = "word";
      for (const char of word) {
        const tile = document.createElement("span");
        tile.className = "letter";
        if (/[a-z]/i.test(char)) {
          tile.textContent = guessed.has(char.toLowerCase()) || finished ? char : "";
          if (guessed.has(char.toLowerCase()) || finished) tile.classList.add("revealed");
          tile.setAttribute("aria-label", guessed.has(char.toLowerCase()) || finished ? char : "hidden letter");
        } else {
          tile.textContent = char;
          tile.classList.add("punctuation");
        }
        wordEl.appendChild(tile);
      }
      phraseEl.appendChild(wordEl);
      if (wordIndex < words.length - 1) {
        const spacer = document.createElement("span");
        spacer.className = "letter space";
        phraseEl.appendChild(spacer);
      }
    });
  }

  function renderKeyboard() {
    keyboardEl.replaceChildren();
    for (const letter of "abcdefghijklmnopqrstuvwxyz") {
      const btn = document.createElement("button");
      btn.type = "button";
      btn.className = "key";
      btn.textContent = letter.toUpperCase();
      btn.disabled = guessed.has(letter) || finished;
      if (guessed.has(letter)) btn.classList.add(target.includes(letter) ? "correct" : "wrong");
      btn.setAttribute("aria-label", "Guess " + letter);
      btn.addEventListener("click", () => guess(letter));
      keyboardEl.appendChild(btn);
    }
  }

  function renderStatus() {
    statusEl.innerHTML = "Mistakes: " + wrong.length + " / " + maxWrong +
      "<small>" + (finished ? "Round finished." : (maxWrong - wrong.length) + " guesses left") + "</small>";
    wrongEl.textContent = wrong.length ? wrong.join(" · ").toUpperCase() : "None yet";
    parts.forEach((id, index) => document.getElementById(id).setAttribute("opacity", index < wrong.length ? "1" : "0"));
  }

  function finish(won) {
    finished = true;
    renderPhrase();
    renderKeyboard();
    messageEl.className = won ? "win" : "lose";
    messageEl.textContent = won ? "🎉 You got it! Message decoded." : "💀 Game over! The message is revealed.";
    renderStatus();
  }

  function guess(letter) {
    if (finished || guessed.has(letter)) return;
    guessed.add(letter);
    if (!target.includes(letter)) wrong.push(letter);
    const letters = [...target].filter(char => /[a-z]/.test(char));
    const won = letters.every(letter => guessed.has(letter));
    if (won) finish(true);
    else if (wrong.length >= maxWrong) finish(false);
    else {
      renderPhrase();
      renderKeyboard();
      renderStatus();
      messageEl.className = "";
      messageEl.textContent = target.includes(letter) ? "Nice! Keep going." : "Nope, try another letter.";
    }
  }

  function reset() {
    guessed = new Set();
    wrong = [];
    finished = false;
    messageEl.className = "";
    messageEl.textContent = "";
    renderPhrase();
    renderKeyboard();
    renderStatus();
  }

  document.addEventListener("keydown", event => {
    if (/^[a-z]$/i.test(event.key) && !event.ctrlKey && !event.altKey && !event.metaKey) guess(event.key.toLowerCase());
  });
  restartBtn.addEventListener("click", reset);
  revealBtn.addEventListener("click", () => finish(false));
  reset();
})();
</script>
</body>
</html>