<template>
  <ion-page>

    <!-- ═══════════════════════════════════════════
         SIDEBAR MENU (ion-menu)
    ═══════════════════════════════════════════ -->
    <ion-menu content-id="main-content" type="overlay">
      <ion-header>
        <ion-toolbar color="primary">
          <ion-title>🎬 MOVIERS</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content>
        <ion-list lines="none">

          <ion-item button @click="navigate('home')" :color="currentPage === 'home' ? 'light' : ''">
            <ion-icon :icon="homeOutline" slot="start" />
            <ion-label>Home</ion-label>
          </ion-item>

          <ion-item button @click="navigate('all')" :color="currentPage === 'all' ? 'light' : ''">
            <ion-icon :icon="listOutline" slot="start" />
            <ion-label>All Movies</ion-label>
          </ion-item>

          <ion-item button @click="navigate('toprated')" :color="currentPage === 'toprated' ? 'light' : ''">
            <ion-icon :icon="trophyOutline" slot="start" />
            <ion-label>Top Rated</ion-label>
          </ion-item>

          <ion-item button @click="navigate('watched')" :color="currentPage === 'watched' ? 'light' : ''">
            <ion-icon :icon="eyeOutline" slot="start" />
            <ion-label>Watched</ion-label>
          </ion-item>

          <ion-item button @click="navigate('watchlist')" :color="currentPage === 'watchlist' ? 'light' : ''">
            <ion-icon :icon="timeOutline" slot="start" />
            <ion-label>Watchlist</ion-label>
          </ion-item>

          <ion-item-divider>
            <ion-label>Library</ion-label>
          </ion-item-divider>

          <ion-item button @click="navigate('add')" :color="currentPage === 'add' ? 'light' : ''">
            <ion-icon :icon="addCircleOutline" slot="start" />
            <ion-label>Add Movie</ion-label>
          </ion-item>

        </ion-list>

        <!-- Stats inside sidebar -->
        <div style="padding: 16px; margin-top: 8px;">
          <p style="font-size: 12px; color: gray; margin-bottom: 8px;">YOUR STATS</p>
          <div style="display: flex; flex-direction: column; gap: 8px;">
            <div style="display: flex; justify-content: space-between;">
              <span style="font-size: 13px;">Total</span>
              <ion-badge color="primary">{{ movies.length }}</ion-badge>
            </div>
            <div style="display: flex; justify-content: space-between;">
              <span style="font-size: 13px;">Watched</span>
              <ion-badge color="success">{{ watchedCount }}</ion-badge>
            </div>
            <div style="display: flex; justify-content: space-between;">
              <span style="font-size: 13px;">Watchlist</span>
              <ion-badge color="warning">{{ notWatchedCount }}</ion-badge>
            </div>
          </div>
        </div>

      </ion-content>
    </ion-menu>

    <!-- ═══════════════════════════════════════════
         MAIN CONTENT
    ═══════════════════════════════════════════ -->
    <ion-page id="main-content">

      <!-- Header -->
      <ion-header>
        <ion-toolbar color="primary">
          <ion-buttons slot="start">
            <ion-menu-button />
          </ion-buttons>
          <ion-title>{{ pageTitle }}</ion-title>
          <ion-buttons slot="end" v-if="currentPage !== 'add'">
            <ion-button @click="navigate('add')">
              <ion-icon :icon="addOutline" slot="icon-only" />
            </ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>

      <ion-content class="ion-padding">

        <!-- ─── LOADING ─── -->
        <div v-if="loading" class="ion-text-center ion-padding">
          <ion-spinner name="crescent" />
          <p>Loading...</p>
        </div>

        <div v-else>

          <!-- ════════════════════════════
               PAGE: HOME
          ════════════════════════════ -->
          <div v-if="currentPage === 'home'">

            <!-- Stats cards -->
            <div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 20px;">
              <ion-card style="margin: 0; text-align: center;">
                <ion-card-content style="padding: 12px;">
                  <div style="font-size: 24px; font-weight: bold; color: var(--ion-color-primary);">{{ movies.length }}</div>
                  <div style="font-size: 11px; color: gray;">Total</div>
                </ion-card-content>
              </ion-card>
              <ion-card style="margin: 0; text-align: center;">
                <ion-card-content style="padding: 12px;">
                  <div style="font-size: 24px; font-weight: bold; color: var(--ion-color-success);">{{ watchedCount }}</div>
                  <div style="font-size: 11px; color: gray;">Watched</div>
                </ion-card-content>
              </ion-card>
              <ion-card style="margin: 0; text-align: center;">
                <ion-card-content style="padding: 12px;">
                  <div style="font-size: 24px; font-weight: bold; color: var(--ion-color-warning);">{{ notWatchedCount }}</div>
                  <div style="font-size: 11px; color: gray;">To Watch</div>
                </ion-card-content>
              </ion-card>
            </div>

            <!-- Recently added -->
            <p style="font-weight: 600; margin-bottom: 10px;">Recently Added</p>
            <div v-if="recentMovies.length === 0" style="color: gray; font-size: 13px; margin-bottom: 20px;">No movies yet. Add one!</div>
            <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 20px;">
              <ion-card
                v-for="movie in recentMovies"
                :key="movie.id"
                style="margin: 0; cursor: pointer;"
                @click="openMovieDetail(movie)"
              >
                <!-- Cover image or placeholder -->
                <div :style="coverStyle(movie)" style="height: 100px; display: flex; align-items: center; justify-content: center; position: relative;">
                  <ion-icon v-if="!movie.imageUrl" :icon="filmOutline" style="font-size: 36px; color: rgba(255,255,255,0.4);" />
                  <div style="position: absolute; top: 6px; right: 6px;">
                    <ion-badge :color="movie.status === 'Watched' ? 'success' : 'warning'" style="font-size: 9px;">
                      {{ movie.status === 'Watched' ? '✓' : '⏳' }}
                    </ion-badge>
                  </div>
                  <div style="position: absolute; bottom: 6px; left: 6px;">
                    <ion-badge color="dark" style="font-size: 9px;">{{ movie.genre }}</ion-badge>
                  </div>
                </div>
                <ion-card-content style="padding: 8px 10px;">
                  <p style="font-weight: 600; font-size: 13px; margin: 0 0 2px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{{ movie.title }}</p>
                  <p style="font-size: 11px; color: gray; margin: 0;">{{ movie.year }} · ⭐ {{ movie.rating }}/10</p>
                </ion-card-content>
              </ion-card>
            </div>

            <!-- Top rated preview -->
            <p style="font-weight: 600; margin-bottom: 10px;">Top Rated</p>
            <div v-if="topMovies.length === 0" style="color: gray; font-size: 13px;">No movies yet.</div>
            <ion-list v-else style="border-radius: 12px; overflow: hidden;">
              <ion-item
                v-for="(movie, index) in topMovies.slice(0, 5)"
                :key="movie.id"
                button
                @click="openMovieDetail(movie)"
              >
                <div slot="start" style="width: 24px; text-align: center; font-weight: bold; color: gray; font-size: 13px;">
                  #{{ index + 1 }}
                </div>
                <ion-label>
                  <h3>{{ movie.title }}</h3>
                  <p>{{ movie.genre }} · {{ movie.year }}</p>
                </ion-label>
                <div slot="end" style="font-size: 13px; font-weight: 600; color: var(--ion-color-warning);">
                  ⭐ {{ movie.rating }}
                </div>
              </ion-item>
            </ion-list>

          </div>

          <!-- ════════════════════════════
               PAGE: ALL MOVIES
          ════════════════════════════ -->
          <div v-else-if="currentPage === 'all'">
            <ion-searchbar v-model="searchQuery" placeholder="Search movies..." style="padding: 0 0 12px;" />
            <div v-if="filteredBySearch.length === 0" class="ion-text-center ion-padding">
              <ion-icon :icon="filmOutline" style="font-size: 48px; color: #ccc;" />
              <p style="color: gray;">No movies found.</p>
            </div>
            <ion-list v-else style="border-radius: 12px; overflow: hidden;">
              <ion-item
                v-for="movie in filteredBySearch"
                :key="movie.id"
                button
                @click="openMovieDetail(movie)"
              >
                <div slot="start" :style="coverStyle(movie)" style="width: 44px; height: 44px; border-radius: 8px; display: flex; align-items: center; justify-content: center;">
                  <ion-icon v-if="!movie.imageUrl" :icon="filmOutline" style="font-size: 20px; color: rgba(255,255,255,0.5);" />
                </div>
                <ion-label style="margin-left: 8px;">
                  <h3>{{ movie.title }}</h3>
                  <p>{{ movie.genre }} · {{ movie.year }} · ⭐ {{ movie.rating }}/10</p>
                </ion-label>
                <ion-badge slot="end" :color="movie.status === 'Watched' ? 'success' : 'warning'">
                  {{ movie.status }}
                </ion-badge>
              </ion-item>
            </ion-list>
          </div>

          <!-- ════════════════════════════
               PAGE: TOP RATED
          ════════════════════════════ -->
          <div v-else-if="currentPage === 'toprated'">
            <div v-if="topMovies.length === 0" class="ion-text-center ion-padding">
              <ion-icon :icon="trophyOutline" style="font-size: 48px; color: #ccc;" />
              <p style="color: gray;">No movies yet.</p>
            </div>
            <ion-list v-else style="border-radius: 12px; overflow: hidden;">
              <ion-item
                v-for="(movie, index) in topMovies"
                :key="movie.id"
                button
                @click="openMovieDetail(movie)"
              >
                <div slot="start" style="width: 28px; text-align: center;">
                  <span v-if="index === 0" style="font-size: 20px;">🥇</span>
                  <span v-else-if="index === 1" style="font-size: 20px;">🥈</span>
                  <span v-else-if="index === 2" style="font-size: 20px;">🥉</span>
                  <span v-else style="font-weight: bold; color: gray; font-size: 13px;">#{{ index + 1 }}</span>
                </div>
                <ion-label>
                  <h3>{{ movie.title }}</h3>
                  <p>{{ movie.genre }} · {{ movie.year }}</p>
                </ion-label>
                <div slot="end" style="font-size: 15px; font-weight: 700; color: var(--ion-color-warning);">
                  ⭐ {{ movie.rating }}/10
                </div>
              </ion-item>
            </ion-list>
          </div>

          <!-- ════════════════════════════
               PAGE: WATCHED
          ════════════════════════════ -->
          <div v-else-if="currentPage === 'watched'">
            <div v-if="watchedMovies.length === 0" class="ion-text-center ion-padding">
              <ion-icon :icon="eyeOutline" style="font-size: 48px; color: #ccc;" />
              <p style="color: gray;">No watched movies yet.</p>
            </div>
            <ion-list v-else style="border-radius: 12px; overflow: hidden;">
              <ion-item
                v-for="movie in watchedMovies"
                :key="movie.id"
                button
                @click="openMovieDetail(movie)"
              >
                <ion-label>
                  <h3>{{ movie.title }}</h3>
                  <p>{{ movie.genre }} · {{ movie.year }} · ⭐ {{ movie.rating }}/10</p>
                </ion-label>
                <ion-badge slot="end" color="success">Watched</ion-badge>
              </ion-item>
            </ion-list>
          </div>

          <!-- ════════════════════════════
               PAGE: WATCHLIST
          ════════════════════════════ -->
          <div v-else-if="currentPage === 'watchlist'">
            <div v-if="notWatchedMovies.length === 0" class="ion-text-center ion-padding">
              <ion-icon :icon="timeOutline" style="font-size: 48px; color: #ccc;" />
              <p style="color: gray;">Your watchlist is empty.</p>
            </div>
            <ion-list v-else style="border-radius: 12px; overflow: hidden;">
              <ion-item
                v-for="movie in notWatchedMovies"
                :key="movie.id"
                button
                @click="openMovieDetail(movie)"
              >
                <ion-label>
                  <h3>{{ movie.title }}</h3>
                  <p>{{ movie.genre }} · {{ movie.year }} · ⭐ {{ movie.rating }}/10</p>
                </ion-label>
                <ion-badge slot="end" color="warning">Not Watched</ion-badge>
              </ion-item>
            </ion-list>
          </div>

          <!-- ════════════════════════════
               PAGE: ADD / EDIT MOVIE
          ════════════════════════════ -->
          <div v-else-if="currentPage === 'add'">
            <ion-card>
              <ion-card-header>
                <ion-card-title>{{ editingId ? 'Edit Movie' : 'Add a Movie' }}</ion-card-title>
              </ion-card-header>
              <ion-card-content>

                <!-- Cover image upload -->
                <div style="text-align: center; margin-bottom: 16px;">
                  <div
                    :style="form.imageUrl ? `background-image: url(${form.imageUrl}); background-size: cover; background-position: center;` : 'background: #1a1a2e;'"
                    style="width: 120px; height: 160px; border-radius: 10px; margin: 0 auto 10px; display: flex; align-items: center; justify-content: center; cursor: pointer; position: relative;"
                    @click="triggerImageUpload"
                  >
                    <ion-icon v-if="!form.imageUrl" :icon="cameraOutline" style="font-size: 32px; color: rgba(255,255,255,0.5);" />
                    <div v-if="form.imageUrl" style="position: absolute; bottom: 0; left: 0; right: 0; background: rgba(0,0,0,0.5); border-radius: 0 0 10px 10px; padding: 4px; text-align: center;">
                      <span style="font-size: 10px; color: white;">Change photo</span>
                    </div>
                  </div>
                  <input type="file" ref="imageInput" accept="image/*" style="display: none;" @change="handleImageUpload" />
                  <ion-button fill="outline" size="small" @click="triggerImageUpload">
                    <ion-icon :icon="cameraOutline" slot="start" />
                    {{ form.imageUrl ? 'Change Cover' : 'Add Cover Photo' }}
                  </ion-button>
                </div>

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

                <ion-item>
                  <ion-label position="stacked">Your Review (optional)</ion-label>
                  <ion-textarea v-model="form.review" placeholder="Write your thoughts about this movie..." :auto-grow="true" />
                </ion-item>

                <div class="ion-margin-top" style="display: flex; gap: 8px;">
                  <ion-button expand="block" style="flex: 1;" @click="saveMovie">
                    {{ editingId ? 'Update Movie' : 'Add Movie' }}
                  </ion-button>
                  <ion-button v-if="editingId" expand="block" fill="outline" color="medium" style="flex: 1;" @click="cancelEdit">
                    Cancel
                  </ion-button>
                </div>

              </ion-card-content>
            </ion-card>
          </div>

        </div>
      </ion-content>
    </ion-page>

    <!-- ═══════════════════════════════════════════
         MOVIE DETAIL MODAL
    ═══════════════════════════════════════════ -->
    <ion-modal :is-open="showDetail" @did-dismiss="closeDetail">
      <ion-header>
        <ion-toolbar color="primary">
          <ion-title>{{ selectedMovie?.title }}</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="closeDetail">
              <ion-icon :icon="closeOutline" slot="icon-only" />
            </ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>

      <ion-content v-if="selectedMovie">

        <!-- Cover image -->
        <div
          :style="coverStyle(selectedMovie)"
          style="height: 200px; display: flex; align-items: flex-end; padding: 16px; position: relative;"
        >
          <div v-if="!selectedMovie.imageUrl" style="position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;">
            <ion-icon :icon="filmOutline" style="font-size: 64px; color: rgba(255,255,255,0.3);" />
          </div>
          <div style="position: relative; z-index: 1;">
            <ion-badge :color="selectedMovie.status === 'Watched' ? 'success' : 'warning'" style="margin-right: 6px;">
              {{ selectedMovie.status }}
            </ion-badge>
            <ion-badge color="dark">{{ selectedMovie.genre }}</ion-badge>
          </div>
        </div>

        <div class="ion-padding">

          <!-- Movie info -->
          <h2 style="font-size: 22px; font-weight: 700; margin: 0 0 4px;">{{ selectedMovie.title }}</h2>
          <p style="color: gray; margin: 0 0 12px;">{{ selectedMovie.year }} · ⭐ {{ selectedMovie.rating }}/10</p>

          <!-- Action buttons -->
          <div style="display: flex; gap: 8px; margin-bottom: 20px;">
            <ion-button fill="outline" size="small" @click="startEdit(selectedMovie)">
              <ion-icon :icon="createOutline" slot="start" />
              Edit
            </ion-button>
            <ion-button fill="outline" color="danger" size="small" @click="deleteMovie(selectedMovie.id)">
              <ion-icon :icon="trashOutline" slot="start" />
              Delete
            </ion-button>
          </div>

          <!-- Owner review -->
          <div v-if="selectedMovie.review" style="background: var(--ion-color-light); border-radius: 10px; padding: 12px; margin-bottom: 20px;">
            <p style="font-size: 12px; font-weight: 600; color: gray; margin: 0 0 4px;">📝 OWNER REVIEW</p>
            <p style="font-size: 14px; margin: 0;">{{ selectedMovie.review }}</p>
          </div>

          <!-- ── COMMENTS SECTION ── -->
          <p style="font-weight: 600; font-size: 16px; margin-bottom: 12px;">
            💬 Community Reviews ({{ movieComments.length }})
          </p>

          <!-- Existing comments -->
          <div v-if="movieComments.length === 0" style="color: gray; font-size: 13px; margin-bottom: 16px;">
            No reviews yet. Be the first to leave one!
          </div>
          <div v-else style="margin-bottom: 16px;">
            <div
              v-for="comment in movieComments"
              :key="comment.id"
              style="background: var(--ion-color-light); border-radius: 10px; padding: 12px; margin-bottom: 8px;"
            >
              <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px;">
                <span style="font-weight: 600; font-size: 13px;">👤 {{ comment.name }}</span>
                <span style="font-size: 11px; color: gray;">{{ comment.date }}</span>
              </div>
              <p style="font-size: 13px; margin: 0;">{{ comment.text }}</p>
            </div>
          </div>

          <!-- Add comment form -->
          <ion-card>
            <ion-card-header>
              <ion-card-title style="font-size: 15px;">Leave a Review</ion-card-title>
            </ion-card-header>
            <ion-card-content>
              <ion-item>
                <ion-label position="stacked">Your Name *</ion-label>
                <ion-input v-model="commentName" placeholder="e.g. Juan" />
              </ion-item>
              <ion-item>
                <ion-label position="stacked">Your Review *</ion-label>
                <ion-textarea v-model="commentText" placeholder="What did you think of this movie?" :auto-grow="true" />
              </ion-item>
              <ion-button expand="block" class="ion-margin-top" @click="submitComment">
                <ion-icon :icon="sendOutline" slot="start" />
                Post Review
              </ion-button>
            </ion-card-content>
          </ion-card>

        </div>
      </ion-content>
    </ion-modal>

  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonMenu, IonMenuButton, IonButtons, IonButton,
  IonList, IonItem, IonItemDivider, IonLabel, IonIcon,
  IonCard, IonCardHeader, IonCardTitle, IonCardContent,
  IonInput, IonSelect, IonSelectOption, IonTextarea,
  IonBadge, IonSearchbar, IonSpinner, IonModal,
  alertController, toastController, menuController
} from '@ionic/vue';
import {
  homeOutline, listOutline, trophyOutline, eyeOutline,
  timeOutline, addCircleOutline, addOutline, filmOutline,
  createOutline, trashOutline, closeOutline, cameraOutline,
  sendOutline
} from 'ionicons/icons';

