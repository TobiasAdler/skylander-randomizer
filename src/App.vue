<template>
    <div id="app">
      <h1>Character Picker</h1>
      <div class="filters">
        <label>
          Wähle ein Spiel:
          <select v-model="selectedGame">
            <option v-for="game in uniqueGames" :key="game" :value="game">{{ game }}</option>
          </select>
        </label>
        <button @click="pickRandomCharacter">Zufälligen Charakter wählen</button>
      </div>
      
      <div v-if="selectedCharacter" class="character-display">
        <h2>{{ selectedCharacter.name }}</h2>
        <p>Element: {{ selectedCharacter.element }}</p>
        <p>Typ: {{ selectedCharacter.type }}</p>
        <img :src="selectedCharacter.img" alt="Character Image" />
      </div>
    </div>
  </template>
  
  <script>
  import characters from './assets/characters.json';
  
  export default {
    data() {
      return {
        characters,
        selectedGame: null,
        selectedCharacter: null,
      };
    },
    computed: {
      uniqueGames() {
        return [...new Set(this.characters.map((char) => char.game))].sort((a, b) => a - b);
      },
    },
    methods: {
      pickRandomCharacter() {
        // Filtere die Charaktere nach aufwärtskompatiblen Spielen
        const filteredCharacters = this.characters.filter(
          (char) => char.game <= this.selectedGame
        );
        if (filteredCharacters.length > 0) {
          // Zufälligen Charakter auswählen
          const randomIndex = Math.floor(Math.random() * filteredCharacters.length);
          this.selectedCharacter = filteredCharacters[randomIndex];
        } else {
          alert("Keine Charaktere für das gewählte Spiel gefunden!");
        }
      },
    },
  };
  </script>
  
  <style>
  #app {
    font-family: Arial, sans-serif;
    text-align: center;
    margin: 20px;
  }
  .filters {
    margin: 20px;
  }
  .character-display img {
    max-width: 200px;
    height: auto;
  }
  </style>
  