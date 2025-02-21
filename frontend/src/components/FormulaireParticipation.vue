<template>
  <div class="form-container">
    <h2>Enregistrer une participation</h2>

    <!-- Sélection de la personne -->
    <div class="form-group">
      <label>Personne</label>
      <select v-model="selectedPers">
        <option :value="null" disabled>-- Choisir une personne --</option>
        <option
          v-for="pers in personnes"
          :key="pers.matricule"
          :value="pers.matricule"
        >
          {{ pers.nom }} {{ pers.prenom }}
        </option>
      </select>
    </div>

    <!-- Sélection du projet -->
    <div class="form-group">
      <label>Projet</label>
      <select v-model="selectedProjet">
        <option :value="null" disabled>-- Choisir un projet --</option>
        <option
          v-for="proj in projets"
          :key="proj.id"
          :value="proj.id"
        >
          {{ proj.nom }}
        </option>
      </select>
    </div>

    <!-- Rôle -->
    <div class="form-group">
      <label>Rôle</label>
      <input type="text" v-model="inputRole" />
    </div>

    <!-- Pourcentage (slider) -->
    <div class="form-group">
      <label>Pourcentage</label>
      <div class="slider-container">
        <input
          type="range"
          min="0"
          max="100"
          step="1"
          v-model="selectedPourcentage"
        />
        <span class="slider-value">{{ selectedPourcentage }}%</span>
      </div>
    </div>

    <!-- Bouton Valider -->
    <button class="btn-submit" @click="enregistrerParticipation">Enregistrer</button>

    <!-- Affichage de l'erreur ou succès -->
    <div v-if="message" :class="messageType" class="message-popup">
      {{ message }}
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, toRefs } from "vue";
// Importer la fonction doAjaxRequest qui gère les erreurs d'API
import doAjaxRequest from "@/util/util.js"

// Les données du composant
let data = reactive({
  projets: [],
  personnes: [],
  inputRole: '',
  selectedProjet: null,
  selectedPers: null,
  selectedPourcentage: 0,
  message: '', // Success or error message
  messageType: '', // Type of message ('success' or 'error')
});

const {
  personnes,
  projets,
  selectedPers,
  selectedProjet,
  inputRole,
  selectedPourcentage,
  message,
  messageType
} = toRefs(data)

// Enregistrer la participation
function enregistrerParticipation() {
  data.message = '';
  data.messageType = '';

  if (!data.selectedPers || !data.selectedProjet) {
    data.message = 'Sélectionnez une personne et un projet.';
    data.messageType = 'error';
    return;
  }
  if (!data.inputRole) {
    data.message = 'Précisez le rôle.';
    data.messageType = 'error';
    return;
  }
  if (data.selectedPourcentage <= 0) {
    data.message = 'Le pourcentage doit être > 0.';
    data.messageType = 'error';
    return;
  }

  const url =
    `http://localhost:8989/api/gestion/participation?matricule=${data.selectedPers}` +
    `&codeProjet=${data.selectedProjet}` +
    `&role=${encodeURIComponent(data.inputRole)}` +
    `&pourcentage=${Number(data.selectedPourcentage/100)}`;

  doAjaxRequest(url, { method: 'POST' })
    .then((result) => {
      console.log('Réponse du serveur :', result);
      data.message = 'Participation enregistrée avec succès !';
      data.messageType = 'success';
      data.selectedPers = null;
      data.selectedProjet = null;
      data.inputRole = '';
      data.selectedPourcentage = 0;
    })
    .catch(error => {
      data.message = error.message || 'Une erreur est survenue.';
      data.messageType = 'error';
    });
}

function fetchProjet() {
  doAjaxRequest("/api/projets") // Méthode GET par défaut
    .then((result) => {
      data.projets = result._embedded.projets;
    })
    .catch(error => alert(error.message));
}

function fetchPersonne() {
  doAjaxRequest("/api/personnes") // Méthode GET par défaut
    .then((result) => {
      data.personnes = result._embedded.personnes;
      console.log("Personnes récupérées :", result);
    })
    .catch(error => alert(error.message));
}

onMounted(() => {
  fetchProjet();
  fetchPersonne();
});
</script>

<style scoped>
.form-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #f4f8fc; /* Light blue background */
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Title */
.form-container h2 {
  margin-bottom: 1.5rem;
  font-size: 1.2rem;
  font-weight: bold;
  color: #2b3a67; /* Darker blue for the title */
}

/* Form Groups */
.form-group {
  margin-bottom: 1rem;
}

/* Labels */
.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #4f6d99; /* Medium blue for labels */
}

/* Select and Text Inputs */
.form-group select,
.form-group input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  font-size: 0.9rem;
  border: 1px solid #5a7bb5; /* Lighter blue border */
  border-radius: 4px;
  background-color: #e1ecf4; /* Very light blue background */
  color: #2b3a67; /* Dark blue text */
}

/* Container for Slider and Value */
.slider-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

/* Slider (Range) */
.slider-container input[type="range"] {
  flex: 1;
  accent-color: #4f6d99; /* Blue accent color for slider */
}

/* Percentage Value */
.slider-value {
  font-weight: 500;
  color: #2b3a67; /* Dark blue text */
}

/* Submit Button */
.btn-submit {
  display: inline-block;
  padding: 0.6rem 1.2rem;
  font-size: 0.9rem;
  color: #fff;
  background-color: #4f6d99; /* Bluish button color */
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.btn-submit:hover {
  background-color: #3b5d85; /* Slightly darker blue on hover */
}

/* Popup Message */
.message-popup {
  margin-top: 1rem;
  padding: 0.6rem;
  border-radius: 4px;
  text-align: center;
  font-weight: 500;
}

.success {
  background-color: #4caf50; /* Green for success */
  color: white;
}

.error {
  background-color: #a776ff; /* Red for error */
  color: white;
}
</style>
