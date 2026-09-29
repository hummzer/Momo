<script setup>
import { computed, ref } from 'vue'

const languages = ['MQL5', 'MQL4', 'Pine Script', 'Python', 'Rust']
const search = ref('')
const type = ref('all')
const language = ref('all')
const selected = ref(null)

const WHATSAPP_NUMBER = '254716475923'
function whatsappLink(systemName) {
  const text = systemName
    ? 'Hi, I\'d like to order a custom build of "' + systemName + '" from Momo.'
    : "Hi, I'd like to talk about a custom EA/indicator build."
  return 'https://wa.me/' + WHATSAPP_NUMBER + '?text=' + encodeURIComponent(text)
}

const metricFields = [
  ['History Quality', 'historyQuality'],
  ['Bars', 'bars'],
  ['Ticks', 'ticks'],
  ['Symbols', 'symbols'],
  ['Initial Deposit', 'initialDeposit'],
  ['Withdrawal', 'withdrawal'],
  ['Net Profit', 'netProfit'],
  ['Gross Profit / Loss', 'grossProfitLoss'],
  ['Balance Drawdown', 'balanceDrawdown'],
  ['Equity Drawdown', 'equityDrawdown'],
  ['Profit Factor', 'profitFactor'],
  ['Recovery Factor', 'recoveryFactor'],
  ['Expected Payoff', 'expectedPayoff'],
  ['Sharpe Ratio', 'sharpe'],
  ['AHPR / GHPR', 'ahprGhpr'],
  ['LR Correlation', 'lrCorrelation'],
  ['LR Standard Error', 'lrStdError'],
  ['Margin Level', 'marginLevel'],
  ['Z-Score', 'zScore'],
  ['OnTester', 'onTester'],
  ['Total Trades / Deals', 'totalTrades'],
  ['Long / Short Stats', 'longShort'],
  ['Profit/Loss Trade Stats', 'plTradeStats'],
  ['Largest / Average Trades', 'largestAvgTrades'],
  ['Consecutive Wins / Losses', 'consecutiveWinsLosses'],
  ['Holding Time', 'holdingTime'],
  ['MFE/MAE Correlation', 'mfeMae'],
]

