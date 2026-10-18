<script setup>
import { ref, computed } from 'vue'
import { IonApp, IonPage, IonHeader, IonToolbar, IonContent, IonButton } from '@ionic/vue'

// All logic and styling are contained in this App.vue file.
const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')
const cipher = ref('vigenere')
const mode = ref('encrypt')
const message = ref('THANKYOU')
const keyword = ref('COVER')
const caesarShift = ref(3)
const vigenereShift = ref(0)
const activeIndex = ref(0)
const copyMessage = ref('')

function normalizeShift(value) {
  const number = Number(value)
  return Number.isFinite(number) ? ((Math.trunc(number) % 26) + 26) % 26 : 0
}

function clampShift(value) {
  const number = Number(value)
  return Number.isFinite(number) ? Math.max(0, Math.min(25, Math.trunc(number))) : 0
}

function isLetter(char) {
  return /^[A-Za-z]$/.test(char)
}

function rotate(char, amount) {
  if (!isLetter(char)) return char
  const code = char.charCodeAt(0)
  const base = code >= 97 && code <= 122 ? 97 : 65
  return String.fromCharCode(base + ((code - base + amount) % 26 + 26) % 26)
}

const cleanKey = computed(() => keyword.value.toUpperCase().replace(/[^A-Z]/g, ''))
const invalidKey = computed(() => cipher.value === 'vigenere' && !cleanKey.value)

const currentShift = computed({
  get: () => cipher.value === 'caesar' ? caesarShift.value : vigenereShift.value,
  set: value => {
    const safe = clampShift(value)
    if (cipher.value === 'caesar') caesarShift.value = safe
    else vigenereShift.value = safe
  }
})

// Each step represents one alphabetic character (spaces/punctuation are skipped).
const steps = computed(() => {
  if (invalidKey.value) return []
  const list = []
  let letterIndex = 0

  for (const char of message.value) {
    if (!isLetter(char)) continue

    const keyLetter = cipher.value === 'vigenere'
      ? cleanKey.value[letterIndex % cleanKey.value.length]
      : ''
    const keyShift = keyLetter ? keyLetter.charCodeAt(0) - 65 : 0
    const amount = cipher.value === 'caesar'
      ? normalizeShift(caesarShift.value)
      : normalizeShift(keyShift + vigenereShift.value)
    const output = rotate(char, mode.value === 'decrypt' ? -amount : amount)

    list.push({
      index: letterIndex,
      input: char.toUpperCase(),
      output: output.toUpperCase(),
      outputRaw: output,
      keyLetter,
      effectiveKey: alphabet[amount],
      amount,
      plaintext: (mode.value === 'encrypt' ? char : output).toUpperCase(),
      ciphertext: (mode.value === 'encrypt' ? output : char).toUpperCase()
    })
    letterIndex++
  }
  return list
})

const result = computed(() => {
  if (invalidKey.value) return ''
  let index = 0
  return [...message.value].map(char => isLetter(char) ? steps.value[index++].outputRaw : char).join('')
})

const activeStep = computed(() => steps.value[Math.min(activeIndex.value, steps.value.length - 1)] || null)
const previewShift = computed(() => normalizeShift(caesarShift.value))
const shiftedAlphabet = computed(() => alphabet.map(letter => rotate(letter, mode.value === 'encrypt' ? previewShift.value : -previewShift.value)))
const letterCount = computed(() => steps.value.length)

function matrixLetter(rowIndex, columnIndex) {
  return alphabet[(rowIndex + columnIndex) % 26]
}

function chooseCipher(next) {
  cipher.value = next
  activeIndex.value = 0
  copyMessage.value = ''
}

function chooseMode(next) {
  mode.value = next
  activeIndex.value = 0
  copyMessage.value = ''
}

function reverseText() {
  if (!result.value) return
  message.value = result.value
  mode.value = mode.value === 'encrypt' ? 'decrypt' : 'encrypt'
  activeIndex.value = 0
}

function useExample() {
  if (cipher.value === 'vigenere') {
    message.value = 'THANKYOU'
    keyword.value = 'COVER'
    vigenereShift.value = 0
  } else {
    message.value = 'HELLO WORLD'
    caesarShift.value = 3
  }
  mode.value = 'encrypt'
  activeIndex.value = 0
}

async function copyResult() {
  if (!result.value) return
  try {
    await navigator.clipboard.writeText(result.value)
    copyMessage.value = 'Copied to clipboard!'
  } catch {
    copyMessage.value = 'Select and copy the output manually.'
  }
}
</script>

