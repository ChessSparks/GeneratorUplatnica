<template>
  <div class="uplatnica-wrapper" id="izvoz-uplatnice">
    <!-- Platitelj (top-left) -->
    <div class="platitelj">
      <input type="text" class="input-field" :value="userData.imePlatitelja" readonly />
      <input type="text" class="input-field" :value="userData.adresaPlatitelja" readonly />
      <input type="text" class="input-field" :value="userData.postanskiBrojIMjestoPlatitelja" readonly />
    </div>

    <!-- Valuta -->
    <input class="valuta" type="text" value=" EUR" disabled />

    <!-- Iznos (left) -->
    <input class="iznos" type="text" :value="formattedAmount" readonly />

    <!-- IBAN primatelja -->
    <input class="iban-primatelja" type="text" :value="userData.ibanPrimatelja" readonly />

    <!-- Primatelj (bottom-left) -->
    <div class="primatelj">
      <input type="text" class="input-field" :value="userData.imePrimatelja" readonly />
      <input type="text" class="input-field" :value="userData.adresaPrimatelja" readonly />
      <input type="text" class="input-field" :value="userData.postanskiBrojIMjestoPrimatelja" readonly />
    </div>

    <!-- Model plaćanja -->
    <input class="model-placanja" type="text" :value="userData.modelPlacanja" readonly />

    <!-- Poziv na broj -->
    <input class="poziv-na-broj-primatelja" type="text" :value="userData.pozivNaBroj" readonly />

    <!-- Šifra namjene -->
    <input class="sifra-namjene" type="text" :value="userData.sifraNamjene" readonly />

    <!-- Opis plaćanja -->
    <textarea class="opis-placanja" :value="userData.opisPlacanja" readonly></textarea>

    <!-- Barcode (bottom-left) -->
    <div class="generated-barcode">
      <canvas ref="barcodeCanvas"></canvas>
      <span v-if="barcodeError" class="barcode-error">{{ barcodeError }}</span>
    </div>

    <!-- Right copy: iznos -->
    <input class="iznos-desno" type="text" :value="formattedAmount" readonly />

    <!-- Right copy: IBAN -->
    <input class="iban-primatelja-desno" type="text" :value="userData.ibanPrimatelja" readonly />

    <!-- Right copy: model -->
    <input class="model-placanja-desno" type="text" :value="userData.modelPlacanja" readonly />

    <!-- Right copy: poziv na broj -->
    <input class="poziv-na-broj-primatelja-desno" type="text" :value="userData.pozivNaBroj" readonly />

    <!-- Right copy: opis -->
    <textarea class="opis-placanja-desno" :value="userData.opisPlacanja" readonly></textarea>
  </div>
</template>


<script>
import { ref, computed, onMounted, watch } from 'vue';

export default {
  name: 'PaymentSlip',
  props: {
    userData: {
      type: Object,
      required: true,
    },
  },
  emits: ['barcode-generated'],
  setup(props, { emit }) {
    const barcodeCanvas = ref(null);
    const barcodeError = ref(null);

    const formattedAmount = computed(() => {
      const raw = (props.userData.iznosTransakcije || '').toString().replace(',', '.');
      const num = parseFloat(raw);
      if (isNaN(num)) return '';
      return num.toFixed(2).replace('.', ',');
    });

    function buildBarcodeText(data) {
      const raw = (data.iznosTransakcije || '').toString().replace(',', '.');
      const amount = Math.round(parseFloat(raw || 0) * 100)
        .toString()
        .padStart(15, '0');

      return [
        'HRVHUB30',
        'EUR',
        amount,
        (data.imePlatitelja || '').substring(0, 30),
        (data.adresaPlatitelja || '').substring(0, 27),
        (data.postanskiBrojIMjestoPlatitelja || '').substring(0, 27),
        (data.imePrimatelja || '').substring(0, 25),
        (data.adresaPrimatelja || '').substring(0, 25),
        (data.postanskiBrojIMjestoPrimatelja || '').substring(0, 27),
        (data.ibanPrimatelja || '').replace(/\s/g, '').substring(0, 21),
        (data.modelPlacanja || 'HR00').substring(0, 4),
        (data.pozivNaBroj || '').substring(0, 22),
        (data.sifraNamjene || '').substring(0, 4),
        (data.opisPlacanja || '').substring(0, 35),
      ].join('\n');
    }

    async function generateBarcode() {
      if (!barcodeCanvas.value) return;
      barcodeError.value = null;
      try {
        const bwipjs = (await import('bwip-js')).default;
        const text = buildBarcodeText(props.userData);
        bwipjs.toCanvas(barcodeCanvas.value, {
          bcid: 'pdf417',
          text,
          scale: 2,
          height: 24,
          includetext: false,
        });
        emit('barcode-generated', text);
      } catch (err) {
        barcodeError.value = 'Greška pri generiranju barkoda';
        console.error(err);
      }
    }

    onMounted(generateBarcode);
    watch(() => props.userData, generateBarcode, { deep: true });

    return { barcodeCanvas, barcodeError, formattedAmount };
  },
};
</script>

