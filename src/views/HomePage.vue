<template>
  <ion-page>
    <!-- ── Header ── -->
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>🎬 Movie Watchlist</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">

      <!-- ── ADD / EDIT FORM ── -->
      <ion-card>
        <ion-card-header>
          <ion-card-title>{{ editingId ? 'Edit Movie' : 'Add a Movie' }}</ion-card-title>
        </ion-card-header>
        <ion-card-content>

          <ion-item>
            <ion-label position="stacked">Movie Title *</ion-label>
            <ion-input v-model="form.title" placeholder="e.g. Inception" />
          </ion-item>

          <ion-item>
            <ion-label position="stacked">Genre *</ion-label>
            <ion-select v-model="form.genre" placeholder="Select genre">
              <ion-select-option value="Action">Action</ion-select-option>
              <ion-select-option value="Comedy">Comedy</ion-select-option>
              <ion-select-option value="Drama">Drama</ion-select-option>
              <ion-select-option value="Horror">Horror</ion-select-option>
              <ion-select-option value="Romance">Romance</ion-select-option>
              <ion-select-option value="Sci-Fi">Sci-Fi</ion-select-option>
              <ion-select-option value="Thriller">Thriller</ion-select-option>
              <ion-select-option value="Animation">Animation</ion-select-option>
              <ion-select-option value="Documentary">Documentary</ion-select-option>
              <ion-select-option value="Fantasy">Fantasy</ion-select-option>
            </ion-select>
          </ion-item>

          <ion-item>
            <ion-label position="stacked">Year *</ion-label>
            <ion-input v-model.number="form.year" type="number" placeholder="e.g. 2010" />
          </ion-item>

          <ion-item>
            <ion-label position="stacked">Rating (1–10) *</ion-label>
            <ion-input v-model.number="form.rating" type="number" min="1" max="10" placeholder="e.g. 8" />
          </ion-item>

          <ion-item>
            <ion-label position="stacked">Status</ion-label>
            <ion-select v-model="form.status" placeholder="Select status">
              <ion-select-option value="Watched">Watched</ion-select-option>
              <ion-select-option value="Not Watched">Not Watched</ion-select-option>
            </ion-select>
          </ion-item>

          <div class="ion-margin-top" style="display: flex; gap: 8px;">
            <ion-button expand="block" style="flex: 1;" @click="saveMovie">
              {{ editingId ? 'Update Movie' : 'Add Movie' }}
            </ion-button>
            <ion-button
              v-if="editingId"
              expand="block"
              fill="outline"
              color="medium"
              style="flex: 1;"
              @click="cancelEdit"
            >
              Cancel
            </ion-button>
          </div>

        </ion-card-content>
      </ion-card>

      <!-- ── FILTER BAR ── -->
      <div style="display: flex; gap: 8px; margin: 12px 0; flex-wrap: wrap;">
        <ion-chip
          :color="filter === 'All' ? 'primary' : 'medium'"
          @click="filter = 'All'"
        >All ({{ movies.length }})</ion-chip>
        <ion-chip
          :color="filter === 'Watched' ? 'success' : 'medium'"
          @click="filter = 'Watched'"
        >Watched ({{ watchedCount }})</ion-chip>
        <ion-chip
          :color="filter === 'Not Watched' ? 'warning' : 'medium'"
          @click="filter = 'Not Watched'"
        >Not Watched ({{ notWatchedCount }})</ion-chip>
      </div>

      <!-- ── LOADING STATE ── -->
      <div v-if="loading" class="ion-text-center ion-padding">
        <ion-spinner name="crescent" />
        <p>Loading movies...</p>
      </div>

      <!-- ── EMPTY STATE ── -->
      <div v-else-if="filteredMovies.length === 0" class="ion-text-center ion-padding">
        <ion-icon :icon="filmOutline" style="font-size: 48px; color: #ccc;" />
        <p style="color: #999;">No movies found. Add one above!</p>
      </div>

      <!-- ── MOVIE LIST (READ) ── -->
      <ion-list v-else>
        <ion-card
          v-for="movie in filteredMovies"
          :key="movie.id"
          style="margin-bottom: 10px;"
        >
          <ion-card-content>
            <div style="display: flex; justify-content: space-between; align-items: flex-start;">
              <div style="flex: 1;">
                <h2 style="font-weight: bold; font-size: 1.1em; margin: 0 0 4px;">
                  {{ movie.title }}
                </h2>
                <p style="margin: 2px 0; color: #555;">
                  🎭 {{ movie.genre }} &nbsp;|&nbsp; 📅 {{ movie.year }}
                </p>
                <p style="margin: 2px 0; color: #555;">
                  ⭐ {{ movie.rating }}/10
                </p>
                <ion-badge :color="movie.status === 'Watched' ? 'success' : 'warning'">
                  {{ movie.status }}
                </ion-badge>
              </div>
              <div style="display: flex; flex-direction: column; gap: 6px; margin-left: 8px;">
                <!-- UPDATE button -->
                <ion-button size="small" fill="outline" @click="startEdit(movie)">
                  Edit
                </ion-button>
                <!-- DELETE button -->
                <ion-button size="small" fill="outline" color="danger" @click="deleteMovie(movie.id)">
                  Delete
                </ion-button>
              </div>
            </div>
          </ion-card-content>
        </ion-card>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
