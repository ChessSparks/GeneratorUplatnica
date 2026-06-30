<template>
  <div class="csv-upload">
    <h2>Uvoz iz CSV datoteke</h2>

    <div
      class="drop-zone"
      :class="{ dragging: isDragging, loaded: slips.length > 0 }"
      @dragover.prevent="isDragging = true"
      @dragleave="isDragging = false"
      @drop.prevent="onDrop"
    >
      <template v-if="!fileName">
        <div class="drop-icon">📂</div>
        <p>
          Povucite CSV ovdje ili
          <label class="file-link">
            kliknite za odabir
            <input type="file" accept=".csv" @change="onFileSelect" />
          </label>
        </p>
      </template>
      <div v-else class="file-info">
        <span class="file-name">{{ fileName }}</span>
        <button class="btn-remove" @click.stop="clearFile">✕ Ukloni</button>
      </div>
    </div>

    <p v-if="parseError" class="msg-error">⚠️ {{ parseError }}</p>
    <p v-if="slips.length" class="msg-success">✅ Učitano {{ slips.length }} uplatnica(e).</p>

    <div class="csv-info">
      <p class="info-title">Očekivani stupci CSV-a:</p>
      <code class="csv-cols">imePlatitelja, adresaPlatitelja, postanskiBrojIMjestoPlatitelja, imePrimatelja, adresaPrimatelja, postanskiBrojIMjestoPrimatelja, ibanPrimatelja, iznosTransakcije, modelPlacanja, pozivNaBroj, sifraNamjene, opisPlacanja</code>
      <button class="btn-template" @click="downloadTemplate">⬇ Preuzmi predložak CSV</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import Papa from 'papaparse';

const CSV_COLUMNS = [
  'imePlatitelja',
  'adresaPlatitelja',
  'postanskiBrojIMjestoPlatitelja',
  'imePrimatelja',
  'adresaPrimatelja',
  'postanskiBrojIMjestoPrimatelja',
  'ibanPrimatelja',
  'iznosTransakcije',
  'modelPlacanja',
  'pozivNaBroj',
  'sifraNamjene',
  'opisPlacanja',
];

const EXAMPLE_ROWS = [
  ['Ivan Ivić', 'Ilica 1', '10000 Zagreb', 'Udruga XYZ', 'Trg J.J.Strossmayera 1', '31000 Osijek', 'HR1210010051863000160', '100,00', 'HR00', '2026-001', 'SALA', 'Članarina 2026'],
  ['Marija Horvat', 'Vlaška 22', '10000 Zagreb', 'Udruga XYZ', 'Trg J.J.Strossmayera 1', '31000 Osijek', 'HR1210010051863000160', '150,50', 'HR01', '2026-002', 'SALA', 'Članarina 2026'],
  ['Stjepan Kovačević', 'Frankopanska 5', '51000 Rijeka', 'Udruga XYZ', 'Trg J.J.Strossmayera 1', '31000 Osijek', 'HR1210010051863000160', '200,00', 'HR00', '2026-003', 'SALA', 'Članarina 2026'],
  ['Ana Šimić', 'Mažuranićevo šetalište 3', '21000 Split', 'Udruga XYZ', 'Trg J.J.Strossmayera 1', '31000 Osijek', 'HR1210010051863000160', '75,00', 'HR02', '2026-004', 'SALA', 'Članarina 2026'],
  ['Tomislav Blažević', 'Gundulićeva 10', '20000 Dubrovnik', 'Udruga XYZ', 'Trg J.J.Strossmayera 1', '31000 Osijek', 'HR1210010051863000160', '120,00', 'HR00', '2026-005', 'SALA', 'Članarina 2026'],
];