<style>
.uplatnica-wrapper {
  width: 931px;
  height: 380px;
  background-size: cover;
  background-image: url('../assets/uplatnica.png');
  margin: 0 auto;
  position: relative;
}

.uplatnica-wrapper .platitelj {
  position: absolute;
  display: flex;
  flex-direction: column;
  max-width: 200px;
  left: 31px;
  top: 44px;
}
.uplatnica-wrapper .platitelj input {
  height: 20px;
  border: 0px solid transparent;
  font-size: 12px;
}

.uplatnica-wrapper .valuta {
  width: 50px;
  height: 27px;
  top: 26px;
  position: absolute;
  left: 334px;
  border: 0 solid;
  background: transparent;
  font-size: 12px;
}

.uplatnica-wrapper .iznos {
  position: absolute;
  top: 24px;
  left: 365px;
  width: 285px;
  background: transparent;
  font-size: 12px;
  border: 0 solid;
  text-align: right;
  letter-spacing: 3px;
  height: 27px;
  padding-top: 5px;
}

.uplatnica-wrapper .iban-primatelja {
  position: absolute;
  top: 115px;
  left: 344px;
  border: 0 solid transparent;
  background-color: transparent;
  font-size: 12px;
  letter-spacing: 8px;
  max-width: 312px;
  width: 100%;
  height: 27px;
}

.uplatnica-wrapper .primatelj {
  position: absolute;
  display: flex;
  flex-direction: column;
  max-width: 200px;
  left: 31px;
  top: 170px;
}
.uplatnica-wrapper .primatelj input {
  height: 20px;
  border: 0px solid transparent;
  font-size: 12px;
}

.uplatnica-wrapper .model-placanja {
  position: absolute;
  top: 150px;
  background: transparent;
  left: 236px;
  letter-spacing: 7px;
  width: 57px;
  height: 27px;
  border: 0 solid;
  font-size: 12px;
}

.uplatnica-wrapper .poziv-na-broj-primatelja {
  height: 27px;
  width: 200px;
  position: absolute;
  top: 148px;
  left: 323px;
  border: 0 solid;
  background: transparent;
  font-size: 12px;
  padding-top: 5px;
}

.uplatnica-wrapper .sifra-namjene {
  height: 27px;
  position: absolute;
  top: 187px;
  left: 232px;
  border: 0 solid;
  background: transparent;
  width: 57px;
  font-size: 12px;
}

.uplatnica-wrapper .opis-placanja {
  width: 249px;
  position: absolute;
  background: transparent;
  top: 178px;
  left: 384px;
  height: 50px;
  border: 0 solid;
  font-size: 12px;
  resize: none;
}

.uplatnica-wrapper .generated-barcode {
  position: absolute;
  width: 250px;
  left: 30px;
  bottom: 33px;
}
.uplatnica-wrapper .generated-barcode canvas {
  width: 100%;
  display: block;
}
.uplatnica-wrapper .generated-barcode .barcode-error {
  font-size: 8pt;
  color: #c00;
}

/* Right copy */
.uplatnica-wrapper .iznos-desno {
  height: 27px;
  padding-top: 5px;
  position: absolute;
  right: 19px;
  top: 26px;
  width: 150px;
  text-align: right;
  background: transparent;
  border: 0 solid;
  font-size: 12px;
}

.uplatnica-wrapper .iban-primatelja-desno {
  position: absolute;
  right: 94px;
  top: 113px;
  border: transparent;
  background: transparent;
  padding-top: 3px;
  font-size: 12px;
  height: 33px;
}

.uplatnica-wrapper .model-placanja-desno {
  position: absolute;
  right: 194px;
  top: 156px;
  width: 40px;
  background: transparent;
  border: 0;
  font-size: 12px;
}

.uplatnica-wrapper .poziv-na-broj-primatelja-desno {
  position: absolute;
  right: 30px;
  top: 156px;
  border: 0 solid;
  background: transparent;
  padding-top: 3px;
  padding-bottom: 5px;
  height: 33px;
  font-size: 12px;
}

.uplatnica-wrapper .opis-placanja-desno {
  position: absolute;
  bottom: 149px;
  right: 12px;
  width: 238px;
  border: 0 transparent;
  background: transparent;
  height: 45px;
  font-size: 12px;
  padding-top: 5px;
  resize: none;
  word-wrap: break-word;
  word-break: break-all;
}

/* Suppress all focus/outline on overlaid inputs */
.uplatnica-wrapper input,
.uplatnica-wrapper textarea {
  background: transparent !important;
  outline: none !important;
  box-shadow: none !important;
}
.uplatnica-wrapper input:focus,
.uplatnica-wrapper textarea:focus {
  outline: none !important;
  background: transparent !important;
  box-shadow: none !important;
}

.input-field {
  background: transparent !important;
  border: 0 !important;
  outline: none !important;
  font-size: 12px;
  height: 20px;
}
</style>
