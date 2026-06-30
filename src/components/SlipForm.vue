<template>
  <div class="slip-form">
    <h2>Podaci uplatnice</h2>

    <div class="form-grid">
      <fieldset>
        <legend>Platitelj</legend>
        <div class="form-field">
          <label>Ime i prezime / naziv <span class="opt">(maks. 30)</span></label>
          <input v-model="form.imePlatitelja" maxlength="30" placeholder="npr. Ivan Ivić" />
        </div>
        <div class="form-field">
          <label>Adresa <span class="opt">(maks. 27)</span></label>
          <input v-model="form.adresaPlatitelja" maxlength="27" placeholder="npr. Ilica 1" />
        </div>
        <div class="form-field">
          <label>Poštanski broj i grad <span class="opt">(maks. 27)</span></label>
          <input v-model="form.postanskiBrojIMjestoPlatitelja" maxlength="27" placeholder="npr. 10000 Zagreb" />
        </div>
      </fieldset>

      <fieldset>
        <legend>Primatelj</legend>
        <div class="form-field">
          <label>Ime / naziv <span class="opt">(maks. 25)</span></label>
          <input v-model="form.imePrimatelja" maxlength="25" placeholder="npr. Udruga XYZ" />
        </div>
        <div class="form-field">
          <label>Adresa <span class="opt">(maks. 25)</span></label>
          <input v-model="form.adresaPrimatelja" maxlength="25" placeholder="npr. Trg J.J.S. 1" />
        </div>
        <div class="form-field">
          <label>Poštanski broj i grad <span class="opt">(maks. 27)</span></label>
          <input v-model="form.postanskiBrojIMjestoPrimatelja" maxlength="27" placeholder="npr. 10000 Zagreb" />
        </div>
        <div class="form-field">
          <label>IBAN primatelja <span class="required">*</span></label>
          <input
            v-model="form.ibanPrimatelja"
            maxlength="21"
            placeholder="HR1234567890123456789"
            class="mono"
            :class="{ invalid: attempted && !form.ibanPrimatelja.trim() }"
          />
        </div>
      </fieldset>

      <fieldset>
        <legend>Detalji plaćanja</legend>
        <div class="form-row">
          <div class="form-field">
            <label>Iznos (EUR) <span class="required">*</span></label>
            <input
              :value="iznosDisplay"
              @keydown="onIznosKey"
              class="iznos-input"
              :class="{ invalid: attempted && iznos === 0 }"
              placeholder="0,00"
              autocomplete="off"
            />
          </div>
          <div class="form-field">
            <label>Model plaćanja</label>
            <select v-model="form.modelPlacanja">
              <option value="" disabled>-</option>
              <option v-for="m in modelOptions" :key="m" :value="m">{{ m }}</option>
            </select>
          </div>
        </div>
        <div class="form-field">
          <label>Poziv na broj <span class="opt">(maks. 22)</span></label>
          <input v-model="form.pozivNaBroj" maxlength="22" placeholder="npr. 2026-001" />
        </div>
        <div class="form-row">
          <div class="form-field">
            <label>Šifra namjene</label>
            <select v-model="form.sifraNamjene">
              <option value="" selected disabled>-</option>
              <option value="ADMG">Administracija</option>
              <option value="BUSB">Autobusni</option>
              <option value="CPYR">Autorsko pravo</option>
              <option value="HSPC">Bolnička njega</option>
              <option value="RDTX">Cestarina</option>
              <option value="DEPT">Depozit</option>
              <option value="DERI">Derivati</option>
              <option value="FREX">Devizno tržište</option>
              <option value="CGDD">Direktno terećenje (kartica)</option>
              <option value="DIVD">Dividenda</option>
              <option value="BECH">Dječji doplatak</option>
              <option value="CHAR">Dobrotvorno plaćanje</option>
              <option value="ETUP">Doplata e-novca</option>
              <option value="MTUP">Doplata mobilnog uređaja</option>
              <option value="GOVI">Državno osiguranje</option>
              <option value="ENRG">Energenti</option>
              <option value="CDCD">Gotovinska isplata</option>
              <option value="CSDB">Gotovinska isplata</option>
              <option value="TCSC">Gradske naknade</option>
              <option value="CDCS">Isplata gotovine s naknadom</option>
              <option value="FAND">Isplata naknade za el. nepogode</option>
              <option value="CSLP">Isplata socijalnih zajmova</option>
              <option value="RHBS">Isplata za rehabilitaciju</option>
              <option value="GWLT">Isplata žrtvama rata i invalidima</option>
              <option value="ADCS">Isplate za donacije/sponzorstva</option>
              <option value="PADD">Izravno terećenje</option>
              <option value="INTE">Kamata</option>
              <option value="CDDP">Kartično plaćanje s odgodom</option>
              <option value="CDCB">Kartično plaćanje uz povrat</option>
              <option value="COMC">Komercijalno plaćanje</option>
              <option value="UBIL">Komunalne usluge</option>
              <option value="COMT">Konsolidirano plaćanje</option>
              <option value="SEPI">Kupnja vrijednosnica (interna)</option>
              <option value="GDDS">Kupovina-prodaja roba</option>
              <option value="GSCB">Kupovina-prodaja roba uz povrat</option>
              <option value="GDSV">Kupovina/prodaja roba i usluga</option>
              <option value="SCVE">Kupovina/prodaja usluga</option>
              <option value="HLTC">Kućna njega bolesnika</option>
              <option value="CBLK">Masovni kliring kartica</option>
              <option value="MDCS">Medicinske usluge</option>
              <option value="NWCM">Mrežna komunikacija</option>
              <option value="RENT">Najam</option>
              <option value="ALLW">Naknada</option>
              <option value="SSBE">Naknada socijalnog osiguranja</option>
              <option value="LICF">Naknada za licencu</option>
              <option value="GFRP">Naknada za nezaposlene (stečaj)</option>
              <option value="BENE">Naknada za nezaposlenost/invaliditet</option>
              <option value="CFEE">Naknada za poništenje</option>
              <option value="AEMP">Naknada za zapošljavanje</option>
              <option value="COLL">Naplata</option>
              <option value="FCOL">Naplata naknade (kartica)</option>
              <option value="DBTC">Naplata putem terećenja</option>
              <option value="NOWS">Nenavedeno</option>
              <option value="IDCP">Neopozivo plaćanje (debitna)</option>
              <option value="ICCP">Neopozivo plaćanje (kreditna)</option>
              <option value="BONU">Novčana nagrada (bonus)</option>
              <option value="PAYR">Obračun plaća</option>
              <option value="BLDM">Održavanje zgrada</option>
              <option value="HEDG">Omeđivanje rizika</option>
              <option value="PPTI">Osiguranje imovine</option>
              <option value="LBRI">Osiguranje iz rada</option>
              <option value="OTHR">Ostalo</option>
              <option value="CLPR">Otplata glavnice (auto)</option>
              <option value="HLRP">Otplata stambenog kredita</option>
              <option value="LOAR">Otplata zajma</option>
              <option value="ALMY">Plaćanje alimentacije</option>
              <option value="PRCP">Plaćanje cijene</option>
              <option value="SUPP">Plaćanje dobavljača</option>
              <option value="CFDI">Plaćanje dospjele glavnice</option>
              <option value="GOVT">Plaćanje države</option>
              <option value="PENS">Plaćanje mirovine</option>
              <option value="DCRD">Plaćanje na račun debitne kartice</option>
              <option value="CCRD">Plaćanje na račun kreditne kartice</option>
              <option value="SALA">Plaćanje plaće</option>
              <option value="REBT">Plaćanje popusta/rabata</option>
              <option value="TAXS">Plaćanje poreza</option>
              <option value="VATX">Plaćanje poreza na dodanu vrijednost</option>
              <option value="RINP">Plaćanje rata (ponavljajuće)</option>
              <option value="IHRP">Plaćanje rate na otplatu</option>
              <option value="IVPT">Plaćanje računa</option>
              <option value="CDBL">Plaćanje računa kreditne kartice</option>
              <option value="TREA">Plaćanje riznice</option>
              <option value="CMDT">Plaćanje roba</option>
              <option value="INTC">Plaćanje unutar društva</option>
              <option value="INVS">Plaćanje za fondove i vrijednosnice</option>
              <option value="PRME">Plemeniti metali</option>
              <option value="AGRT">Poljoprivredni transfer</option>
              <option value="INTX">Porez na dohodak</option>
              <option value="PTXP">Porez na imovinu</option>
              <option value="NITX">Porez na neto dohodak</option>
              <option value="ESTX">Porez na ostavštinu</option>
              <option value="GSTX">Porez na robu i usluge</option>
              <option value="HSTX">Porez na stambeni prostor</option>
              <option value="FWLV">Porez na strane radnike</option>
              <option value="WHLD">Porez po odbitku</option>
              <option value="BEXP">Poslovni troškovi</option>
              <option value="REFU">Povrat</option>
              <option value="TAXR">Povrat poreza</option>
              <option value="RIMB">Povrat pogrešne transakcije</option>
              <option value="OFEE">Početna naknada</option>
              <option value="ADVA">Predujam</option>
              <option value="INSU">Premija osiguranja</option>
              <option value="INPC">Premija osiguranja za vozilo</option>
              <option value="TRPT">Prepaid cestarina (ENC)</option>
              <option value="SUBS">Pretplata</option>
              <option value="CASH">Prijenos gotovine</option>
              <option value="PENO">Prisilna naplata</option>
              <option value="COMM">Provizija</option>
              <option value="INSM">Rata</option>
              <option value="ELEC">Račun za el. energiju</option>
              <option value="CBTV">Račun za kabelsku TV</option>
              <option value="OTLC">Račun za ostale telekom usluge</option>
              <option value="GASB">Račun za plin</option>
              <option value="WTER">Račun za vodu</option>
              <option value="ANNI">Renta</option>
              <option value="BBSC">Rodiljna naknada</option>
              <option value="NETT">Saldiranje</option>
              <option value="STDY">Studiranje</option>
              <option value="ROYA">Tantijeme</option>
              <option value="PHON">Telefonski račun</option>
              <option value="FERB">Trajektni</option>
              <option value="DMEQ">Trajna medicinska pomagala</option>
              <option value="WEBI">Transakcija – internet</option>
              <option value="TELI">Transakcija – telefon</option>
              <option value="TRAD">Trgovinske usluge</option>
              <option value="COST">Troškovi</option>
              <option value="CPKC">Troškovi parkiranja</option>
              <option value="TBIL">Troškovi telekomunikacija</option>
              <option value="NWCH">Troškovi za mrežu</option>
              <option value="EDUC">Troškovi školovanja</option>
              <option value="LIMA">Upravljanje likvidnošću</option>
              <option value="ACCT">Upravljanje računom</option>
              <option value="ANTS">Usluge anestezije</option>
              <option value="VIEW">Usluge oftalmološke skrbi</option>
              <option value="LTCF">Ustanova dugoročne zdravstvene skrbi</option>
              <option value="ICRF">Ustanova socijalne skrbi</option>
              <option value="CVCF">Ustanova za rekonvalescente</option>
              <option value="PTSP">Uvjeti plaćanja</option>
              <option value="MSVC">Višestruke usluge</option>
              <option value="SECU">Vrijednosni papiri</option>
              <option value="LOAN">Zajam</option>
              <option value="FCPM">Zakašnjele naknade</option>
              <option value="TRFD">Zaklada</option>
              <option value="CDQC">Zamjenska gotovina</option>
              <option value="HLTI">Zdravstveno osiguranje</option>
              <option value="AIRB">Zračni</option>
              <option value="DNTS">Zubarske usluge</option>
              <option value="SAVG">Štednja</option>
              <option value="RLWY">Željeznički</option>
              <option value="LIFI">Životno osiguranje</option>
            </select>
          </div>
          <div class="form-field">
            <label>Opis plaćanja <span class="required">*</span> <span class="opt">(maks. 35)</span></label>
            <textarea
              v-model="form.opisPlacanja"
              maxlength="35"
              placeholder="npr. Članarina 2026"
              rows="2"
              :class="{ invalid: attempted && !form.opisPlacanja.trim() }"
            ></textarea>
          </div>
        </div>
      </fieldset>
    </div>

    <div class="form-actions">
      <button class="btn-primary" @click="generate">Generiraj uplatnicu</button>
      <button class="btn-secondary" @click="clear">Poništi</button>
    </div>
    <p v-if="attempted && !isValid" class="error-msg">
      Molimo unesite IBAN primatelja, iznos i opis plaćanja.
    </p>
  </div>

  <BmcModal :modelValue="showModal" @skip="continueGenerate" />