export default {
  name: 'CsvUpload',
  emits: ['slips-loaded'],
  setup(_, { emit }) {
    const isDragging = ref(false);
    const fileName = ref('');
    const parseError = ref('');
    const slips = ref([]);

    function rowToSlip(row) {
      return {
        imePlatitelja: row.imePlatitelja || '',
        adresaPlatitelja: row.adresaPlatitelja || '',
        postanskiBrojIMjestoPlatitelja: row.postanskiBrojIMjestoPlatitelja || '',
        imePrimatelja: row.imePrimatelja || '',
        adresaPrimatelja: row.adresaPrimatelja || '',
        postanskiBrojIMjestoPrimatelja: row.postanskiBrojIMjestoPrimatelja || '',
        ibanPrimatelja: row.ibanPrimatelja || '',
        iznosTransakcije: row.iznosTransakcije || '',
        modelPlacanja: row.modelPlacanja || 'HR00',
        pozivNaBroj: row.pozivNaBroj || '',
        sifraNamjene: row.sifraNamjene || '',
        opisPlacanja: row.opisPlacanja || '',
      };
    }

    function processFile(file) {
      if (!file) return;
      parseError.value = '';
      slips.value = [];
      fileName.value = file.name;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        encoding: 'UTF-8',
        complete(results) {
          if (results.errors.length) {
            parseError.value = results.errors[0].message;
            return;
          }
          slips.value = results.data.map(rowToSlip);
          emit('slips-loaded', slips.value);
        },
        error(err) {
          parseError.value = err.message;
        },
      });
    }

    function onFileSelect(e) {
      processFile(e.target.files[0]);
      e.target.value = '';
    }

    function onDrop(e) {
      isDragging.value = false;
      processFile(e.dataTransfer.files[0]);
    }

    function clearFile() {
      fileName.value = '';
      slips.value = [];
      parseError.value = '';
      emit('slips-loaded', []);
    }

    function downloadTemplate() {
      function esc(val) {
        const s = String(val);
        if (s.includes(',') || s.includes('"') || s.includes('\n') || s.includes('\r')) {
          return '"' + s.replace(/"/g, '""') + '"';
        }
        return s;
      }
      const rows = [
        CSV_COLUMNS.map(esc).join(','),
        ...EXAMPLE_ROWS.map(r => r.map(esc).join(',')),
      ].join('\r\n');
      // UTF-8 BOM so Excel opens Croatian characters correctly
      const blob = new Blob(['﻿' + rows], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'predlozak-uplatnica.csv';
      a.click();
      URL.revokeObjectURL(url);
    }

    return { isDragging, fileName, parseError, slips, onFileSelect, onDrop, clearFile, downloadTemplate };
  },
};
</script>

<style scoped>
.csv-upload {
  background: white;
  border-radius: 20px;
  padding: 2rem 2.25rem;
  box-shadow: 0 4px 32px rgba(14, 30, 80, 0.08), 0 1px 4px rgba(0,0,0,0.04);
  border: 1px solid rgba(59, 108, 247, 0.07);
}

.csv-upload h2 {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  color: #1a2035;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.csv-upload h2::before {
  content: '';
  display: inline-block;
  width: 4px;
  height: 18px;
  border-radius: 2px;
  background: linear-gradient(180deg, #3b6cf7, #6b4ef7);
}

.drop-zone {
  border: 2px dashed #c5d5fb;
  border-radius: 14px;
  padding: 2.5rem 2rem;
  text-align: center;
  background: #f7f9ff;
  transition: all 0.2s;
  cursor: default;
}

.drop-zone.dragging {
  background: #eef2ff;
  border-color: #3b6cf7;
  box-shadow: 0 0 0 4px rgba(59,108,247,0.1);
}

.drop-zone.loaded {
  background: #f0fff8;
  border-color: #38a169;
  border-style: solid;
}

.drop-icon {
  font-size: 2.5rem;
  margin-bottom: 0.65rem;
  filter: drop-shadow(0 2px 8px rgba(59,108,247,0.2));
}

.drop-zone p {
  font-size: 0.88rem;
  color: #64748b;
  font-weight: 500;
}

.file-link {
  color: #3b6cf7;
  cursor: pointer;
  font-weight: 600;
  text-decoration: none;
  border-bottom: 1px solid rgba(59,108,247,0.3);
  padding-bottom: 1px;
  transition: border-color 0.15s;
}

.file-link:hover { border-color: #3b6cf7; }
.file-link input[type='file'] { display: none; }

.file-info {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
}

.file-name {
  font-weight: 600;
  color: #276749;
  font-size: 0.9rem;
}

.btn-remove {
  background: none;
  border: 1.5px solid #e53e3e;
  color: #e53e3e;
  border-radius: 6px;
  padding: 3px 10px;
  font-size: 0.75rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-remove:hover { background: #fff5f5; }

.msg-error {
  margin-top: 0.85rem;
  color: #e53e3e;
  font-size: 0.83rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.msg-success {
  margin-top: 0.85rem;
  color: #276749;
  font-size: 0.85rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.35rem;
}

.csv-info {
  margin-top: 1.5rem;
  padding-top: 1.25rem;
  border-top: 1px solid #e8edfc;
}

.info-title {
  font-size: 0.72rem;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 0.5rem;
}

.csv-cols {
  display: block;
  font-size: 0.7rem;
  background: #f0f4ff;
  padding: 8px 12px;
  border-radius: 8px;
  margin-bottom: 1rem;
  word-break: break-all;
  color: #3b5298;
  font-family: 'Courier New', monospace;
  border: 1px solid #dde6fc;
  line-height: 1.6;
}

.btn-template {
  padding: 0.5rem 1.2rem;
  background: white;
  border: 1.5px solid #3b6cf7;
  color: #3b6cf7;
  border-radius: 9px;
  font-size: 0.85rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-template:hover {
  background: #f0f4ff;
  box-shadow: 0 2px 10px rgba(59,108,247,0.15);
}
</style>
