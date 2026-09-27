<script setup>
import { computed, ref } from 'vue'

const languages = ['MQL5', 'MQL4', 'Pine Script', 'Python', 'Rust']
const search = ref('')
const type = ref('all')
const language = ref('all')
const selected = ref(null)

const strategies = [
  ['Opening Range Breakout','Tracks a designated London or New York opening range. A confirmed close beyond the range triggers directional entry.','15/30m window · breakout close','SL opposite extreme or midpoint'],
  ['Supply & Demand Zone Rejection','Detects base zones before sharp momentum extensions and seeks confirmed pullbacks into unmitigated zones.','Zone detection · confirmation candle','Zone invalidation / structural target'],
  ['Moving Average Cross + Trend Filter','Fast EMA crosses the slow EMA while a macro condition such as 200 SMA slope or ADX validates direction.','EMA 9/20 vs 50/200 · ADX option','Trend-filtered crossover'],
  ['Bollinger Band Mean Reversion','Fades closes outside the outer Bollinger Band when RSI or Stochastic confirms an overextended state.','BB(20,2) · RSI/Stoch filter','Return toward SMA20'],
  ['Donchian Channel Breakout','Turtle-style breakout using the highest high and lowest low over a rolling N-bar channel.','20-bar default','Opposite shorter channel / exit rule'],
  ['MACD Zero-Line Momentum','Requires a MACD zero-line crossover plus increasing histogram momentum for at least two bars.','MACD · histogram acceleration','Momentum invalidation / structural exit'],
  ['Fibonacci Retracement Pullback','Maps a recent swing high-low and looks for reversal inside the 61.8–78.6% retracement area.','61.8/78.6% zone','Prior structural extreme'],
  ['SuperTrend MTF Trend Following','ATR-backed SuperTrend flips only on confirmed closes, with higher-timeframe trend context available.','ATR SuperTrend · MTF','Dynamic SuperTrend stop'],
  ['VWAP Session Reversion','Uses session VWAP and standard-deviation bands to fade extreme deviations back toward fair value.','VWAP · ±1/±2σ','VWAP mean reversion'],
  ['BOS & ChoCH Structure Engine','Tracks confirmed swing pivots. Minor breaks classify continuation while major counter-trend breaks flag structural reversal.','Swing pivots · BOS · ChoCH','Structural invalidation / liquidity'],
  ['RSI Divergence Reversal','Identifies price higher-high / RSI lower-high or inverse divergence, then waits for a countertrend swing break.','RSI divergence · swing break','Countertrend structure'],
  ['ATR Breakout Volatility Expansion','Detects compressed ranges relative to rolling ATR and places conditional breakout orders beyond the compression bar.','ATR compression threshold','Opposite side / volatility stop']
]

const indicators = [
  ['Adaptive RSI-EMA (ARESI)','Smooths RSI through an EMA whose effective period scales with normalized ATR, filtering chop while adapting to expansion.','RSI · adaptive EMA · ATR normalization'],
  ['Volumetric ATR Envelope (V-ATRE)','Multiplies ATR by a volume-intensity factor to create adaptive support and resistance envelopes.','ATR · tick/real volume'],
  ['Stochastic Momentum Z-Score (SMZ)','Normalizes stochastic momentum into rolling Z-scores to express deviation from its recent distribution.','Stochastic · rolling Z-score'],
  ['MACD-SuperTrend Hybrid Filter (MSTF)','Requires MACD histogram acceleration and a volatility SuperTrend regime to align before producing a signal.','MACD histogram · SuperTrend'],
  ['Multi-Timeframe Hull Trend Ribbon (MTF-HR)','Four HMA lengths with recursive timeframe scaling provide a non-repainting short/intermediate/long trend ribbon.','HMA · MTF recursion'],
  ['Bollinger-RSI Momentum Squeeze (BRMS)','Combines BB width, Keltner/ATR containment and RSI proximity to 50 for a quantified squeeze warning.','Bollinger · Keltner · RSI'],
  ['Volume-Weighted ATR Stop (VW-ATRS)','Adjusts ATR trailing distance using volume accumulation: tighter in heavy participation and wider in thin liquidity.','ATR stop · volume weighting'],
  ['Fractal-Chande Momentum Oscillator (F-CMO)','Validates CMO momentum against confirmed Bill Williams fractal turns to reduce unconfirmed reversals.','Fractals · CMO'],
  ['Parabolic SAR Volume Intensity Index (PSAR-VII)','Scales PSAR acceleration using relative volume spikes to make the stop responsive to participation.','PSAR · relative volume'],
  ['Composite Trend Exhaustion Index (CTEI)','Combines ROC, normalized CCI and distance from 200 EMA into a -100 to +100 exhaustion oscillator.','ROC · CCI · EMA200']
]