</template>

<script>
import { ref, reactive, computed } from 'vue';
import BmcModal from './BmcModal.vue';

const MODEL_OPTIONS = [
  'HR00','HR01','HR02','HR03','HR04','HR05','HR06','HR07','HR08','HR09','HR10',
  'HR11','HR12','HR13','HR14','HR15','HR16','HR17','HR18','HR23','HR24',
  'HR25','HR26','HR27','HR28','HR29','HR30','HR31','HR33','HR34','HR40',
  'HR41','HR42','HR43','HR50','HR55','HR62','HR63','HR64','HR65','HR67',
  'HR68','HR69','HR83','HR84','HR99',
];

export default {
  name: 'SlipForm',
  components: { BmcModal },
  emits: ['generate'],
  setup(_, { emit }) {
    const attempted = ref(false);
    const iznos = ref(0); // stored as integer cents (lipe)

    const iznosDisplay = computed(() => {
      const padded = iznos.value.toString().padStart(3, '0');
      const intPart = parseInt(padded.slice(0, -2)) || 0;
      const decPart = padded.slice(-2);
      return `${intPart},${decPart}`;
    });

    function onIznosKey(e) {
      if (e.key >= '0' && e.key <= '9') {
        e.preventDefault();
        if (iznos.value >= 9999999999999) return;
        iznos.value = iznos.value * 10 + parseInt(e.key);
      } else if (e.key === 'Backspace') {
        e.preventDefault();
        iznos.value = Math.floor(iznos.value / 10);
      }
    }

    const form = reactive({
      imePlatitelja: '',
      adresaPlatitelja: '',
      postanskiBrojIMjestoPlatitelja: '',
      imePrimatelja: '',
      adresaPrimatelja: '',
      postanskiBrojIMjestoPrimatelja: '',
      ibanPrimatelja: '',
      modelPlacanja: 'HR00',
      pozivNaBroj: '',
      sifraNamjene: '',
      opisPlacanja: '',
    });

    const isValid = computed(
      () =>
        form.ibanPrimatelja.trim() !== '' &&
        iznos.value > 0 &&
        form.opisPlacanja.trim() !== ''
    );

    const showModal = ref(false);
    const pendingData = ref(null);

    function generate() {
      attempted.value = true;
      if (!isValid.value) return;
      pendingData.value = { ...form, iznosTransakcije: iznosDisplay.value };
      showModal.value = true;
    }

    function continueGenerate() {
      showModal.value = false;
      if (pendingData.value) {
        emit('generate', pendingData.value);
        pendingData.value = null;
      }
    }

    function clear() {
      Object.assign(form, {
        imePlatitelja: '',
        adresaPlatitelja: '',
        postanskiBrojIMjestoPlatitelja: '',
        imePrimatelja: '',
        adresaPrimatelja: '',
        postanskiBrojIMjestoPrimatelja: '',
        ibanPrimatelja: '',
        modelPlacanja: 'HR00',
        pozivNaBroj: '',
        sifraNamjene: '',
        opisPlacanja: '',
      });
      iznos.value = 0;
      attempted.value = false;
    }

    return { form, attempted, isValid, iznos, iznosDisplay, onIznosKey, generate, continueGenerate, showModal, clear, modelOptions: MODEL_OPTIONS };
  },
};
</script>

