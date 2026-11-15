<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import { devises, taux as tauxFixes } from '../data/taux.js'

// ---------- Configuration ----------
// false = taux fixes du TP (tests : 102.56, 340.00, 67.24, 10.00) ; true = taux réels via l'API
// (utile pour les captures d'écran des 4 tests : 102.56, 340.00, 67.24, 10.00)
const UTILISER_API_REEL = false
const URL_API = 'https://open.er-api.com/v6/latest/EUR' // taux pour 1 EUR, sans clé
const LONGUEUR_MAX = 10

// ---------- État ----------
const montant = ref('0')          // chaîne pour faciliter la saisie
const deviseSource = ref('USD')
const deviseCible = ref('EUR')
const resultat = ref('')
const erreurMontant = ref(false)  // affiche « Veuillez saisir un montant »

// Taux actuellement utilisés : fixes par défaut, remplacés par l'API si elle répond
const tauxActifs = ref({ ...tauxFixes })
const origineTaux = ref('fixes')  // 'fixes' ou 'reels'

// Les 12 touches du clavier (grille 4 x 3)
const touches = ['7', '8', '9', '4', '5', '6', '1', '2', '3', 'C', '0', '.']

// ---------- Saisie ----------
function appuyer(touche) {
  // Le message d'erreur disparaît à la saisie suivante
  erreurMontant.value = false

  // Touche C : remise à zéro
  if (touche === 'C') {
    montant.value = '0'
    resultat.value = ''
    return
  }

  // Un seul point décimal autorisé
  if (touche === '.' && montant.value.includes('.')) return

  // Limite de 10 caractères
  if (montant.value.length >= LONGUEUR_MAX) return

  // Le montant initial « 0 » est remplacé par le premier chiffre tapé
  if (montant.value === '0' && touche !== '.') {
    montant.value = touche
  } else {
    montant.value += touche
  }
}

// ---------- Calcul (pivot : l'euro) ----------
// Retourne le résultat sous forme de texte à 2 décimales, ou '' si le montant est nul
function calculer() {
  const valeur = parseFloat(montant.value)
  if (!valeur) return ''

  // Même devise : résultat = montant
  if (deviseSource.value === deviseCible.value) return valeur.toFixed(2)

  const montantEnEuro = valeur / tauxActifs.value[deviseSource.value]
  const montantConverti = montantEnEuro * tauxActifs.value[deviseCible.value]
  return montantConverti.toFixed(2)
}

// ---------- Bouton « Convertir » ----------
function convertir() {
  // Règle de gestion : montant nul -> message d'erreur
  if (!parseFloat(montant.value)) {
    erreurMontant.value = true
    resultat.value = ''
    return
  }
  erreurMontant.value = false
  resultat.value = calculer()
}

// ---------- Inversion ----------
function inverser() {
  const temp = deviseSource.value
  deviseSource.value = deviseCible.value
  deviseCible.value = temp
  // Relance la conversion si un résultat est déjà affiché
  if (resultat.value !== '') convertir()
}

// ---------- Bonus 1 : conversion automatique (watch) ----------
// Dès que le montant, une devise ou les taux changent, on recalcule le résultat
watch([montant, deviseSource, deviseCible, tauxActifs], () => {
  resultat.value = calculer()
})

// ---------- Bonus 2 : taux réels via API (fetch) avec repli sur les taux fixes ----------
async function chargerTauxReels() {
  if (!UTILISER_API_REEL) return
  try {
    const reponse = await fetch(URL_API)
    if (!reponse.ok) throw new Error('Erreur HTTP ' + reponse.status)
    const donnees = await reponse.json()

    // On ne garde que nos devises et on vérifie qu'aucune ne manque
    const nouveauxTaux = {}
    for (const d of devises) {
      const t = donnees.rates?.[d]
      if (typeof t !== 'number') throw new Error('Taux manquant : ' + d)
      nouveauxTaux[d] = t
    }
    tauxActifs.value = nouveauxTaux
    origineTaux.value = 'reels'
  } catch (erreur) {
    // Erreur réseau ou réponse invalide : on garde les taux fixes
    console.warn('API indisponible, taux fixes utilisés :', erreur.message)
  }
}