<template>
  <IonApp>
    <IonPage>
      <IonHeader class="ion-no-border">
        <IonToolbar class="rose-toolbar">
          <div class="nav-inner">
            <div class="logo-lockup">
              <div class="logo-emblem">✦</div>
              <div class="logo-copy"><b>CYBER<span>LEN</span></b><small>CRYPTOGRAPHY STUDIO</small></div>
            </div>
            <div class="nav-status"><span class="pulse"></span> LAB ONLINE <span class="nav-version">// V 1.0</span>
            </div>
          </div>
        </IonToolbar>
      </IonHeader>

      <IonContent>
        <main class="site-background">
          <div class="page-wrap">
            <section class="hero">
              <div class="hero-copy">
                <div class="eyebrow"><span class="eyebrow-spark">✧</span> A FEMININE TAKE ON CYBER SECURITY</div>
                <h1>Encrypt in <em>style.</em><br />Decode with <em>power.</em></h1>
                <p>Turn ordinary text into secret messages with Caesar and Vigenère ciphers. Explore the classic cipher
                  table, one letter at a time.</p>
                <div class="hero-chips"><span>✦ CAESAR CIPHER</span><span>✧ VIGENÈRE CIPHER</span><span>◈ INTERACTIVE
                    MATRIX</span></div>
              </div>
              <div class="hero-art" aria-label="Female cybersecurity analyst illustration">
                <div class="art-ring ring-outer"></div>
                <div class="art-ring ring-inner"></div>
                <div class="art-stars star-one">✦</div>
                <div class="art-stars star-two">✧</div>
                <div class="art-stars star-three">✦</div>
                <div class="avatar-orb"><span class="avatar-emoji" role="img"
                    aria-label="Woman using a laptop">👩🏻‍💻</span></div>
                <div class="floating-label"><span class="label-dot"></span> CYBER ANALYST <b>01</b></div>
              </div>
            </section>

            <div class="work-grid">
              <section class="glass-panel control-panel">
                <div class="panel-head">
                  <div><small>01 // CONTROL CENTER</small>
                    <h2>Choose your cipher <span>✧</span></h2>
                    <p>Personalize the way your message transforms.</p>
                  </div><span class="head-symbol">⌘</span>
                </div>

                <div class="form-section">
                  <div class="field-title"><span class="field-count">01</span> Encryption method</div>
                  <div class="method-grid">
                    <button type="button" class="method-card" :class="{ picked: cipher === 'caesar' }"
                      @click="chooseCipher('caesar')"><span class="method-monogram">C</span><b>Caesar</b><small>Fixed
                        shift</small><span class="method-check">✓</span></button>
                    <button type="button" class="method-card" :class="{ picked: cipher === 'vigenere' }"
                      @click="chooseCipher('vigenere')"><span
                        class="method-monogram">V</span><b>Vigenère</b><small>Keyword based</small><span
                        class="method-check">✓</span></button>
                  </div>
                </div>

                <div class="form-section">
                  <div class="field-title"><span class="field-count">02</span> Select operation</div>
                  <div class="mode-switch"><button type="button" :class="{ chosen: mode === 'encrypt' }"
                      @click="chooseMode('encrypt')">↗ Encrypt</button><button type="button"
                      :class="{ chosen: mode === 'decrypt' }" @click="chooseMode('decrypt')">↙ Decrypt</button></div>
                </div>

                <div v-if="cipher === 'vigenere'" class="form-section">
                  <label class="field-title" for="keyword"><span class="field-count">03</span> Secret keyword</label>
                  <div class="field-input"><span>✦ KEY</span><input id="keyword" v-model="keyword"
                      placeholder="e.g. COVER" type="text" autocomplete="off" spellcheck="false" /></div>
                  <p v-if="invalidKey" class="input-warning">Please enter at least one English letter.</p>
                  <p v-else class="input-help">Repeating keyword: <strong>{{ cleanKey }}</strong></p>
                </div>

                <div class="form-section">
                  <label class="field-title" for="shift-number"><span class="field-count">{{ cipher === 'caesar' ? '03'
                      : '04' }}</span> {{ cipher === 'caesar' ? 'Alphabet shift' : 'Optional extra shift' }}</label>
                  <div class="shift-card">
                    <div class="shift-top">
                      <div><small>SHIFT VALUE</small>
                        <p>{{ cipher === 'caesar' ? 'Rotate every letter' : 'Added to keyword shifts' }}</p>
                      </div>
                      <div class="shift-value"><input id="shift-number" v-model.number="currentShift" type="number"
                          min="0" max="25" @blur="currentShift = clampShift(currentShift)" /><span>/25</span></div>
                    </div>
                    <input v-model.number="currentShift" class="rose-range" type="range" min="0" max="25" step="1"
                      aria-label="Shift amount" />
                    <div class="range-ends"><span>0 / NO EXTRA ROTATION</span><span>25 / MAX</span></div>
                  </div>
                </div>
                <div class="tip-box"><span>♡</span>
                  <p v-if="cipher === 'caesar'"><strong>Caesar tip:</strong> A shift of 3 changes A → D, B → E and C →
                    F. Decryption moves in the opposite direction.</p>
                  <p v-else><strong>Vigenère tip:</strong> The secret keyword is repeated over the letters of your
                    message. Set the additional shift to 0 for the traditional method.</p>
                </div>
              </section>

              <section class="glass-panel editor-panel">
                <div class="panel-head">
                  <div><small>02 // SECRET MESSAGE</small>
                    <h2>Message lab <span>♡</span></h2>
                    <p>Type your text and see the result instantly.</p>
                  </div><span class="head-symbol">✉</span>
                </div>
                <div class="editor-top"><label for="source"><span class="dot violet"></span> YOUR MESSAGE</label>
                  <div class="editor-links"><button type="button" @click="useExample">Example</button><button
                      type="button" @click="message = ''; activeIndex = 0; copyMessage = ''">Clear</button></div>
                </div>
                <div class="editor-area"><textarea id="source" v-model="message"
                    placeholder="Type a secret message here..." spellcheck="false"
                    @input="activeIndex = 0; copyMessage = ''"></textarea>
                  <div class="editor-meta"><span>{{ message.length }} characters</span><span>{{ letterCount }}
                      letters</span></div>
                </div>
                <div class="transform-indicator">
                  <div class="transform-line"></div>
                  <div class="transform-text"><span>↓</span> {{ mode === 'encrypt' ? 'ENCRYPTING' : 'DECRYPTING' }}
                    <b>{{ cipher === 'caesar' ? 'CAESAR' : 'VIGENÈRE' }}</b></div>
                  <div class="transform-line"></div>
                </div>
                <div class="editor-top"><label for="output"><span class="dot pink"></span> {{ mode === 'encrypt' ?
                    'ENCRYPTED' : 'DECRYPTED' }} RESULT</label><span class="result-state">{{ result ? '● READY' : '○ WAITING' }}</span></div>
                <div class="editor-area output-area"><textarea id="output" :value="result"
                    :placeholder="invalidKey ? 'Enter a keyword to continue...' : 'The magic appears here...'" readonly
                    spellcheck="false"></textarea>
                  <div class="editor-meta"><span>{{ result.length }} characters</span><span>LIVE OUTPUT</span></div>
                </div>
                <div class="buttons-row">
                  <IonButton class="copy-btn" expand="block" :disabled="!result" @click="copyResult">✦ &nbsp; Copy
                    result</IonButton>
                  <IonButton class="reverse-btn" expand="block" :disabled="!result" @click="reverseText">⇄ &nbsp;
                    Reverse</IonButton>
                </div>
                <div v-if="copyMessage" class="copy-feedback" role="status">{{ copyMessage }}</div>
              </section>
            </div>

            <section class="glass-panel visual-panel">
              <div class="visual-head">
                <div><small>03 // BEHIND THE ENCRYPTION</small>
                  <h2>{{ cipher === 'vigenere' ? 'Vigenère cipher table' : 'Caesar alphabet shift' }} <span>✧</span>
                  </h2>
                  <p v-if="cipher === 'vigenere'">The authentic 26 × 26 Tabula Recta. Select a letter below to highlight
                    its row, column, and intersection.</p>
                  <p v-else>See how the entire alphabet rotates using your selected shift.</p>
                </div>
                <div class="visual-badge">{{ cipher === 'vigenere' ? '26 × 26 MATRIX' : `SHIFT ${mode === 'encrypt' ?
                  '+' : '−'}${previewShift}` }}</div>
              </div>

              <template v-if="cipher === 'vigenere'">
                <div v-if="steps.length" class="trace-wrap">
                  <div class="trace-header"><strong>MESSAGE WALKTHROUGH</strong><span>Tap any letter to inspect its
                      transformation</span></div>
                  <div class="trace-list"><button v-for="step in steps" :key="step.index" type="button"
                      class="trace-letter" :class="{ focused: activeStep?.index === step.index }"
                      @click="activeIndex = step.index"><small>#{{ step.index + 1 }}</small><strong>{{ step.input
                        }}</strong><span>{{ step.output }}</span></button></div>
                </div>
                <div v-if="activeStep" class="equation-box">
                  <div><small>{{ mode === 'encrypt' ? 'PLAINTEXT' : 'CIPHERTEXT' }}</small><strong>{{ activeStep.input
                      }}</strong></div><span class="math-sign">+</span>
                  <div><small>KEY LETTER</small><strong>{{ activeStep.keyLetter }}</strong></div><span
                    class="math-sign">{{ vigenereShift ? `+ ${vigenereShift}` : '→' }}</span>
                  <div><small>EFFECTIVE SHIFT</small><strong>{{ activeStep.effectiveKey }} / {{ activeStep.amount
                      }}</strong></div><span class="math-sign">=</span>
                  <div class="equation-answer"><small>{{ mode === 'encrypt' ? 'CIPHERTEXT' : 'PLAINTEXT'
                      }}</small><strong>{{ activeStep.output }}</strong></div>
                </div>
                <div class="matrix-scroll">
                  <table class="cipher-matrix" aria-label="Interactive Vigenère cipher table">
                    <thead>
                      <tr>
                        <th class="matrix-corner" scope="col">KEY ↓<br />TEXT →</th>
                        <th v-for="letter in alphabet" :key="letter" scope="col"
                          :class="{ 'matrix-col-active': activeStep?.plaintext === letter }">{{ letter }}</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(rowLetter, rowIndex) in alphabet" :key="rowLetter">
                        <th scope="row" :class="{ 'matrix-row-active': activeStep?.effectiveKey === rowLetter }">{{
                          rowLetter }}</th>
                        <td v-for="(columnLetter, colIndex) in alphabet" :key="columnLetter"
                          :class="{ 'matrix-row-highlight': activeStep?.effectiveKey === rowLetter, 'matrix-col-highlight': activeStep?.plaintext === columnLetter, 'matrix-intersection': activeStep?.effectiveKey === rowLetter && activeStep?.plaintext === columnLetter }">
                          {{ matrixLetter(rowIndex, colIndex) }}</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
                <div class="matrix-legend"><span><i class="legend-pink"></i> Effective keyword row</span><span><i
                      class="legend-violet"></i> Plaintext column</span><span><i class="legend-gold"></i> Ciphertext
                    intersection</span></div>
                <p class="matrix-note">{{ activeStep ? `Selected letter: ${activeStep.plaintext} (column) +
                  ${activeStep.effectiveKey} (row) = ${activeStep.ciphertext} (intersection).` : 'Enter a message andkeyword to highlight an intersection.' }} On decryption, the intersection shows the input ciphertext
                  and the column reveals the original plaintext.</p>
              </template>

              <template v-else>
                <div class="caesar-scroll">
                  <div class="caesar-grid">
                    <div v-for="(letter, i) in alphabet" :key="letter" class="caesar-letter"><span>{{ letter
                        }}</span><em>↓</em><b>{{ shiftedAlphabet[i] }}</b></div>
                  </div>
                </div>
                <p class="matrix-note">{{ mode === 'encrypt' ? 'Encryption' : 'Decryption' }} shifts letters {{ mode ===
                  'encrypt' ? 'forward' : 'backward' }} by {{ previewShift }} positions. Spaces, numbers, and
                  punctuation stay unchanged.</p>
              </template>
            </section>

            <footer class="page-footer">
              <div><b>✦ CYBERROSE</b><span> / YOUR CIPHER STUDIO</span></div>
              <p>For learning classical cryptography. Neither cipher is secure for sensitive data.</p>
            </footer>
          </div>
        </main>
      </IonContent>
    </IonPage>
  </IonApp>
