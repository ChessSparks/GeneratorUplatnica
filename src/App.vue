<template>
  <transition name="splash-fade">
    <div v-if="loading" class="splash">
      <div class="splash-card">
        <div class="splash-logo">
          <div class="splash-mark"></div>
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
        <div class="app-brand-mark">GU</div>
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
        Buy me a coffee
      </a>
    </header>

    <main class="app-main">
      <!-- Single slip -->
      <div v-if="activeTab === 'single'">
        <SlipForm @generate="onSingleGenerate" />
        <template v-if="singleSlip">
          <div class="preview-bar">
            <span class="preview-label">Pregled uplatnice</span>
            <button class="btn-print" @click="printSlips">Ispiši / Spremi PDF</button>
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
            <div class="preview-actions">
              <button class="btn-zip" @click="downloadZip" :disabled="zipping">
                {{ zipping ? 'Generiranje...' : 'Preuzmi ZIP' }}
              </button>
              <button class="btn-print" @click="printSlips">Ispiši sve / Spremi PDF</button>
            </div>
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
    const zipping = ref(false);

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

    async function downloadZip() {
      if (!csvSlips.value.length || zipping.value) return;
      zipping.value = true;
      try {
        const [{ default: JSZip }, { default: html2canvas }] = await Promise.all([
          import('jszip'),
          import('html2canvas'),
        ]);

        const zip = new JSZip();
        const slipEls = document.querySelectorAll('.uplatnica-wrapper');

        for (let i = 0; i < slipEls.length; i++) {
          const slip = csvSlips.value[i];
          const rawName = (slip.imePlatitelja || '').trim() || `uplatnica_${i + 1}`;
          const safeName = rawName.replace(/[\\/:*?"<>|]/g, '_');

          const canvas = await html2canvas(slipEls[i], {
            scale: 2,
            useCORS: true,
            width: 931,
            height: 380,
            windowWidth: 1400,
            logging: false,
          });

          const dataUrl = canvas.toDataURL('image/png');
          zip.file(`${safeName}.png`, dataUrl.split(',')[1], { base64: true });
        }

        const blob = await zip.generateAsync({ type: 'blob' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'uplatnice.zip';
        a.click();
        URL.revokeObjectURL(url);
      } finally {
        zipping.value = false;
      }
    }

    return { loading, activeTab, singleSlip, csvSlips, zipping, switchTab, onSingleGenerate, onCsvLoaded, onBarcodeGenerated, printSlips, downloadZip };
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

/* ── Reset ── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Inter', Arial, sans-serif;
  background: #07091c;
  background-image:
    radial-gradient(ellipse at 15% 50%, rgba(63, 114, 255, 0.18) 0%, transparent 55%),
    radial-gradient(ellipse at 85% 5%,  rgba(120, 80, 255, 0.14) 0%, transparent 55%),
    radial-gradient(ellipse at 50% 95%, rgba(20, 180, 220, 0.08) 0%, transparent 50%);
  min-height: 100vh;
  color: #dde6ff;
  -webkit-font-smoothing: antialiased;
}

#app { min-height: 100vh; }

/* ── Splash ── */
.splash {
  position: fixed;
  inset: 0;
  background: #07091c;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99999;
}

.splash-card { text-align: center; color: white; user-select: none; }
.splash-logo { margin-bottom: 2rem; }

.splash-mark {
  width: 52px;
  height: 64px;
  background: linear-gradient(145deg, #4f7cff 0%, #7c5cfc 100%);
  border-radius: 10px 10px 10px 10px;
  margin: 0 auto;
  position: relative;
  box-shadow: 0 0 40px rgba(79,124,255,0.5), 0 0 80px rgba(124,92,252,0.25);
}

.splash-mark::before {
  content: '';
  position: absolute;
  top: 0; right: 0;
  width: 16px; height: 16px;
  background: #07091c;
  clip-path: polygon(100% 0, 0 0, 100% 100%);
  border-radius: 0 10px 0 0;
}

.splash-mark::after {
  content: '';
  position: absolute;
  bottom: 14px; left: 10px; right: 10px;
  height: 2px;
  background: rgba(255,255,255,0.5);
  border-radius: 2px;
  box-shadow: 0 -8px 0 rgba(255,255,255,0.35), 0 -16px 0 rgba(255,255,255,0.2);
}

.splash-title {
  font-size: 2.6rem;
  font-weight: 800;
  letter-spacing: -1px;
  line-height: 1.1;
  margin-bottom: 0.65rem;
  background: linear-gradient(135deg, #fff 20%, rgba(180,200,255,0.8));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.splash-sub {
  font-size: 0.68rem;
  color: rgba(255,255,255,0.3);
  letter-spacing: 2.5px;
  text-transform: uppercase;
  margin-bottom: 2.5rem;
}

.splash-bar {
  width: 180px;
  height: 2px;
  background: rgba(255,255,255,0.08);
  border-radius: 999px;
  margin: 0 auto;
  overflow: hidden;
}

.splash-bar-fill {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, #4f7cff, #7c5cfc, #4f7cff);
  background-size: 200% 100%;
  border-radius: 999px;
  animation: splash-progress 2.8s ease-out forwards;
}

@keyframes splash-progress {
  0%   { width: 0%; }
  60%  { width: 75%; }
  100% { width: 100%; }
}

.splash-fade-leave-active { transition: opacity 0.6s ease; }
.splash-fade-leave-to { opacity: 0; }

/* ── Header ── */
.app-header {
  background: rgba(7, 9, 28, 0.88);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  color: white;
  padding: 0.85rem 2.5rem;
  display: flex;
  align-items: center;
  gap: 2rem;
  box-shadow: 0 1px 0 rgba(255,255,255,0.06), 0 8px 32px rgba(0,0,0,0.4);
  border-bottom: 1px solid rgba(255,255,255,0.06);
  position: sticky;
  top: 0;
  z-index: 100;
}

.app-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-shrink: 0;
}

.app-brand-mark {
  width: 34px;
  height: 34px;
  background: linear-gradient(135deg, #4f7cff 0%, #7c5cfc 100%);
  border-radius: 9px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.7rem;
  font-weight: 900;
  letter-spacing: -0.5px;
  color: white;
  box-shadow: 0 0 18px rgba(79,124,255,0.45);
  flex-shrink: 0;
}

.app-brand-text {
  display: flex;
  flex-direction: column;
  line-height: 1.2;
}

.app-brand-title {
  font-size: 1rem;
  font-weight: 700;
  white-space: nowrap;
  letter-spacing: -0.3px;
  color: #e8eeff;
}

.app-brand-sub {
  font-size: 0.6rem;
  color: rgba(255,255,255,0.3);
  letter-spacing: 1.5px;
  text-transform: uppercase;
}

/* ── Tabs ── */
.tabs {
  display: flex;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 999px;
  padding: 3px;
  gap: 2px;
}

.tab-btn {
  padding: 0.38rem 1.15rem;
  border: none;
  background: transparent;
  color: rgba(255,255,255,0.45);
  border-radius: 999px;
  cursor: pointer;
  font-size: 0.84rem;
  font-family: inherit;
  font-weight: 500;
  transition: all 0.2s;
}

.tab-btn:hover { color: rgba(255,255,255,0.85); background: rgba(255,255,255,0.08); }

.tab-btn.active {
  background: linear-gradient(135deg, #4f7cff 0%, #7c5cfc 100%);
  color: white;
  font-weight: 700;
  box-shadow: 0 2px 14px rgba(79,124,255,0.45);
}

/* ── BMC button ── */
.bmc-btn {
  margin-left: auto;
  display: inline-flex;
  align-items: center;
  padding: 0.42rem 1rem;
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.12);
  color: rgba(255,255,255,0.75);
  border-radius: 999px;
  font-size: 0.78rem;
  font-weight: 600;
  font-family: inherit;
  text-decoration: none;
  white-space: nowrap;
  transition: all 0.2s;
}

.bmc-btn:hover {
  background: rgba(255,255,255,0.12);
  color: white;
  border-color: rgba(255,255,255,0.2);
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
  background: rgba(255,255,255,0.04);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-radius: 14px;
  padding: 0.85rem 1.25rem;
  border: 1px solid rgba(255,255,255,0.08);
}

.preview-label {
  font-weight: 600;
  font-size: 0.9rem;
  color: #c4d0ff;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.preview-label::before {
  content: '';
  display: inline-block;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: linear-gradient(135deg, #4f7cff, #7c5cfc);
  box-shadow: 0 0 8px rgba(79,124,255,0.8);
}

.preview-actions {
  display: flex;
  gap: 0.6rem;
  align-items: center;
}

.btn-zip {
  padding: 0.52rem 1.2rem;
  background: rgba(124,92,252,0.12);
  border: 1.5px solid rgba(124,92,252,0.35);
  color: #a78bfa;
  border-radius: 10px;
  font-size: 0.855rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
  letter-spacing: 0.01em;
}

.btn-zip:hover:not(:disabled) {
  background: rgba(124,92,252,0.22);
  border-color: rgba(124,92,252,0.6);
  color: #c4b5fd;
}

.btn-zip:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-print {
  padding: 0.52rem 1.35rem;
  background: linear-gradient(135deg, #3a60e8 0%, #4f7cff 60%, #6d96ff 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 0.855rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 4px 18px rgba(79,124,255,0.45);
  transition: transform 0.15s, box-shadow 0.15s;
  letter-spacing: 0.01em;
}

.btn-print:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 28px rgba(79,124,255,0.6);
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
    font-size: 0.72rem;
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

  .preview-actions { flex-direction: column; }

  .btn-zip,
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
