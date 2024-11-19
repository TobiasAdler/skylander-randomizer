<template>
  <div id="app">
    <h1>Character Picker</h1>

    <!-- Button, um die Besitzliste ein-/auszublenden -->
    <button @click="toggleOwnedCharacters">
      {{ showOwnedCharacters ? 'Besitzte Charaktere ausblenden' : 'Besitzte Charaktere anzeigen' }}
    </button>

    <!-- Checkbox-Liste (nach Spielen und Elementen gruppiert) -->
    <div v-if="showOwnedCharacters" class="character-setup">
      <h2>Besitzte Charaktere</h2>
      <div v-for="(elements, game) in charactersByGame" :key="game" class="game-group">
        <h3>Spiel {{ game }}</h3>
        <div v-for="(characters, element) in elements" :key="element" class="element-group">
          <h4>{{ element }}</h4>
          <div v-for="character in characters" :key="character.name" class="character-checkbox">
            <label>
              <input 
                type="checkbox" 
                v-model="ownedCharacters" 
                :value="character.name" 
              />
              <img :src="character.img" :alt="character.name" class="character-image" />
              {{ character.name }}
            </label>
          </div>
        </div>
      </div>
      <button @click="saveOwnedCharacters">Speichern</button>
    </div>

    <!-- Filter und Zufallsgenerator -->
    <div class="filters">
      <label>
        Wähle ein Spiel:
        <select v-model="selectedGame">
          <option v-for="game in uniqueGames" :key="game" :value="game">{{ game }}</option>
        </select>
      </label>
      <button @click="pickRandomCharacter">Zufälligen Charakter wählen</button>
    </div>

    <!-- Anzeige des zufällig gewählten Charakters -->
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
      characters, // JSON-Daten laden
      selectedGame: null,
      selectedCharacter: null,
      ownedCharacters: [], // Liste der besitzten Charaktere
      showOwnedCharacters: false, // Steuert die Sichtbarkeit der Checkbox-Liste
    };
  },
  computed: {
    uniqueGames() {
      return [...new Set(this.characters.map((char) => char.game))].sort((a, b) => a - b);
    },
    charactersByGame() {
      // Gruppiere Charaktere nach Spiel und dann nach Element
      return this.characters.reduce((gameGroups, character) => {
        if (!gameGroups[character.game]) {
          gameGroups[character.game] = {};
        }
        if (!gameGroups[character.game][character.element]) {
          gameGroups[character.game][character.element] = [];
        }
        gameGroups[character.game][character.element].push(character);
        return gameGroups;
      }, {});
    },
  },
  methods: {
    toggleOwnedCharacters() {
      // Sichtbarkeit der Besitzliste toggeln
      this.showOwnedCharacters = !this.showOwnedCharacters;
    },
    pickRandomCharacter() {
      // Filtere Charaktere nach Spiel und Besitzstatus
      const filteredCharacters = this.characters.filter(
        (char) => char.game <= this.selectedGame && this.ownedCharacters.includes(char.name)
      );
      if (filteredCharacters.length > 0) {
        // Zufälligen Charakter auswählen
        const randomIndex = Math.floor(Math.random() * filteredCharacters.length);
        this.selectedCharacter = filteredCharacters[randomIndex];
      } else {
        alert("Keine Charaktere für das gewählte Spiel gefunden oder nicht besessen!");
      }
    },
    saveOwnedCharacters() {
      // Speichere die Liste der besitzten Charaktere im localStorage
      localStorage.setItem('ownedCharacters', JSON.stringify(this.ownedCharacters));
      alert('Besitzte Charaktere gespeichert!');
    },
    loadOwnedCharacters() {
      // Lade die Liste der besitzten Charaktere aus dem localStorage
      const storedCharacters = localStorage.getItem('ownedCharacters');
      if (storedCharacters) {
        this.ownedCharacters = JSON.parse(storedCharacters);
      }
    },
  },
  mounted() {
    // Lade die gespeicherten Charaktere beim Start der Anwendung
    this.loadOwnedCharacters();
  },
};
</script>

<style>
/* Einfaches Styling */
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin: 20px;
}

button {
  margin: 10px;
  padding: 10px 15px;
  cursor: pointer;
}

.character-setup {
  margin: 20px auto;
  text-align: left;
  display: inline-block;
}

.game-group {
  margin-bottom: 15px;
}

.element-group {
  margin-left: 20px;
}

.character-checkbox {
  margin: 5px 0;
  display: flex;
  align-items: center;
}

.character-checkbox img {
  width: 40px;
  height: 40px;
  margin-right: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.character-display img {
  max-width: 200px;
  height: auto;
}
</style>