// Vue 3 Composition API with <script setup>
import { ref, computed, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonCard, IonCardHeader, IonCardTitle, IonCardContent,
  IonItem, IonLabel, IonInput, IonSelect, IonSelectOption,
  IonButton, IonList, IonBadge, IonChip, IonSpinner, IonIcon,
  alertController, toastController
} from '@ionic/vue';
import { filmOutline } from 'ionicons/icons';

// Firebase Realtime Database functions
import { db } from '@/firebase';
import {
  ref as dbRef,    // renamed to avoid conflict with Vue's ref()
  push,            // CREATE: add a new record
  onValue,         // READ: listen for changes
  update,          // UPDATE: edit a record
  remove           // DELETE: remove a record
} from 'firebase/database';

// ── TYPES ──────────────────────────────────────────────
interface Movie {
  id: string;
  title: string;
  genre: string;
  year: number;
  rating: number;
  status: 'Watched' | 'Not Watched';
}

// ── REACTIVE STATE ─────────────────────────────────────
const movies = ref<Movie[]>([]);
const loading = ref(true);
const filter = ref<'All' | 'Watched' | 'Not Watched'>('All');
const editingId = ref<string | null>(null);

// Form fields — used for both Add and Edit
const form = ref({
  title: '',
  genre: '',
  year: new Date().getFullYear(),
  rating: 8,
  status: 'Not Watched' as 'Watched' | 'Not Watched'
});

// ── COMPUTED ────────────────────────────────────────────
const filteredMovies = computed(() => {
  if (filter.value === 'All') return movies.value;
  return movies.value.filter(m => m.status === filter.value);
});

const watchedCount = computed(() =>
  movies.value.filter(m => m.status === 'Watched').length
);

const notWatchedCount = computed(() =>
  movies.value.filter(m => m.status === 'Not Watched').length
);

// ── READ: listen to Firebase in real time ───────────────
onMounted(() => {
  const moviesRef = dbRef(db, 'movies');
  onValue(moviesRef, (snapshot) => {
    const data = snapshot.val();
    if (data) {
      movies.value = Object.entries(data).map(([id, val]: [string, any]) => ({
        id,
        ...val
      }));
    } else {
      movies.value = [];
    }
    loading.value = false;
  });
});

// ── VALIDATE FORM ────────────────────────────────────────
function isValid(): boolean {
  if (!form.value.title.trim()) return false;
  if (!form.value.genre) return false;
  if (!form.value.year || form.value.year < 1888 || form.value.year > 2100) return false;
  if (!form.value.rating || form.value.rating < 1 || form.value.rating > 10) return false;
  return true;
}

// ── CREATE / UPDATE ──────────────────────────────────────
async function saveMovie() {
  if (!isValid()) {
    const alert = await alertController.create({
      header: 'Missing Information',
      message: 'Please fill in all fields. Rating must be 1–10. Year must be valid.',
      buttons: ['OK']
    });
    await alert.present();
    return;
  }

  const movieData = {
    title: form.value.title.trim(),
    genre: form.value.genre,
    year: form.value.year,
    rating: form.value.rating,
    status: form.value.status
  };

  if (editingId.value) {
    // UPDATE existing record
    await update(dbRef(db, `movies/${editingId.value}`), movieData);
    showToast('Movie updated!', 'success');
    editingId.value = null;
  } else {
    // CREATE new record — push() generates a unique Firebase key
    await push(dbRef(db, 'movies'), movieData);
    showToast('Movie added!', 'success');
  }

  resetForm();
}

// ── START EDIT ───────────────────────────────────────────
function startEdit(movie: Movie) {
  editingId.value = movie.id;
  form.value = {
    title: movie.title,
    genre: movie.genre,
    year: movie.year,
    rating: movie.rating,
    status: movie.status
  };
  // Scroll back to top so user sees the form
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

function cancelEdit() {
  editingId.value = null;
  resetForm();
}

// ── DELETE ───────────────────────────────────────────────
async function deleteMovie(id: string) {
  const alert = await alertController.create({
    header: 'Delete Movie',
    message: 'Are you sure you want to remove this movie?',
    buttons: [
      { text: 'Cancel', role: 'cancel' },
      {
        text: 'Delete',
        role: 'destructive',
        handler: async () => {
          await remove(dbRef(db, `movies/${id}`));
          showToast('Movie deleted.', 'danger');
        }
      }
    ]
  });
  await alert.present();
}

// ── HELPERS ──────────────────────────────────────────────
function resetForm() {
  form.value = {
    title: '',
    genre: '',
    year: new Date().getFullYear(),
    rating: 8,
    status: 'Not Watched'
  };
}

async function showToast(message: string, color: string) {
  const toast = await toastController.create({
    message,
    duration: 2000,
    color,
    position: 'bottom'
  });
  await toast.present();
}
</script>