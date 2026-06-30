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
        <p class="drop-hint">
          Povucite CSV ovdje ili
          <label class="file-link">
            kliknite za odabir
            <input type="file" accept=".csv" @change="onFileSelect" />
          </label>
        </p>
      </template>
      <div v-else class="file-info">
        <span class="file-name">{{ fileName }}</span>
        <button class="btn-remove" @click.stop="clearFile">Ukloni</button>
      </div>
    </div>

    <p v-if="parseError" class="msg-error">{{ parseError }}</p>
    <p v-if="slips.length" class="msg-success">Učitano {{ slips.length }} uplatnica(e).</p>

    <div class="csv-info">
      <p class="info-title">Očekivani stupci CSV-a:</p>
      <code class="csv-cols">imePlatitelja, adresaPlatitelja, postanskiBrojIMjestoPlatitelja, imePrimatelja, adresaPrimatelja, postanskiBrojIMjestoPrimatelja, ibanPrimatelja, iznosTransakcije, modelPlacanja, pozivNaBroj, sifraNamjene, opisPlacanja</code>
      <button class="btn-template" @click="downloadTemplate">Preuzmi predložak CSV</button>
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
  background: rgba(255,255,255,0.03);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 22px;
  padding: 2rem 2.25rem;
  border: 1px solid rgba(255,255,255,0.08);
  box-shadow: 0 8px 48px rgba(0,0,0,0.3);
}

.csv-upload h2 {
  font-size: 0.72rem;
  font-weight: 700;
  margin-bottom: 1.4rem;
  color: rgba(255,255,255,0.35);
  text-transform: uppercase;
  letter-spacing: 2px;
}

.drop-zone {
  border: 1.5px dashed rgba(255,255,255,0.12);
  border-radius: 16px;
  padding: 2.5rem 2rem;
  text-align: center;
  background: rgba(255,255,255,0.03);
  transition: all 0.2s;
  cursor: default;
}

.drop-zone.dragging {
  background: rgba(79,124,255,0.08);
  border-color: #4f7cff;
  box-shadow: 0 0 0 3px rgba(79,124,255,0.15);
}

.drop-zone.loaded {
  background: rgba(74,222,128,0.06);
  border-color: rgba(74,222,128,0.4);
  border-style: solid;
}

.drop-hint {
  font-size: 0.88rem;
  color: rgba(255,255,255,0.35);
  font-weight: 500;
  margin: 0;
}

.file-link {
  color: #6d96ff;
  cursor: pointer;
  font-weight: 600;
  text-decoration: none;
  border-bottom: 1px solid rgba(109,150,255,0.3);
  padding-bottom: 1px;
  transition: border-color 0.15s, color 0.15s;
}

.file-link:hover { border-color: #6d96ff; color: #8fb0ff; }
.file-link input[type='file'] { display: none; }

.file-info {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
}

.file-name {
  font-weight: 600;
  color: #4ade80;
  font-size: 0.9rem;
}

.btn-remove {
  background: rgba(248,113,113,0.08);
  border: 1.5px solid rgba(248,113,113,0.35);
  color: #f87171;
  border-radius: 6px;
  padding: 3px 10px;
  font-size: 0.75rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-remove:hover {
  background: rgba(248,113,113,0.15);
  border-color: rgba(248,113,113,0.6);
}

.msg-error {
  margin-top: 0.85rem;
  color: #f87171;
  font-size: 0.83rem;
  font-weight: 500;
}

.msg-success {
  margin-top: 0.85rem;
  color: #4ade80;
  font-size: 0.85rem;
  font-weight: 600;
}

.csv-info {
  margin-top: 1.5rem;
  padding-top: 1.25rem;
  border-top: 1px solid rgba(255,255,255,0.07);
}

.info-title {
  font-size: 0.65rem;
  font-weight: 700;
  color: rgba(255,255,255,0.25);
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 0.5rem;
}

.csv-cols {
  display: block;
  font-size: 0.68rem;
  background: rgba(255,255,255,0.04);
  padding: 8px 12px;
  border-radius: 8px;
  margin-bottom: 1rem;
  word-break: break-all;
  color: #6d96ff;
  font-family: 'Courier New', monospace;
  border: 1px solid rgba(255,255,255,0.07);
  line-height: 1.7;
}

.btn-template {
  padding: 0.5rem 1.2rem;
  background: rgba(79,124,255,0.1);
  border: 1.5px solid rgba(79,124,255,0.3);
  color: #6d96ff;
  border-radius: 9px;
  font-size: 0.84rem;
  font-family: inherit;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-template:hover {
  background: rgba(79,124,255,0.18);
  border-color: rgba(79,124,255,0.5);
  color: #8fb0ff;
}

@media (max-width: 480px) {
  .csv-upload {
    padding: 1.25rem 1rem;
    border-radius: 14px;
  }

  .drop-zone { padding: 1.75rem 1rem; }

  .btn-template {
    width: 100%;
    text-align: center;
  }
}
</style>