const systems = [
  ...strategies.map((x,i) => ({ id:i+1, name:x[0], type:'strategy', description:x[1], logic:x[2], exit:x[3] })),
  ...indicators.map((x,i) => ({ id:i+13, name:x[0], type:'indicator', description:x[1], logic:x[2], exit:'Signal / research instrument' }))
]

const filtered = computed(() => systems.filter(s => {
  const q = search.value.trim().toLowerCase()
  return (type.value === 'all' || s.type === type.value) &&
    (!q || [s.name,s.description,s.logic].join(' ').toLowerCase().includes(q))
}))

function open(system) { selected.value = system }
function close() { selected.value = null }
</script>

<template>
  <div class="app">
    <header class="nav">
      <div class="nav-inner">
        <div class="brand">MOMO<span></span></div>
        <nav><a href="#archive">Archive</a><a href="#archive">Strategies</a><a href="#archive">Indicators</a></nav>
      </div>
    </header>

    <main>
      <section class="hero">
        <div class="hero-inner">
          <div class="eyebrow">PRIVATE TRADING SYSTEMS ARCHIVE · 2026</div>
          <h1>BUILD.<br>TEST.<br><em>DEPLOY.</em></h1>
          <p>Momo is the catalogue for systematic trading work across MQL5, MQL4, Pine Script, Python and Rust. Twenty-two core systems. Five implementation languages. No accounts. No noise.</p>
          <a class="hero-link" href="#archive">ENTER THE ARCHIVE <span>↓</span></a>
        </div>
        <div class="seal"><span>VALOR<br>DISCIPLINE<br>PRECISION</span></div>
      </section>

      <section id="archive" class="archive">
        <div class="filters">
          <input v-model="search" placeholder="Search the archive…" aria-label="Search archive">
          <select v-model="type"><option value="all">All systems</option><option value="strategy">Strategies</option><option value="indicator">Indicators</option></select>
          <select v-model="language"><option value="all">All languages</option><option v-for="item in languages" :key="item">{{ item }}</option></select>
          <div class="count">{{ filtered.length }} / 22 SHOWN</div>
        </div>

        <div class="section-head">
          <div><div class="eyebrow">THE ARCHIVE</div><h2>Systems & instruments</h2></div>
          <p>12 strategies · 10 indicators</p>
        </div>

        <div class="grid">
          <article v-for="system in filtered" :key="system.id" class="card" @click="open(system)">
            <div class="card-top">
              <span class="tag" :class="{red:system.type==='strategy'}">{{ system.type }}</span>
              <span class="number">{{ String(system.id).padStart(2,'0') }}</span>
            </div>
            <h3>{{ system.name }}</h3>
            <p>{{ system.description }}</p>
            <div class="pills"><span v-for="lang in languages" :key="lang">{{ lang }}</span></div>
            <div class="card-foot"><span class="price">FROM $—</span><span>SPECIFICATION / BUILD</span></div>
          </article>
        </div>

        <div v-if="!filtered.length" class="empty">No systems match the current filters.</div>
      </section>
    </main>

    <div v-if="selected" class="modal" @click.self="close">
      <aside class="drawer">
        <button class="close" @click="close">×</button>
        <div class="eyebrow">{{ selected.type }} · {{ String(selected.id).padStart(2,'0') }}</div>
        <h2>{{ selected.name }}</h2>
        <p class="drawer-copy">{{ selected.description }}</p>
        <div class="rule"></div>
        <div class="details">
          <div><b>Languages</b><span>All five planned</span></div>
          <div><b>Logic</b><span>{{ selected.logic }}</span></div>
          <div><b>Exit / use</b><span>{{ selected.exit }}</span></div>
          <div><b>Repainting</b><span>Closed-bar design</span></div>
          <div><b>Pairs</b><span>Instrument-specific configuration</span></div>
          <div><b>Price</b><span>Configured before release</span></div>
        </div>
        <div class="notice">No fabricated performance statistics. Verified backtest results can be added to each system as implementations are completed.</div>
      </aside>
    </div>

    <footer><div class="footer-inner"><strong>MOMO</strong><span>MQL5 · MQL4 · PINE · PYTHON · RUST</span></div></footer>
  </div>
</template>