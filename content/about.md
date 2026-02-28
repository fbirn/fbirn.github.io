---
title: "whoami"
draft: false
ShowToc: false
ShowReadingTime: false
ShowBreadCrumbs: false
hideAuthor: true
---

<div style="text-align: center;">
  <img src="/images/fbirn.jfif" alt="Fabio" style="border-radius: 50%; width: 180px; height: 180px; object-fit: cover;">
</div>


<style>

.term-wrap {
  font-family: 'Courier New', Courier, monospace;
  background: #0d0d0d;
  border: 1px solid #00ff41;
  border-radius: 8px;
  padding: 2rem;
  max-width: 720px;
  margin: 2rem auto;
  box-shadow: 0 0 30px rgba(0,255,65,0.15);
  color: #00ff41;
}

.term-titlebar {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 1.5rem;
  padding-bottom: 0.8rem;
  border-bottom: 1px solid #1a3a1a;
}

.term-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}
.term-dot.red   { background: #ff5f57; }
.term-dot.yellow{ background: #ffbd2e; }
.term-dot.green { background: #28ca41; }

.term-title {
  flex: 1;
  text-align: center;
  font-size: 0.75rem;
  color: #555;
}

.term-line {
  margin: 0.25rem 0;
  line-height: 1.7;
  font-size: 0.92rem;
}

.prompt-user { color: #00ff41; font-weight: bold; }
.prompt-at   { color: #ffffff; }
.prompt-host { color: #00bfff; font-weight: bold; }
.prompt-path { color: #888; }
.prompt-dollar { color: #ffffff; }

.key   { color: #00bfff; }
.val   { color: #ffffff; }
.dim   { color: #444; }
.ascii { color: #00ff41; line-height: 1.3; white-space: pre; font-size: 0.78rem; }

.blink {
  animation: blink 1s step-end infinite;
}
@keyframes blink {
  50% { opacity: 0; }
}

.badge {
  display: inline-block;
  background: #001a00;
  border: 1px solid #00ff41;
  border-radius: 4px;
  padding: 0 6px;
  font-size: 0.78rem;
  margin: 2px 2px 2px 0;
  color: #00ff41;
}

.section-gap { margin-top: 1rem; }
</style>

<div class="term-wrap">

  <div class="term-titlebar">
    <div class="term-dot red"></div>
    <div class="term-dot yellow"></div>
    <div class="term-dot green"></div>
    <div class="term-title">fbirn@kali: ~</div>
  </div>

<!-- NEU: -->
<img src="/images/fbirn-ascii.svg" alt="FBIRN" style="max-width:100%; margin-bottom: 0.5rem;">

  <div class="term-line dim">─────────────────────────────────────────────</div>

  <div class="term-line section-gap">
    <span class="prompt-user">fabio</span><span class="prompt-at">@</span><span class="prompt-host">kali</span><span class="prompt-path"> ~/about</span><span class="prompt-dollar"> $</span> whoami
  </div>

  <div class="term-line">
    <span class="key">  user         </span><span class="dim">→</span> <span class="val">Fabio Birnegger</span>
  </div>
  <div class="term-line">
    <span class="key">  age          </span><span class="dim">→</span> <span class="val">27</span>
  </div>
  <div class="term-line">
    <span class="key">  location     </span><span class="dim">→</span> <span class="val">Vienna, Austria 🇦🇹</span>
  </div>

  <div class="term-line section-gap">
    <span class="prompt-user">fabio</span><span class="prompt-at">@</span><span class="prompt-host">kali</span><span class="prompt-path"> ~/about</span><span class="prompt-dollar"> $</span> cat profession.txt
  </div>

  <div class="term-line">
    <span class="key">  role         </span><span class="dim">→</span> <span class="val">Penetration Tester</span>
  </div>
  <div class="term-line">
    <span class="key">  employer     </span><span class="dim">→</span> <span class="val">TÜV AUSTRIA</span>
  </div>
  <div class="term-line">
    <span class="key">  education    </span><span class="dim">→</span> <span class="val">MSc IT-Security</span>
  </div>
  <div class="term-line">
    <span class="key">  university   </span><span class="dim">→</span> <span class="val">Hochschule Campus Wien</span>
  </div>

  <div class="term-line section-gap">
    <span class="prompt-user">fabio</span><span class="prompt-at">@</span><span class="prompt-host">kali</span><span class="prompt-path"> ~/about</span><span class="prompt-dollar"> $</span> cat skills.json
  </div>

  <div class="term-line">
    <span class="dim">{</span>
  </div>
  <div class="term-line">
    &nbsp;&nbsp;<span class="key">"offensive_security"</span><span class="dim">:</span> <span class="val">["Web App Pentesting", "Network Pentesting", "Red Teaming"]</span><span class="dim">,</span>
  </div>
  <div class="term-line">
    &nbsp;&nbsp;<span class="key">"tools"</span><span class="dim">:</span> <span class="val">["Burp Suite", "Metasploit", "Nmap", "BloodHound", "Cobalt Strike"]</span>
  </div>
  <div class="term-line">
    <span class="dim">}</span>
  </div>

  <div class="term-line section-gap">
    <span class="prompt-user">fabio</span><span class="prompt-at">@</span><span class="prompt-host">kali</span><span class="prompt-path"> ~/about</span><span class="prompt-dollar"> $</span> ls hobbies/
  </div>

  <div class="term-line">
    <span class="badge">⚔️ Hacking</span>
    <span class="badge">🎮 Gaming</span>
    <span class="badge">🏋️ Kraftsport</span>
    <span class="badge">🚩 CTFs</span>
  </div>

  <div class="term-line section-gap">
    <span class="prompt-user">fabio</span><span class="prompt-at">@</span><span class="prompt-host">kali</span><span class="prompt-path"> ~/about</span><span class="prompt-dollar"> $</span> <span class="blink">█</span>
  </div>

</div>