import { db } from '@/firebase';
import {
  ref as dbRef,
  push,
  onValue,
  update,
  remove
} from 'firebase/database';

// ── TYPES ──────────────────────────────────────────
interface Comment {
  id: string;
  name: string;
  text: string;
  date: string;
}

interface Movie {
  id: string;
  title: string;
  genre: string;
  year: number;
  rating: number;
  status: 'Watched' | 'Not Watched';
  review?: string;
  imageUrl?: string;
  comments?: Record<string, Omit<Comment, 'id'>>;
}

// ── STATE ───────────────────────────────────────────
const movies = ref<Movie[]>([]);
const loading = ref(true);
const currentPage = ref('home');
const searchQuery = ref('');
const editingId = ref<string | null>(null);

// Detail modal
const showDetail = ref(false);
const selectedMovie = ref<Movie | null>(null);
const movieComments = ref<Comment[]>([]);

// Comment form
const commentName = ref('');
const commentText = ref('');

// Image upload
const imageInput = ref<HTMLInputElement | null>(null);

// Add/Edit form
const form = ref({
  title: '',
  genre: '',
  year: new Date().getFullYear(),
  rating: 8,
  status: 'Not Watched' as 'Watched' | 'Not Watched',
  review: '',
  imageUrl: ''
});

// ── PAGE TITLES ─────────────────────────────────────
const pageTitles: Record<string, string> = {
  home: '🎬 MOVIERS',
  all: 'All Movies',
  toprated: 'Top Rated',
  watched: 'Watched',
  watchlist: 'Watchlist',
  add: 'Add Movie'
};

