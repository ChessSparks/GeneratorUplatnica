<template>
  <transition name="splash-fade">
    <div v-if="loading" class="splash">
      <div class="splash-card">
        <div class="splash-logo">
          <div class="splash-icon">
            <span class="splash-icon-doc">📄</span>
            <span class="splash-icon-badge">HRK</span>
          </div>
        </div>
        <h1 class="splash-title">Generator<br>Uplatnica</h1>
        <p class="splash-sub">HUB-3A · PDF417 barkod · CSV uvoz</p>
        <div class="splash-bar"><div class="splash-bar-fill"></div></div>
      </div>
    </div>
  </transition>

  <div id="app">
    <header class="app-header">
      <div class="app-brand">
        <div class="app-brand-icon">🧾</div>
        <div class="app-brand-text">
          <span class="app-brand-title">Generator Uplatnica</span>
          <span class="app-brand-sub">HUB-3A · PDF417</span>
        </div>
      </div>
      <nav class="tabs">
        <button
          class="tab-btn"
          :class="{ active: activeTab === 'single' }"
          @click="switchTab('single')"
        >
          Jedna uplatnica
        </button>
        <button
          class="tab-btn"
          :class="{ active: activeTab === 'csv' }"
          @click="switchTab('csv')"
        >
          CSV uvoz
        </button>
      </nav>
      <a class="bmc-btn" href="https://buymeacoffee.com/sparkschess" target="_blank" rel="noopener">
        ☕ Buy me a coffee
      </a>
    </header>

    <main class="app-main">
      <!-- Single slip -->
      <div v-if="activeTab === 'single'">
        <SlipForm @generate="onSingleGenerate" />
        <template v-if="singleSlip">
          <div class="preview-bar">
            <span class="preview-label">Pregled uplatnice</span>
            <button class="btn-print" @click="printSlips">🖨 Ispiši / Spremi PDF</button>
          </div>
          <div class="print-area">
            <PaymentSlip :userData="singleSlip" @barcode-generated="onBarcodeGenerated" />
          </div>
        </template>
      </div>

      <!-- CSV slips -->
      <div v-if="activeTab === 'csv'">
        <CsvUpload @slips-loaded="onCsvLoaded" />
        <template v-if="csvSlips.length > 0">
          <div class="preview-bar">
            <span class="preview-label">{{ csvSlips.length }} uplatnica(e) učitano</span>
            <button class="btn-print" @click="printSlips">🖨 Ispiši sve / Spremi PDF</button>
          </div>
          <div class="print-area">
            <PaymentSlip
              v-for="(slip, index) in csvSlips"
              :key="index"
              :userData="slip"
              @barcode-generated="onBarcodeGenerated"
            />
          </div>
        </template>
      </div>
    </main>

  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import SlipForm from '@/components/SlipForm.vue';
import CsvUpload from '@/components/CsvUpload.vue';
import PaymentSlip from '@/components/PaymentSlip.vue';