</template>

<style>
:root {
  font-family: Inter, 'Segoe UI', Arial, sans-serif;
  color: #f7eaff;
}

* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
}

.rose-toolbar {
  --background: #170f2b;
  --color: #fff4fc;
}

ion-content {
  --background: #100b20;
}

.nav-inner {
  max-width: 1250px;
  margin: auto;
  height: 75px;
  padding: 0 22px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #493050;
}

.logo-lockup {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-emblem {
  width: 42px;
  height: 42px;
  display: grid;
  place-items: center;
  border-radius: 13px;
  color: #fff;
  font-size: 24px;
  background: linear-gradient(135deg, #f365af, #9657ed);
  box-shadow: 0 4px 22px #c34ddd44;
}

.logo-copy b {
  display: block;
  font-size: 20px;
  letter-spacing: 1px;
}

.logo-copy b span {
  color: #ff92cc;
}

.logo-copy small {
  display: block;
  margin-top: 3px;
  color: #baa9cb;
  font-size: 9px;
  letter-spacing: 2px;
}

.nav-status {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #e1bfe2;
  font-size: 10px;
  letter-spacing: 1.5px;
}

.nav-version {
  color: #886fa3;
}

.pulse {
  display: inline-block;
  height: 7px;
  width: 7px;
  background: #ff89cf;
  border-radius: 50%;
  box-shadow: 0 0 12px #ff6abf;
}

.site-background {
  min-height: 100%;
  background: radial-gradient(ellipse at 82% 3%, #6f32763b, transparent 42%), radial-gradient(ellipse at 10% 55%, #5535a52d, transparent 50%), #100b20;
}

.page-wrap {
  max-width: 1200px;
  margin: auto;
  padding: 0 20px;
}

.hero {
  display: grid;
  grid-template-columns: 1.35fr .65fr;
  align-items: center;
  min-height: 310px;
  gap: 20px;
  padding: 38px 0 30px;
}

.eyebrow {
  color: #ff8dc9;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 2px;
}

.eyebrow-spark {
  font-size: 18px;
  margin-right: 8px;
}

.hero h1 {
  color: #fff5ff;
  font-size: clamp(34px, 4.6vw, 54px);
  letter-spacing: -1.8px;
  line-height: 1.12;
  margin: 18px 0;
}

.hero h1 em {
  color: #ff8bcd;
  font-style: normal;
}

.hero p {
  color: #c4b3d5;
  font-size: 14px;
  line-height: 1.9;
  max-width: 570px;
}

.hero-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 23px;
}

.hero-chips span {
  background: #2a183c;
  border: 1px solid #65385f;
  color: #e9b7dc;
  padding: 8px 10px;
  border-radius: 20px;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: .7px;
}

.hero-art {
  width: 268px;
  height: 242px;
  position: relative;
  margin: auto;
  display: grid;
  place-items: center;
}

.art-ring {
  position: absolute;
  border-radius: 50%;
}

.ring-outer {
  width: 230px;
  height: 230px;
  border: 1px dashed #ac70b5a3;
}

.ring-inner {
  width: 185px;
  height: 185px;
  border: 2px solid #ca66b982;
  box-shadow: 0 0 40px #d566c333;
}

.avatar-orb {
  width: 158px;
  height: 158px;
  border-radius: 50%;
  display: grid;
  place-items: center;
  background: linear-gradient(140deg, #ed9ad2, #7e46ba 80%);
  box-shadow: 0 0 38px #ff79c680, inset 0 0 20px #ffffff38;
}

.avatar-emoji {
  font-size: 79px;
  filter: drop-shadow(0 8px 9px #30133c88);
}

.art-stars {
  position: absolute;
  color: #ffb6e6;
  text-shadow: 0 0 20px #ff77c1;
}

.star-one {
  top: 9px;
  left: 35px;
  font-size: 20px;
}

.star-two {
  top: 40px;
  right: 12px;
  font-size: 28px;
}

.star-three {
  bottom: 36px;
  left: 12px;
  font-size: 15px;
}

.floating-label {
  position: absolute;
  bottom: 10px;
  right: -8px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 13px;
  border: 1px solid #d56eaf;
  border-radius: 8px;
  color: #f7d8ed;
  background: #281737;
  box-shadow: 0 12px 28px #08051780;
  font-size: 10px;
  letter-spacing: 1px;
}

.label-dot {
  background: #ff81ca;
  width: 7px;
  height: 7px;
  border-radius: 50%;
}

.floating-label b {
  color: #ff98d2;
}

.work-grid {
  display: grid;
  grid-template-columns: .92fr 1.08fr;
  gap: 18px;
  align-items: stretch;
}

.glass-panel {
  min-width: 0;
  padding: 25px;
  border: 1px solid #614069;
  border-radius: 18px;
  background: linear-gradient(160deg, #281934f2, #1a142ff5);
  box-shadow: 0 15px 45px #08061440, inset 0 1px 0 #e3a2eb1c;
}

.panel-head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  border-bottom: 1px solid #5a3a63;
  padding-bottom: 21px;
  margin-bottom: 24px;
}

.panel-head small,
.visual-head small {
  font-size: 10px;
  font-weight: 800;
  color: #ff8dc9;
  letter-spacing: 1.6px;
}

.panel-head h2,
.visual-head h2 {
  margin: 8px 0;
  color: #fff3fd;
  font-size: 22px;
  letter-spacing: -.5px;
}

.panel-head h2 span,
.visual-head h2 span {
  color: #fe8bcb;
}

.panel-head p,
.visual-head p {
  color: #bba6cd;
  margin: 0;
  font-size: 12px;
  line-height: 1.6;
}

.head-symbol {
  color: #c475c9;
  font-size: 25px;
}

.form-section {
  margin-bottom: 24px;
}

.field-title {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  font-weight: 750;
  margin-bottom: 12px;
  color: #f4deef;
}

.field-count {
  color: #fb87c8;
  font: bold 10px monospace;
}

.method-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.method-card {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 5px;
  text-align: left;
  border: 1px solid #5c4068;
  background: #1b152d;
  border-radius: 12px;
  color: #e9d7ee;
  padding: 13px;
  cursor: pointer;
}

.method-card.picked {
  background: #442042;
  border-color: #fb85c6;
  box-shadow: inset 0 0 0 1px #ff88c52b;
}

.method-monogram {
  display: grid;
  place-items: center;
  width: 34px;
  height: 34px;
  background: #402649;
  color: #ffabd7;
  border-radius: 9px;
  font-weight: bold;
}

.picked .method-monogram {
  background: #a94780;
  color: white;
}

.method-card b {
  font-size: 13px;
  margin-top: 4px;
}

.method-card small {
  font-size: 11px;
  color: #bda7ce;
}

.method-check {
  position: absolute;
  right: 12px;
  top: 13px;
  color: #ff93d2;
  opacity: 0;
}

.picked .method-check {
  opacity: 1;
}

.mode-switch {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 5px;
  border: 1px solid #62436c;
  border-radius: 10px;
  background: #161124;
  padding: 5px;
}

.mode-switch button {
  background: transparent;
  border: 0;
  color: #c6b0d0;
  padding: 11px;
  border-radius: 7px;
  cursor: pointer;
  font-weight: 700;
}

.mode-switch button.chosen {
  color: #fff0fa;
  background: linear-gradient(120deg, #cc549b, #8c4bce);
  box-shadow: 0 4px 16px #a14fc03a;
}

.field-input {
  display: flex;
  border: 1px solid #68466e;
  border-radius: 10px;
  overflow: hidden;
  background: #171226;
}

.field-input:focus-within {
  border-color: #ff8bcc;
}

.field-input span {
  display: grid;
  place-items: center;
  padding: 0 14px;
  border-right: 1px solid #68466e;
  color: #ff9ccd;
  font-size: 10px;
  font-weight: 800;
  white-space: nowrap;
}

.field-input input {
  width: 100%;
  min-width: 0;
  background: transparent;
  border: 0;
  outline: 0;
  padding: 14px;
  color: white;
  font-size: 13px;
}

.field-input input::placeholder {
  color: #9b83ac;
}

.input-help,
.input-warning {
  color: #b9a4c8;
  font-size: 11px;
  margin: 10px 0 0;
}

.input-help strong {
  color: #ff9bd2;
  letter-spacing: 1px;
  overflow-wrap: anywhere;
}

.input-warning {
  color: #ffaaaa;
}

.shift-card {
  border: 1px solid #65436e;
  border-radius: 10px;
  padding: 15px;
  background: #171226;
}

.shift-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.shift-top small {
  color: #bf9ebf;
  font-size: 9px;
  letter-spacing: 1px;
}

.shift-top p {
  color: #efdbef;
  font-size: 12px;
  margin: 6px 0 0;
}

.shift-value {
  display: flex;
  align-items: center;
  gap: 7px;
}

.shift-value input {
  width: 53px;
  height: 41px;
  border-radius: 8px;
  border: 1px solid #b65d9b;
  text-align: center;
  background: #472042;
  color: #ff9dd1;
  font-size: 21px;
}

.shift-value span {
  color: #aa92b9;
  font-size: 11px;
}

.rose-range {
  display: block;
  width: 100%;
  margin: 21px 0 10px;
  accent-color: #fa7ac0;
  cursor: pointer;
}

.range-ends {
  display: flex;
  justify-content: space-between;
  color: #ab93ba;
  font-size: 9px;
  letter-spacing: .3px;
}

.tip-box {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  border: 1px solid #88517b;
  border-radius: 11px;
  padding: 14px;
  background: #40213e80;
}

.tip-box>span {
  color: #ff91c9;
  font-size: 20px;
}

.tip-box p {
  margin: 0;
  font-size: 11px;
  line-height: 1.7;
  color: #d7b9d4;
}

.tip-box strong {
  color: #ffc4e2;
}

.editor-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}

.editor-top label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #ecd7ec;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1px;
}

.dot {
  width: 7px;
  height: 7px;
  display: inline-block;
  border-radius: 50%;
}

.dot.violet {
  background: #b29bff;
}

.dot.pink {
  background: #ff85c5;
}

.editor-links {
  display: flex;
  gap: 12px;
}

.editor-links button {
  border: 0;
  background: transparent;
  font-size: 11px;
  color: #e2a4d4;
  cursor: pointer;
}

.editor-links button:hover {
  color: white;
}

.editor-area {
  border: 1px solid #664671;
  border-radius: 10px;
  background: #151023;
  overflow: hidden;
}

.editor-area:focus-within {
  border-color: #d26eb9;
}

.editor-area textarea {
  display: block;
  resize: vertical;
  width: 100%;
  min-height: 158px;
  border: 0;
  outline: none;
  background: transparent;
  color: #f8eefa;
  padding: 17px;
  font-family: Consolas, 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.8;
}

.editor-area textarea::placeholder {
  color: #816b95;
}

.editor-meta {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  border-top: 1px solid #53395f;
  padding: 10px 14px;
  color: #a993b8;
  font-size: 10px;
}

.output-area {
  background: #341b3c;
  border-color: #a85896;
}

.output-area textarea {
  color: #ffaadd;
}

.output-area .editor-meta {
  border-color: #774a76;
}

.transform-indicator {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 19px 0;
}

.transform-line {
  flex: 1;
  height: 1px;
  background: #60406b;
}

.transform-text {
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid #994c8e;
  border-radius: 30px;
  padding: 9px 12px;
  background: #41213f;
  font-size: 10px;
  letter-spacing: .4px;
  color: #ffadd8;
  white-space: nowrap;
}

.transform-text span {
  font-size: 17px;
}

.transform-text b {
  border-left: 1px solid #985a95;
  padding-left: 8px;
  color: #e9c5e5;
  font-size: 9px;
}

.result-state {
  font-size: 10px;
  white-space: nowrap;
  color: #ff8dcc;
}

.buttons-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-top: 17px;
}

.buttons-row ion-button {
  margin: 0;
  height: 43px;
  font-size: 12px;
  font-weight: 800;
}

.copy-btn {
  --background: #f781c1;
  --background-hover: #ffa3d5;
  --color: #331229;
  --border-radius: 9px;
}

.reverse-btn {
  --background: #5e3873;
  --background-hover: #71478a;
  --color: #f7e8f9;
  --border-radius: 9px;
}

.copy-feedback {
  margin-top: 11px;
  color: #ffc0e3;
  font-size: 11px;
}

.visual-panel {
  margin-top: 20px;
}

.visual-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  margin-bottom: 21px;
}

.visual-badge {
  background: #452342;
  border: 1px solid #a65b95;
  border-radius: 9px;
  padding: 10px 12px;
  color: #ffacd7;
  font: bold 11px monospace;
  white-space: nowrap;
}

.trace-wrap {
  margin-bottom: 19px;
}

.trace-header {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
  color: #e6c6e6;
  font-size: 11px;
  margin-bottom: 11px;
}

.trace-header span {
  color: #af98bd;
}

.trace-list {
  display: flex;
  gap: 7px;
  flex-wrap: wrap;
}

.trace-letter {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 47px;
  gap: 4px;
  padding: 8px 9px;
  border-radius: 9px;
  border: 1px solid #704a78;
  background: #22172f;
  color: #edd7ef;
  cursor: pointer;
}

.trace-letter.focused {
  background: #713365;
  border-color: #ff9bca;
  box-shadow: 0 0 0 1px #ff9bca55;
}

.trace-letter small {
  color: #bc9ac1;
  font-size: 9px;
}

.trace-letter strong {
  font-size: 13px;
}

.trace-letter span {
  font-size: 12px;
  color: #ff9dd2;
}

.equation-box {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
  background: #160f25;
  border: 1px solid #704677;
  border-radius: 11px;
  padding: 13px;
  margin-bottom: 18px;
}

.equation-box>div {
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 4px;
}

.equation-box small {
  color: #b99bc4;
  font-size: 9px;
  letter-spacing: .7px;
}

.equation-box strong {
  color: #f5e0f8;
  font: bold 15px monospace;
}

.equation-box .equation-answer strong {
  color: #ff8fcf;
}

.math-sign {
  color: #e381bd;
  font-size: 13px;
}

.matrix-scroll {
  overflow: auto;
  max-height: 640px;
  border: 1px solid #664572;
  border-radius: 10px;
  background: #1b122a;
}

.cipher-matrix {
  border-collapse: separate;
  border-spacing: 2px;
  min-width: 920px;
  width: 100%;
  table-layout: fixed;
  font-family: Consolas, 'Courier New', monospace;
  font-size: 11px;
  text-align: center;
}

.cipher-matrix th,
.cipher-matrix td {
  height: 26px;
  min-width: 29px;
  padding: 3px 0;
  border-radius: 3px;
  color: #d3bedf;
  background: #281b39;
}

.cipher-matrix thead th {
  position: sticky;
  top: 0;
  z-index: 2;
  color: #ffd2ee;
  background: #51335e;
}

.cipher-matrix tbody th {
  position: sticky;
  left: 0;
  z-index: 1;
  color: #ffd2ee;
  background: #51335e;
}

.cipher-matrix .matrix-corner {
  position: sticky;
  top: 0;
  left: 0;
  z-index: 4;
  min-width: 60px;
  width: 60px;
  color: #ffd5eb;
  background: #744269;
  font: bold 9px/1.1 'Segoe UI', sans-serif;
}

.cipher-matrix .matrix-row-highlight {
  background: #663253;
  color: #ffc6e5;
}

.cipher-matrix .matrix-col-highlight {
  background: #4b397a;
  color: #e8d6ff;
}

.cipher-matrix .matrix-row-active {
  background: #c05391;
  color: white;
}

.cipher-matrix .matrix-col-active {
  background: #8d66d0;
  color: white;
}

.cipher-matrix .matrix-intersection {
  background: #ffc5e4;
  color: #3a1532;
  font-weight: 900;
  outline: 2px solid #ff7abc;
  outline-offset: -1px;
}

.matrix-legend {
  display: flex;
  gap: 18px;
  margin-top: 15px;
  flex-wrap: wrap;
}

.matrix-legend span {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #c7afd3;
  font-size: 10px;
}

.matrix-legend i {
  width: 10px;
  height: 10px;
  display: inline-block;
  border-radius: 2px;
}

.legend-pink {
  background: #c05391;
}

.legend-violet {
  background: #8d66d0;
}

.legend-gold {
  background: #ffc5e4;
}

.matrix-note {
  color: #b9a4c9;
  margin: 15px 0 0;
  font-size: 11px;
  line-height: 1.8;
}

.caesar-scroll {
  overflow-x: auto;
}

.caesar-grid {
  display: grid;
  grid-template-columns: repeat(26, minmax(0, 1fr));
  gap: 5px;
  min-width: 850px;
}

.caesar-letter {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  font-family: monospace;
}

.caesar-letter span,
.caesar-letter b {
  display: grid;
  place-items: center;
  width: 100%;
  height: 35px;
  border-radius: 5px;
}

.caesar-letter span {
  background: #382548;
  color: #e9d4f0;
}

.caesar-letter em {
  color: #b876b7;
  font-style: normal;
}

.caesar-letter b {
  background: #73365f;
  color: #ffc0e3;
}

.page-footer {
  margin-top: 22px;
  padding: 28px 0 35px;
  border-top: 1px solid #493050;
  display: flex;
  justify-content: space-between;
  gap: 15px;
  align-items: center;
}

.page-footer b {
  color: #ff91cc;
  font-size: 12px;
}

.page-footer span,
.page-footer p {
  color: #a58db7;
  font-size: 10px;
}

.page-footer p {
  text-align: right;
}

button:focus-visible,
input:focus-visible,
textarea:focus-visible {
  outline: 2px solid #ffafd9;
  outline-offset: 2px;
}

@media (max-width: 900px) {
  .hero {
    grid-template-columns: 1fr;
  }

  .hero-art {
    display: none;
  }

  .work-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 600px) {
  .page-wrap {
    padding: 0 12px;
  }

  .nav-inner {
    padding: 0 12px;
    height: 67px;
  }

  .nav-status {
    font-size: 0;
  }

  .nav-version {
    display: none;
  }

  .logo-copy b {
    font-size: 17px;
  }

  .logo-copy small {
    font-size: 8px;
  }

  .hero {
    padding: 35px 0 25px;
    min-height: unset;
  }

  .hero h1 {
    font-size: 35px;
  }

  .hero p {
    font-size: 13px;
  }

  .glass-panel {
    padding: 16px;
  }

  .panel-head h2,
  .visual-head h2 {
    font-size: 19px;
  }

  .visual-head {
    flex-direction: column;
    align-items: flex-start;
  }

  .trace-letter {
    min-width: 42px;
  }

  .buttons-row {
    grid-template-columns: 1fr;
  }

  .page-footer {
    flex-direction: column;
    align-items: flex-start;
  }

  .page-footer p {
    text-align: left;
  }

  .equation-box {
    gap: 6px;
  }

  .equation-box small {
    font-size: 8px;
  }
}
</style>