const pageTitle = computed(() => pageTitles[currentPage.value] || 'CineList');

// ── COMPUTED ────────────────────────────────────────
const watchedMovies = computed(() => movies.value.filter(m => m.status === 'Watched'));
const notWatchedMovies = computed(() => movies.value.filter(m => m.status === 'Not Watched'));
const watchedCount = computed(() => watchedMovies.value.length);
const notWatchedCount = computed(() => notWatchedMovies.value.length);
const topMovies = computed(() => [...movies.value].sort((a, b) => b.rating - a.rating));
const recentMovies = computed(() => movies.value.slice(-4).reverse());
const filteredBySearch = computed(() => {
  const q = searchQuery.value.toLowerCase();
  if (!q) return movies.value;
  return movies.value.filter(m =>
    m.title.toLowerCase().includes(q) ||
    m.genre.toLowerCase().includes(q)
  );
});

// ── COVER STYLE ─────────────────────────────────────
function coverStyle(movie: Movie) {
  if (movie.imageUrl) {
    return `background-image: url(${movie.imageUrl}); background-size: cover; background-position: center;`;
  }
  const colors: Record<string, string> = {
    'Action': 'linear-gradient(135deg, #1a1a2e, #e94560)',
    'Comedy': 'linear-gradient(135deg, #f7971e, #ffd200)',
    'Drama': 'linear-gradient(135deg, #232526, #414345)',
    'Horror': 'linear-gradient(135deg, #0f0c29, #302b63)',
    'Romance': 'linear-gradient(135deg, #f953c6, #b91d73)',
    'Sci-Fi': 'linear-gradient(135deg, #1a1a2e, #16213e)',
    'Thriller': 'linear-gradient(135deg, #141e30, #243b55)',
    'Animation': 'linear-gradient(135deg, #11998e, #38ef7d)',
    'Documentary': 'linear-gradient(135deg, #373b44, #4286f4)',
    'Fantasy': 'linear-gradient(135deg, #4776e6, #8e54e9)'
  };
  return `background: ${colors[movie.genre] || 'linear-gradient(135deg, #1a1a2e, #16213e)'};`;
}