<style scoped>
.slip-form {
  background: white;
  border-radius: 20px;
  padding: 2rem 2.25rem;
  box-shadow: 0 4px 32px rgba(14, 30, 80, 0.08), 0 1px 4px rgba(0,0,0,0.04);
  border: 1px solid rgba(59, 108, 247, 0.07);
}

.slip-form h2 {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  color: #1a2035;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.slip-form h2::before {
  content: '';
  display: inline-block;
  width: 4px;
  height: 18px;
  border-radius: 2px;
  background: linear-gradient(180deg, #3b6cf7, #6b4ef7);
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
  gap: 1.1rem;
}

fieldset {
  border: none;
  border-radius: 14px;
  padding: 1.1rem 1.25rem;
  background: #f7f9ff;
  border: 1px solid #e8edfc;
}

legend {
  font-size: 0.68rem;
  font-weight: 700;
  color: #3b6cf7;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  padding: 0 6px;
  background: #f7f9ff;
  border-radius: 4px;
}

.form-field {
  margin-bottom: 0.7rem;
}

.form-field:last-child { margin-bottom: 0; }

.form-field label {
  display: block;
  font-size: 0.73rem;
  font-weight: 500;
  color: #64748b;
  margin-bottom: 4px;
  letter-spacing: 0.01em;
}

.opt { color: #94a3b8; font-weight: 400; }

.required { color: #e53e3e; }

.form-field input,
.form-field select,
.form-field textarea {
  width: 100%;
  padding: 0.52rem 0.8rem;
  border: 1.5px solid #e2e8f0;
  border-radius: 9px;
  font-size: 0.88rem;
  font-family: inherit;
  color: #1a2035;
  background: white;
  transition: border-color 0.15s, box-shadow 0.15s;
}

.form-field input:focus,
.form-field select:focus,
.form-field textarea:focus {
  outline: none;
  border-color: #3b6cf7;
  box-shadow: 0 0 0 3px rgba(59, 108, 247, 0.12);
}

.form-field input.invalid,
.form-field textarea.invalid {
  border-color: #e53e3e;
  box-shadow: 0 0 0 3px rgba(229, 62, 62, 0.1);
}

.iznos-input {
  text-align: right;
  font-family: 'Courier New', monospace;
  letter-spacing: 1px;
  cursor: text;
  font-weight: 600;
  font-size: 0.95rem;
}

.form-field input.mono {
  font-family: 'Courier New', monospace;
  letter-spacing: 1px;
}

.form-field textarea { resize: vertical; }

.form-row { display: flex; gap: 0.6rem; }
.form-row .form-field { flex: 1; }

.form-actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

.btn-primary {
  padding: 0.65rem 1.75rem;
  background: linear-gradient(135deg, #1e4ac8 0%, #3b6cf7 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 0.92rem;
  font-family: inherit;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 16px rgba(30, 74, 200, 0.35);
  transition: transform 0.15s, box-shadow 0.15s;
  letter-spacing: 0.01em;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 22px rgba(30, 74, 200, 0.45);
}

.btn-primary:active { transform: translateY(0); }

.btn-secondary {
  padding: 0.65rem 1.25rem;
  background: white;
  color: #64748b;
  border: 1.5px solid #e2e8f0;
  border-radius: 10px;
  font-size: 0.92rem;
  font-family: inherit;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s;
}

.btn-secondary:hover {
  background: #f7f9ff;
  border-color: #c7d7fc;
  color: #3b6cf7;
}

.error-msg {
  margin-top: 0.6rem;
  color: #e53e3e;
  font-size: 0.8rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.error-msg::before { content: '⚠'; font-size: 0.85rem; }

@media (max-width: 480px) {
  .slip-form {
    padding: 1.25rem 1rem;
    border-radius: 14px;
  }

  .form-row {
    flex-direction: column;
    gap: 0;
  }

  .form-actions {
    flex-direction: column;
  }

  .btn-primary,
  .btn-secondary {
    width: 100%;
    text-align: center;
  }
}
</style>