// ---------- Bonus 3 : clavier physique ----------
function gererClavierPhysique(event) {
  if (/^[0-9.]$/.test(event.key)) appuyer(event.key)
  else if (event.key === 'Backspace' || event.key === 'Escape') appuyer('C')
  else if (event.key === 'Enter') convertir()
}

onMounted(() => {
  window.addEventListener('keydown', gererClavierPhysique)
  chargerTauxReels()
})
onUnmounted(() => window.removeEventListener('keydown', gererClavierPhysique))
</script>

<template>
  <div class="carte">
    <!-- Zone du haut : affichage -->
    <section class="affichage">
      <h1>Currency Converter</h1>

      <div class="ligne">
        <select v-model="deviseSource" aria-label="Devise source">
          <option v-for="d in devises" :key="d" :value="d">{{ d }}</option>
        </select>
        <div class="valeur">{{ montant }}</div>
      </div>

      <button class="inverser" @click="inverser" aria-label="Inverser les devises">⇅</button>

      <div class="ligne">
        <select v-model="deviseCible" aria-label="Devise cible">
          <option v-for="d in devises" :key="d" :value="d">{{ d }}</option>
        </select>
        <div class="valeur">{{ resultat }}</div>
      </div>

      <button class="convertir" @click="convertir">CONVERTIR</button>
      <p v-if="erreurMontant" class="erreur">Veuillez saisir un montant</p>
      <p class="info-taux">
        {{ origineTaux === 'reels' ? 'Taux réels (API)' : 'Taux fixes' }}
      </p>
    </section>

    <!-- Zone du bas : clavier numérique -->
    <section class="clavier">
      <button
        v-for="t in touches"
        :key="t"
        :class="{ 'touche-effacer': t === 'C' }"
        @click="appuyer(t)"
      >
        {{ t }}
      </button>
    </section>
  </div>
</template>

<style scoped>
.carte {
  width: 100%;
  max-width: 360px;
  background: #f7f9fa;
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

/* ----- Zone du haut (Flexbox) ----- */
.affichage {
  background: #2bb8e8;
  padding: 20px 20px 24px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  border-radius: 0 0 24px 24px;
}

h1 {
  margin: 0 0 8px;
  text-align: center;
  color: #fff;
  font-size: 20px;
  font-weight: 600;
}

.ligne {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

select {
  padding: 8px 10px;
  border: 1px solid #cfd8dc;
  border-radius: 8px;
  background: #fff;
  font-size: 15px;
  color: #333;
}

.valeur {
  flex: 1;
  min-height: 56px;
  line-height: 56px;
  text-align: right;
  font-size: 48px;
  color: #fff;
  border-bottom: 2px solid #fff;
  overflow: hidden;
  white-space: nowrap;
}

.inverser {
  align-self: flex-start;
  border: none;
  background: transparent;
  color: #fff;
  font-size: 24px;
  cursor: pointer;
  padding: 0 8px;
}

.convertir {
  align-self: center;
  margin-top: 4px;
  padding: 12px 40px;
  border: none;
  border-radius: 999px;
  background: #fff;
  color: #1ba6c9;
  font-weight: 700;
  letter-spacing: 1px;
  cursor: pointer;
}
.convertir:hover { background: #eaf8fd; }

.erreur {
  margin: 0;
  text-align: center;
  color: #fff;
  background: #f0727a;
  padding: 6px 10px;
  border-radius: 8px;
  font-size: 14px;
}

.info-taux {
  margin: 0;
  text-align: center;
  color: rgba(255, 255, 255, 0.85);
  font-size: 12px;
}

/* ----- Zone du bas (CSS Grid 4 x 3) ----- */
.clavier {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  padding: 16px 8px 24px;
}

.clavier button {
  border: none;
  background: transparent;
  color: #1ba6c9;
  font-size: 32px;
  padding: 14px 0;
  cursor: pointer;
}
.clavier button:active { background: #e6f6fb; }

.clavier .touche-effacer { color: #f0727a; }
</style>