// ── NAVIGATION ───────────────────────────────────────
async function navigate(page: string) {
  currentPage.value = page;
  await menuController.close();
}

// ── READ: Firebase real-time listener ───────────────
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

// ── MOVIE DETAIL MODAL ──────────────────────────────
function openMovieDetail(movie: Movie) {
  selectedMovie.value = movie;
  // Load comments for this movie
  if (movie.comments) {
    movieComments.value = Object.entries(movie.comments).map(([id, c]: [string, any]) => ({
      id,
      ...c
    }));
  } else {
    movieComments.value = [];
  }
  commentName.value = '';
  commentText.value = '';
  showDetail.value = true;
}

function closeDetail() {
  showDetail.value = false;
  selectedMovie.value = null;
  movieComments.value = [];
}

// ── IMAGE UPLOAD ────────────────────────────────────
function triggerImageUpload() {
  imageInput.value?.click();
}

function handleImageUpload(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (e) => {
    form.value.imageUrl = e.target?.result as string;
  };
  reader.readAsDataURL(file);
}

// ── VALIDATE ────────────────────────────────────────
function isValid(): boolean {
  if (!form.value.title.trim()) return false;
  if (!form.value.genre) return false;
  if (!form.value.year || form.value.year < 1888 || form.value.year > 2100) return false;
  if (!form.value.rating || form.value.rating < 1 || form.value.rating > 10) return false;
  return true;
}