export default {
  name: 'App',
  components: { SlipForm, CsvUpload, PaymentSlip },
  setup() {
    const loading = ref(true);
    const activeTab = ref('single');
    const singleSlip = ref(null);
    const csvSlips = ref([]);

    onMounted(() => {
      setTimeout(() => { loading.value = false; }, 3000);
    });

    function switchTab(tab) {
      activeTab.value = tab;
    }

    function onSingleGenerate(data) {
      singleSlip.value = data;
    }

    function onCsvLoaded(slips) {
      csvSlips.value = slips;
    }

    function onBarcodeGenerated(barcodeText) {
      console.log('Barcode generated:', barcodeText.substring(0, 40) + '...');
    }

    function printSlips() {
      window.print();
    }

    return { loading, activeTab, singleSlip, csvSlips, switchTab, onSingleGenerate, onCsvLoaded, onBarcodeGenerated, printSlips };
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

/* ── Reset ── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Inter', Arial, sans-serif;
  background: #e8eeff;
  background-image:
    radial-gradient(ellipse at 10% 30%, rgba(59, 108, 247, 0.13) 0%, transparent 50%),
    radial-gradient(ellipse at 90% 0%,  rgba(120, 80, 255, 0.10) 0%, transparent 50%),
    radial-gradient(ellipse at 60% 90%, rgba(30, 200, 180, 0.07) 0%, transparent 50%);
  min-height: 100vh;
  color: #1a2035;
  -webkit-font-smoothing: antialiased;
}

#app { min-height: 100vh; }

/* ── Splash ── */
.splash {
  position: fixed;
  inset: 0;
  background: linear-gradient(145deg, #080e2a 0%, #0d1f50 50%, #0a1530 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.splash-card { text-align: center; color: white; user-select: none; }
.splash-logo { margin-bottom: 1.5rem; }

.splash-icon {
  position: relative;
  display: inline-block;
  font-size: 5.5rem;
  line-height: 1;
  filter: drop-shadow(0 8px 24px rgba(59,108,247,0.5));
}

.splash-icon-badge {
  position: absolute;
  bottom: 8px;
  right: -16px;
  background: linear-gradient(135deg, #f0c040, #f5d060);
  color: #1a1a1a;
  font-size: 0.6rem;
  font-weight: 900;
  padding: 3px 6px;
  border-radius: 5px;
  letter-spacing: 0.5px;
  box-shadow: 0 2px 8px rgba(240,192,64,0.5);
}

.splash-title {
  font-size: 2.4rem;
  font-weight: 800;
  letter-spacing: -0.5px;
  line-height: 1.15;
  margin-bottom: 0.6rem;
  background: linear-gradient(135deg, #ffffff 30%, rgba(255,255,255,0.7));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.splash-sub {
  font-size: 0.72rem;
  color: rgba(255,255,255,0.4);
  letter-spacing: 2px;
  text-transform: uppercase;
  margin-bottom: 2.25rem;
}

.splash-bar {
  width: 200px;
  height: 3px;
  background: rgba(255,255,255,0.1);
  border-radius: 999px;
  margin: 0 auto;
  overflow: hidden;
}

.splash-bar-fill {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, #3b6cf7, #f0c040);
  border-radius: 999px;
  animation: splash-progress 2.8s ease-out forwards;
}

@keyframes splash-progress {
  0%   { width: 0%; }
  60%  { width: 75%; }
  100% { width: 100%; }
}

.splash-fade-leave-active { transition: opacity 0.5s ease; }
.splash-fade-leave-to { opacity: 0; }

/* ── Header ── */
.app-header {
  background: linear-gradient(135deg, #080f2e 0%, #0f2260 50%, #1a3a8f 100%);
  color: white;
  padding: 0.9rem 2.5rem;
  display: flex;
  align-items: center;
  gap: 2rem;
  box-shadow: 0 4px 32px rgba(8, 15, 46, 0.6), 0 1px 0 rgba(255,255,255,0.06) inset;
  border-bottom: 1px solid rgba(255,255,255,0.08);
  position: sticky;
  top: 0;
  z-index: 100;
}

.app-brand {
  display: flex;
  align-items: center;
  gap: 0.65rem;
  flex-shrink: 0;
}

.app-brand-icon {
  font-size: 1.6rem;
  line-height: 1;
  filter: drop-shadow(0 2px 6px rgba(255,255,255,0.2));
}

.app-brand-text {
  display: flex;
  flex-direction: column;
  line-height: 1.2;
}

.app-brand-title {
  font-size: 1.05rem;
  font-weight: 700;
  white-space: nowrap;
  letter-spacing: -0.2px;
}

.app-brand-sub {
  font-size: 0.62rem;
  color: rgba(255,255,255,0.45);
  letter-spacing: 1px;
  text-transform: uppercase;
}

/* ── Tabs ── */
.tabs {
  display: flex;
  background: rgba(255,255,255,0.1);
  border-radius: 999px;
  padding: 4px;
  gap: 2px;
}

.tab-btn {
  padding: 0.38rem 1.15rem;
  border: none;
  background: transparent;
  color: rgba(255,255,255,0.65);
  border-radius: 999px;
  cursor: pointer;
  font-size: 0.85rem;
  font-family: inherit;
  font-weight: 500;
  transition: all 0.2s;
}

.tab-btn:hover { color: white; background: rgba(255,255,255,0.12); }

.tab-btn.active {
  background: white;
  color: #1a3a7c;
  font-weight: 700;
  box-shadow: 0 2px 10px rgba(0,0,0,0.18);
}

/* ── BMC button ── */
.bmc-btn {
  margin-left: auto;
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.45rem 1.1rem;
  background: linear-gradient(135deg, #f5c842 0%, #fde27a 100%);
  color: #1a1200;
  border-radius: 999px;
  font-size: 0.8rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  white-space: nowrap;
  box-shadow: 0 3px 14px rgba(240,192,64,0.5), 0 1px 0 rgba(255,255,255,0.4) inset;
  transition: transform 0.15s, box-shadow 0.15s;
}

.bmc-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 22px rgba(240,192,64,0.65), 0 1px 0 rgba(255,255,255,0.4) inset;
}

/* ── Main layout ── */
.app-main {
  max-width: 1060px;
  margin: 2rem auto;
  padding: 0 1.5rem;
}

/* ── Preview bar ── */
.preview-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 1.5rem 0 0.85rem;
  background: rgba(255,255,255,0.75);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 14px;
  padding: 0.85rem 1.25rem;
  box-shadow: 0 4px 24px rgba(14,30,80,0.08), 0 1px 0 rgba(255,255,255,0.9) inset;
  border: 1px solid rgba(59,108,247,0.1);
}

.preview-label {
  font-weight: 600;
  font-size: 0.92rem;
  color: #1a2035;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.preview-label::before {
  content: '';
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #3b6cf7, #6b4ef7);
}

.btn-print {
  padding: 0.55rem 1.4rem;
  background: linear-gradient(135deg, #1740b8 0%, #3b6cf7 60%, #5d8aff 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 0.875rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 4px 16px rgba(30,74,200,0.4), 0 1px 0 rgba(255,255,255,0.2) inset;
  transition: transform 0.15s, box-shadow 0.15s;
}

.btn-print:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(30,74,200,0.5), 0 1px 0 rgba(255,255,255,0.2) inset;
}

.print-area {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  padding-bottom: 2.5rem;
}

/* ── Mobile ── */
@media (max-width: 640px) {
  .app-header {
    flex-wrap: wrap;
    padding: 0.75rem 1rem;
    gap: 0.5rem;
  }

  .app-brand { flex: 1; }

  .bmc-btn {
    margin-left: 0;
    font-size: 0.73rem;
    padding: 0.38rem 0.75rem;
  }

  .tabs {
    order: 3;
    width: 100%;
    justify-content: center;
  }

  .tab-btn { flex: 1; text-align: center; }

  .app-main {
    margin: 1rem auto;
    padding: 0 0.75rem;
  }

  .preview-bar {
    flex-direction: column;
    align-items: stretch;
    gap: 0.65rem;
  }

  .btn-print { width: 100%; text-align: center; }
}

/* ── Print ── */
@media print {
  .app-header, .preview-bar, .slip-form, .csv-upload { display: none !important; }
  body { background: white; }
  .app-main { margin: 0; padding: 0; max-width: none; }
  .print-area { gap: 0; }
  .payment-slip { page-break-after: always; break-after: page; }
  .payment-slip:last-child { page-break-after: avoid; break-after: avoid; }
}
</style>
