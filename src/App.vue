<template>
  <div class="app-container">
    <header class="main-header">
      <div class="brand">
        <img src="./assets/logo.png" alt="Cuebuena logo" class="brand-logo" />
      </div>
      <button @click="viewFavorites = !viewFavorites" class="btn-fav-toggle">
        {{ viewFavorites ? '🏠 Ir al Inicio' : '⭐ Mis Favoritos (' + favorites.length + ')' }}
      </button>
    </header>

    <main class="main-content">
      
      <section v-if="viewFavorites">
        <h2 class="section-title">Mis Películas Favoritas</h2>
        <div v-if="favorites.length === 0" class="empty-state">
          <p>No agregaste ninguna película a tus favoritos todavía.</p>
        </div>
        
        <div v-else class="movies-grid">
          <div 
            v-for="movie in favorites" 
            :key="movie.id" 
            class="movie-card"
            @click="selectMovie(movie)"
          >
            <img 
              :src="movie.poster_path ? 'https://image.tmdb.org/t/p/w500' + movie.poster_path : 'https://via.placeholder.com/500x750?text=No+Image'" 
              :alt="movie.title" 
            />
            <div class="movie-info">
              <h3>{{ movie.title }}</h3>
              <span class="rating">⭐ {{ movie.vote_average.toFixed(1) }}</span>
            </div>
          </div>
        </div>
      </section>

      <section v-else>
        <div class="search-filter-container">
          <input 
            v-model="searchQuery" 
            @input="handleSearch" 
            type="text" 
            placeholder="Buscar película por título..." 
            class="search-input"
          />
          
          <select v-model="selectedGenre" @change="handleFilter" class="filter-select">
            <option value="">Todos los géneros</option>
            <option v-for="genre in genres" :key="genre.id" :value="genre.id">
              {{ genre.name }}
            </option>
          </select>
        </div>

        <h2 class="section-title">
          {{ searchQuery ? 'Resultados de Búsqueda' : (selectedGenre ? 'Resultados por Género' : 'Películas Populares') }}
        </h2>

        <div v-if="movies.length === 0" class="empty-state">
          <p>No se encontraron películas.</p>
        </div>
        
        <div v-else class="movies-grid">
          <div 
            v-for="movie in movies" 
            :key="movie.id" 
            class="movie-card"
            @click="selectMovie(movie)"
          >
            <img 
              :src="movie.poster_path ? 'https://image.tmdb.org/t/p/w500' + movie.poster_path : 'https://via.placeholder.com/500x750?text=No+Image'" 
              :alt="movie.title" 
            />
            <div class="movie-info">
              <h3>{{ movie.title }}</h3>
              <span class="rating">⭐ {{ movie.vote_average.toFixed(1) }}</span>
            </div>
          </div>
        </div>
      </section>
    </main>

    <div v-if="selectedMovie" class="modal-overlay" @click.self="selectedMovie = null">
      <div class="modal-content">
        <button class="close-btn" @click="selectedMovie = null">✕</button>
        
        <img 
          :src="selectedMovie.poster_path ? 'https://image.tmdb.org/t/p/w500' + selectedMovie.poster_path : 'https://via.placeholder.com/500x750?text=No+Image'" 
          :alt="selectedMovie.title" 
        />
        
        <div class="modal-body">
          <h2>{{ selectedMovie.title }}</h2>
          <p class="release-date">📅 Año: {{ selectedMovie.release_date ? selectedMovie.release_date.split('-')[0] : 'Desconocido' }}</p>
          <p class="rating-detail">⭐ Puntuación: {{ selectedMovie.vote_average.toFixed(1) }} / 10</p>
          <p class="overview">{{ selectedMovie.overview || 'Sin sinopsis disponible.' }}</p>
          
          <button 
            @click="toggleFavorite(selectedMovie)" 
            :class="['btn-favorite', isFavorite(selectedMovie.id) ? 'is-fav' : '']"
          >
            {{ isFavorite(selectedMovie.id) ? '⭐ Quitar de Favoritos' : '➕ Agregar a Favoritos' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      movies: [],
      genres: [],
      searchQuery: '',
      selectedGenre: '',
      selectedMovie: null,
      favorites: JSON.parse(localStorage.getItem('davinci-movies-favs')) || [],
      viewFavorites: false,
      apiKey: 'a4aa33f44ba6301db4533947f03d98f6', 
      baseUrl: 'https://api.themoviedb.org/3'
    };
  },
  mounted() {
    this.fetchGenres();
    this.fetchPopularMovies();
  },
  methods: {
    async fetchPopularMovies() {
      try {
        const response = await fetch(`${this.baseUrl}/movie/popular?api_key=${this.apiKey}&language=es-ES&page=1`);
        const data = await response.json();
        this.movies = data.results || [];
      } catch (error) {
        console.error("Error populares:", error);
      }
    },

    async fetchGenres() {
      try {
        const response = await fetch(`${this.baseUrl}/genre/movie/list?api_key=${this.apiKey}&language=es-ES`);
        const data = await response.json();
        this.genres = data.genres || [];
      } catch (error) {
        console.error("Error géneros:", error);
      }
    },

    async handleSearch() {
      if (!this.searchQuery.trim()) {
        this.selectedGenre = '';
        this.fetchPopularMovies();
        return;
      }
      try {
        const response = await fetch(`${this.baseUrl}/search/movie?api_key=${this.apiKey}&query=${encodeURIComponent(this.searchQuery)}&language=es-ES`);
        const data = await response.json();
        this.movies = data.results || [];
        this.selectedGenre = '';
      } catch (error) {
        console.error("Error búsqueda:", error);
      }
    },

    async handleFilter() {
      if (!this.selectedGenre) {
        this.fetchPopularMovies();
        return;
      }
      try {
        const response = await fetch(`${this.baseUrl}/discover/movie?api_key=${this.apiKey}&with_genres=${this.selectedGenre}&language=es-ES&sort_by=popularity.desc`);
        const data = await response.json();
        this.movies = data.results || [];
        this.searchQuery = '';
      } catch (error) {
        console.error("Error filtro:", error);
      }
    },

    selectMovie(movie) {
      this.selectedMovie = movie;
    },

    toggleFavorite(movie) {
      const index = this.favorites.findIndex(fav => fav.id === movie.id);
      if (index > -1) {
        this.favorites.splice(index, 1);
      } else {
        this.favorites.push(movie);
      }
      localStorage.setItem('davinci-movies-favs', JSON.stringify(this.favorites));
    },

    isFavorite(id) {
      return this.favorites.some(fav => fav.id === id);
    }
  }
};
</script>