// ── CREATE / UPDATE ──────────────────────────────────
async function saveMovie() {
  if (!isValid()) {
    const alert = await alertController.create({
      header: 'Missing Information',
      message: 'Please fill in all required fields. Rating must be 1–10.',
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
    status: form.value.status,
    review: form.value.review.trim(),
    imageUrl: form.value.imageUrl
  };

  if (editingId.value) {
    await update(dbRef(db, `movies/${editingId.value}`), movieData);
    showToast('Movie updated!', 'success');
    editingId.value = null;
  } else {
    await push(dbRef(db, 'movies'), movieData);
    showToast('Movie added!', 'success');
  }

  resetForm();
  navigate('all');
}

// ── EDIT ─────────────────────────────────────────────
function startEdit(movie: Movie) {
  closeDetail();
  editingId.value = movie.id;
  form.value = {
    title: movie.title,
    genre: movie.genre,
    year: movie.year,
    rating: movie.rating,
    status: movie.status,
    review: movie.review || '',
    imageUrl: movie.imageUrl || ''
  };
  navigate('add');
}

function cancelEdit() {
  editingId.value = null;
  resetForm();
  navigate('all');
}

// ── DELETE ───────────────────────────────────────────
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
          closeDetail();
          showToast('Movie deleted.', 'danger');
        }
      }
    ]
  });
  await alert.present();
}

// ── COMMENTS ─────────────────────────────────────────
async function submitComment() {
  if (!commentName.value.trim() || !commentText.value.trim()) {
    const alert = await alertController.create({
      header: 'Missing Fields',
      message: 'Please enter your name and review.',
      buttons: ['OK']
    });
    await alert.present();
    return;
  }

  if (!selectedMovie.value) return;

  const comment = {
    name: commentName.value.trim(),
    text: commentText.value.trim(),
    date: new Date().toLocaleDateString('en-PH', { year: 'numeric', month: 'short', day: 'numeric' })
  };

  await push(dbRef(db, `movies/${selectedMovie.value.id}/comments`), comment);

  // Add to local list immediately so user sees it right away
  movieComments.value.push({ id: Date.now().toString(), ...comment });

  commentName.value = '';
  commentText.value = '';
  showToast('Review posted!', 'success');
}

// ── HELPERS ──────────────────────────────────────────
function resetForm() {
  form.value = {
    title: '',
    genre: '',
    year: new Date().getFullYear(),
    rating: 8,
    status: 'Not Watched',
    review: '',
    imageUrl: ''
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