/** SVG poster when no real screenshot is uploaded yet */
function posterSvg(id, name, kind) {
  const accent = kind === 'strategy' ? '#b40018' : '#96793f'
  const label = String(id).padStart(2, '0')
  const safe = name.replace(/[<>&"']/g, '')
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" width="800" height="450" viewBox="0 0 800 450">
  <defs>
    <linearGradient id="g" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#141210"/>
      <stop offset="100%" stop-color="#2a2420"/>
    </linearGradient>
  </defs>
  <rect width="800" height="450" fill="url(#g)"/>
  <path d="M40 320 L120 280 L200 300 L280 220 L360 250 L440 180 L520 210 L600 140 L680 170 L760 120"
        fill="none" stroke="${accent}" stroke-width="3" opacity="0.85"/>
  <path d="M40 340 L120 310 L200 330 L280 260 L360 290 L440 230 L520 260 L600 200 L680 230 L760 180 L760 400 L40 400 Z"
        fill="${accent}" opacity="0.12"/>
  <text x="40" y="48" fill="#ffffff" font-family="ui-monospace,monospace" font-size="14" letter-spacing="4">MOMO · ${label}</text>
  <text x="40" y="90" fill="#f5f3ef" font-family="system-ui,sans-serif" font-size="28" font-weight="700">${safe.slice(0, 36)}</text>
  <text x="40" y="420" fill="#8a8880" font-family="ui-monospace,monospace" font-size="12" letter-spacing="2">${kind.toUpperCase()} · CHART PREVIEW</text>
</svg>`
  return 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(svg)
}

function youtubeId(url) {
  if (!url) return null
  const m = String(url).match(/(?:youtu\.be\/|v=|embed\/)([\w-]{11})/)
  return m ? m[1] : null
}

function isDirectVideo(url) {
  if (!url) return false
  return /\.(mp4|webm|ogg)(\?|$)/i.test(url) || url.startsWith('blob:')
}

const strategies = [
  ['Opening Range Breakout', 'Tracks a designated London or New York opening range. A confirmed close beyond the range triggers directional entry.', '15/30m window · breakout close', 'SL opposite extreme or midpoint'],
  ['Supply & Demand Zone Rejection', 'Detects base zones before sharp momentum extensions and seeks confirmed pullbacks into unmitigated zones.', 'Zone detection · confirmation candle', 'Zone invalidation / structural target'],
  ['Moving Average Cross + Trend Filter', 'Fast EMA crosses the slow EMA while a macro condition such as 200 SMA slope or ADX validates direction.', 'EMA 9/20 vs 50/200 · ADX option', 'Trend-filtered crossover'],
  ['Bollinger Band Mean Reversion', 'Fades closes outside the outer Bollinger Band when RSI or Stochastic confirms an overextended state.', 'BB(20,2) · RSI/Stoch filter', 'Return toward SMA20'],
  ['Donchian Channel Breakout', 'Turtle-style breakout using the highest high and lowest low over a rolling N-bar channel.', '20-bar default', 'Opposite shorter channel / exit rule'],
  ['MACD Zero-Line Momentum', 'Requires a MACD zero-line crossover plus increasing histogram momentum for at least two bars.', 'MACD · histogram acceleration', 'Momentum invalidation / structural exit'],
  ['Fibonacci Retracement Pullback', 'Maps a recent swing high-low and looks for reversal inside the 61.8–78.6% retracement area.', '61.8/78.6% zone', 'Prior structural extreme'],
  ['SuperTrend MTF Trend Following', 'ATR-backed SuperTrend flips only on confirmed closes, with higher-timeframe trend context available.', 'ATR SuperTrend · MTF', 'Dynamic SuperTrend stop'],
  ['VWAP Session Reversion', 'Uses session VWAP and standard-deviation bands to fade extreme deviations back toward fair value.', 'VWAP · ±1/±2σ', 'VWAP mean reversion'],
  ['BOS & ChoCH Structure Engine', 'Tracks confirmed swing pivots. Minor breaks classify continuation while major counter-trend breaks flag structural reversal.', 'Swing pivots · BOS · ChoCH', 'Structural invalidation / liquidity'],
  ['RSI Divergence Reversal', 'Identifies price higher-high / RSI lower-high or inverse divergence, then waits for a countertrend swing break.', 'RSI divergence · swing break', 'Countertrend structure'],
  ['ATR Breakout Volatility Expansion', 'Detects compressed ranges relative to rolling ATR and places conditional breakout orders beyond the compression bar.', 'ATR compression threshold', 'Opposite side / volatility stop'],
]

const indicators = [
  ['Adaptive RSI-EMA (ARESI)', 'Smooths RSI through an EMA whose effective period scales with normalized ATR, filtering chop while adapting to expansion.', 'RSI · adaptive EMA · ATR normalization'],
  ['Volumetric ATR Envelope (V-ATRE)', 'Multiplies ATR by a volume-intensity factor to create adaptive support and resistance envelopes.', 'ATR · tick/real volume'],
  ['Stochastic Momentum Z-Score (SMZ)', 'Normalizes stochastic momentum into rolling Z-scores to express deviation from its recent distribution.', 'Stochastic · rolling Z-score'],
  ['MACD-SuperTrend Hybrid Filter (MSTF)', 'Requires MACD histogram acceleration and a volatility SuperTrend regime to align before producing a signal.', 'MACD histogram · SuperTrend'],
  ['Multi-Timeframe Hull Trend Ribbon (MTF-HR)', 'Four HMA lengths with recursive timeframe scaling provide a non-repainting short/intermediate/long trend ribbon.', 'HMA · MTF recursion'],
  ['Bollinger-RSI Momentum Squeeze (BRMS)', 'Combines BB width, Keltner/ATR containment and RSI proximity to 50 for a quantified squeeze warning.', 'Bollinger · Keltner · RSI'],
  ['Volume-Weighted ATR Stop (VW-ATRS)', 'Adjusts ATR trailing distance using volume accumulation: tighter in heavy participation and wider in thin liquidity.', 'ATR stop · volume weighting'],
  ['Fractal-Chande Momentum Oscillator (F-CMO)', 'Validates CMO momentum against confirmed Bill Williams fractal turns to reduce unconfirmed reversals.', 'Fractals · CMO'],
  ['Parabolic SAR Volume Intensity Index (PSAR-VII)', 'Scales PSAR acceleration using relative volume spikes to make the stop responsive to participation.', 'PSAR · relative volume'],
  ['Composite Trend Exhaustion Index (CTEI)', 'Combines ROC, normalized CCI and distance from 200 EMA into a -100 to +100 exhaustion oscillator.', 'ROC · CCI · EMA200'],
]

/**
 * Media fields:
 * - imageUrl: chart / EA screenshot (path under /media/ or absolute URL)
 * - videoUrl: YouTube link or direct .mp4/.webm walkthrough
 * Drop real files in public/media/ and set paths like '/media/01-orb.jpg'
 */
const systems = [
  ...strategies.map((x, i) => {
    const id = i + 1
    return {
      id,
      name: x[0],
      type: 'strategy',
      description: x[1],
      logic: x[2],
      exit: x[3],
      priceFrom: 89,
      metrics: null,
      imageUrl: `/media/${String(id).padStart(2, '0')}.jpg`,
      videoUrl: null,
      _fallback: posterSvg(id, x[0], 'strategy'),
    }
  }),
  ...indicators.map((x, i) => {
    const id = i + 13
    return {
      id,
      name: x[0],
      type: 'indicator',
      description: x[1],
      logic: x[2],
      exit: 'Signal / research instrument',
      priceFrom: 39,
      metrics: null,
      imageUrl: `/media/${String(id).padStart(2, '0')}.jpg`,
      videoUrl: null,
      _fallback: posterSvg(id, x[0], 'indicator'),
    }
  }),
]

const filtered = computed(() =>
  systems.filter((s) => {
    const q = search.value.trim().toLowerCase()
    return (
      (type.value === 'all' || s.type === type.value) &&
      (language.value === 'all' || languages.includes(language.value)) &&
      (!q || [s.name, s.description, s.logic].join(' ').toLowerCase().includes(q))
    )
  }),
)

function setTypeFilter(t) {
  type.value = t
  document.getElementById('archive')?.scrollIntoView({ behavior: 'smooth' })
}

function open(system) {
  selected.value = system
}
function close() {
  selected.value = null
}

function onImgError(e, system) {
  e.target.src = system._fallback
}
</script>

<template>
  <div class="app">
    <header class="nav">
      <div class="nav-inner">
        <div class="brand">MOMO<span></span></div>
        <nav>
          <a href="#archive" @click.prevent="setTypeFilter('all')">Archive</a>
          <a href="#archive" @click.prevent="setTypeFilter('strategy')">Strategies</a>
          <a href="#archive" @click.prevent="setTypeFilter('indicator')">Indicators</a>
        </nav>
      </div>
    </header>

    <main>
      <section class="intro">
        <div class="intro-inner">
          <div class="eyebrow">TRADING SYSTEMS ARCHIVE</div>
          <h1>Systematic trading work, built to spec.</h1>
          <p>
            22 catalogued systems across MQL5, MQL4, Pine Script, Python and Rust. Every build ships with a full
            verified backtest report before delivery — no fabricated performance claims.
          </p>
          <div class="intro-actions">
            <a class="btn-primary" href="#archive">Browse the archive</a>
            <a class="btn-ghost" :href="whatsappLink()" target="_blank" rel="noopener">Request a custom build</a>
          </div>
        </div>
      </section>

      <section id="archive" class="archive">
        <div class="filters">
          <input v-model="search" placeholder="Search the archive…" aria-label="Search archive" />
          <select v-model="type">
            <option value="all">All systems</option>
            <option value="strategy">Strategies</option>
            <option value="indicator">Indicators</option>
          </select>
          <select v-model="language">
            <option value="all">All languages</option>
            <option v-for="item in languages" :key="item" :value="item">{{ item }}</option>
          </select>
          <div class="count">{{ filtered.length }} / 22 SHOWN</div>
        </div>

        <div class="section-head">
          <div>
            <div class="eyebrow">THE ARCHIVE</div>
            <h2>Systems & instruments</h2>
          </div>
          <p>12 strategies · 10 indicators</p>
        </div>

        <div class="grid">
          <article v-for="system in filtered" :key="system.id" class="card" @click="open(system)">
            <div class="card-media">
              <img
                :src="system.imageUrl"
                :alt="system.name + ' preview'"
                loading="lazy"
                @error="onImgError($event, system)"
              />
              <span v-if="system.videoUrl" class="play-badge" aria-hidden="true">▶</span>
              <span class="media-tag" :class="{ red: system.type === 'strategy' }">{{ system.type }}</span>
            </div>
            <div class="card-body">
              <div class="card-top">
                <span class="number">{{ String(system.id).padStart(2, '0') }}</span>
              </div>
              <h3>{{ system.name }}</h3>
              <p>{{ system.description }}</p>
              <div class="pills">
                <span v-for="lang in languages" :key="lang">{{ lang }}</span>
              </div>
              <div class="card-foot">
                <span class="price">FROM ${{ system.priceFrom }}</span>
                <span>{{ system.videoUrl ? 'VIDEO + SPEC' : 'SPECIFICATION / BUILD' }}</span>
              </div>
            </div>
          </article>
        </div>

        <div v-if="!filtered.length" class="empty">No systems match the current filters.</div>
      </section>
    </main>

    <div v-if="selected" class="modal" @click.self="close">
      <aside class="drawer">
        <button class="close" type="button" @click="close" aria-label="Close">×</button>

        <div class="drawer-media">
          <img
            class="drawer-image"
            :src="selected.imageUrl"
            :alt="selected.name"
            @error="onImgError($event, selected)"
          />

          <div v-if="selected.videoUrl && youtubeId(selected.videoUrl)" class="video-frame">
            <iframe
              :src="'https://www.youtube.com/embed/' + youtubeId(selected.videoUrl)"
              title="Backtest walkthrough"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen
            />
          </div>
          <div v-else-if="selected.videoUrl && isDirectVideo(selected.videoUrl)" class="video-frame">
            <video :src="selected.videoUrl" controls playsinline preload="metadata" />
          </div>
          <div v-else class="video-frame empty-video">
            <div class="empty-video-inner">
              <span class="empty-ico">▶</span>
              <p>Backtest walkthrough video appears here once you set <code>videoUrl</code> (YouTube or .mp4).</p>
            </div>
          </div>
        </div>

        <div class="eyebrow">{{ selected.type }} · {{ String(selected.id).padStart(2, '0') }}</div>
        <h2>{{ selected.name }}</h2>
        <p class="drawer-copy">{{ selected.description }}</p>

        <div class="rule"></div>

        <div class="details">
          <div>
            <b>Languages</b>
            <span>All five planned</span>
          </div>
          <div>
            <b>Logic</b>
            <span>{{ selected.logic }}</span>
          </div>
          <div>
            <b>Exit / use</b>
            <span>{{ selected.exit }}</span>
          </div>
          <div>
            <b>Repainting</b>
            <span>Closed-bar design</span>
          </div>
          <div>
            <b>Pairs</b>
            <span>Instrument-specific configuration</span>
          </div>
          <div>
            <b>Price</b>
            <span>From ${{ selected.priceFrom }} · final quote after spec review</span>
          </div>
        </div>

        <div class="rule"></div>

        <div class="backtest-head">
          <div class="eyebrow">MT5 BACKTEST REPORT</div>
          <h3>Performance metrics</h3>
        </div>

        <div v-if="selected.metrics" class="details metrics-grid">
          <div v-for="[label, key] in metricFields" :key="key">
            <b>{{ label }}</b>
            <span>{{ selected.metrics[key] ?? '—' }}</span>
          </div>
        </div>
        <div v-else class="notice">
          No verified backtest yet for this system. Metrics populate once a real MT5 Strategy Tester report is
          attached. No fabricated performance statistics.
        </div>

        <a class="btn-primary full" :href="whatsappLink(selected.name)" target="_blank" rel="noopener">
          Order this build on WhatsApp
        </a>
      </aside>
    </div>

    <footer>
      <div class="footer-inner">
        <strong>MOMO</strong>
        <span>MQL5 · MQL4 · PINE · PYTHON · RUST</span>
      </div>
    </footer>
  </div>
</template>
