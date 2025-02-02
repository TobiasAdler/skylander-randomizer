<template>
    <div id="app">
        <div><img src="images/other/Skylanders_Logo.webp" class="headline-image" alt="Skylanders"/></div>
        <div id="headline"><h2>Character Randomizer</h2></div>

        <div class="filters">
            <label>
                <select v-model="selectedGame">
                    <option value="" disabled selected hidden>Choose A Game</option>
                    <option v-for="game in uniqueGames" :key="game" :value="game">{{ getNameOfGame(game) }}</option>
                </select>
            </label>
        </div>

        <button @click="pickRandomCharacter">Choose Random Character</button>

        <div v-if="selectedCharacter" class="character-display">
            <img :src="selectedCharacter.img" alt="Character Image"/>
            <h2>{{ selectedCharacter.name }}</h2>
        </div>

        <div v-if="selectedGame" class="element-buttons">
            <button
                v-for="element in elements"
                :key="element"
                @click="pickRandomCharacterByElement(element)"
                class="element-btn">
                <img :src="`images/other/${capitalizeFirstLetter(element)}.webp`" :alt="element" />
            </button>
        </div>

        <div>
            <button @click="toggleOwnedCharacters">
                {{ showOwnedCharacters ? 'Hide character List' : 'Show Character List' }}
            </button>
        </div>

        <div v-if="showOwnedCharacters" class="character-setup">
            <h2>All Characters</h2>
            <div v-for="(elements, game) in charactersByGame" :key="game" class="game-group">
                <h3>Skylanders: {{ getNameOfGame(game) }}</h3>
                <label>
                    <input
                        type="checkbox"
                        :checked="areAllCharactersChecked(game)"
                        @change="toggleAllCharactersForGame(game, $event.target.checked)"
                    />
                    Complete {{ getNameOfGame(game) }} collection
                </label>
                <div v-for="(characters, element) in elements" :key="element" class="element-group">
                    <h4>{{ capitalizeFirstLetter(element) }}</h4>
                    <div v-for="character in characters" :key="character.name" class="character-checkbox">
                        <label>
                            <input
                                type="checkbox"
                                v-model="ownedCharacters"
                                :value="character.name"
                            />
                            <img :src="character.img" :alt="character.name" class="character-image"/>
                            {{ character.name }}
                        </label>
                    </div>
                </div>
            </div>
            <button @click="saveOwnedCharacters">Save</button>
        </div>

    </div>
</template>

<script>
import characters from './assets/characters.json';

export default {
    data() {
        return {
            characters,
            selectedGame: "",
            selectedCharacter: null,
            ownedCharacters: [],
            showOwnedCharacters: false,
            elements: ['fire', 'water', 'air', 'earth', 'magic', 'tech', 'life', 'undead', 'light', 'dark']
        };
    },
    computed: {
        uniqueGames() {
            return [...new Set(this.characters.map((char) => char.game))].sort((a, b) => a - b);
        },
        charactersByGame() {
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
        getNameOfGame(gameId) {
            const games = {
                1: "Spyros Adventure",
                2: "Giants",
                3: "Swap Force",
                4: "Trap Team",
                5: "Superchargers"
            };
            return games[gameId] || "Imaginators";
        },
        areAllCharactersChecked(game) {
            const charactersInGame = this.getCharactersInGame(game);
            return charactersInGame.every((character) =>
                this.ownedCharacters.includes(character.name)
            );
        },
        toggleAllCharactersForGame(game, isChecked) {
            const charactersInGame = this.getCharactersInGame(game);
            if (isChecked) {
                charactersInGame.forEach((character) => {
                    if (!this.ownedCharacters.includes(character.name)) {
                        this.ownedCharacters.push(character.name);
                    }
                });
            } else {
                this.ownedCharacters = this.ownedCharacters.filter(
                    (characterName) =>
                        !charactersInGame.some(
                            (character) => character.name === characterName
                        )
                );
            }
        },
        getCharactersInGame(game) {
            const elements = this.charactersByGame[game];
            return Object.values(elements).flat();
        },
        capitalizeFirstLetter(str) {
            if (!str) return "";
            return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
        },
        toggleOwnedCharacters() {
            this.showOwnedCharacters = !this.showOwnedCharacters;
        },
        pickRandomCharacter() {
            const filteredCharacters = this.characters.filter(
                (char) => char.game <= this.selectedGame && this.ownedCharacters.includes(char.name)
            );
            if (filteredCharacters.length > 0) {
                const randomIndex = Math.floor(Math.random() * filteredCharacters.length);
                this.selectedCharacter = filteredCharacters[randomIndex];
            } else {
                alert("No game selected.");
            }
        },
        pickRandomCharacterByElement(element) {
            if (!this.selectedGame) {
                alert("Select a game first.");
                return;
            }
            const filteredCharacters = this.characters.filter(
                (char) =>
                    char.game <= this.selectedGame &&
                    char.element === element &&
                    this.ownedCharacters.includes(char.name)
            );
            if (filteredCharacters.length > 0) {
                const randomIndex = Math.floor(Math.random() * filteredCharacters.length);
                this.selectedCharacter = filteredCharacters[randomIndex];
            } else {
                alert(`No character for element: ${this.capitalizeFirstLetter(element)}`);
            }
        },
        saveOwnedCharacters() {
            localStorage.setItem('ownedCharacters', JSON.stringify(this.ownedCharacters));
            alert('Owned characters saved.');
        },
        loadOwnedCharacters() {
            const storedCharacters = localStorage.getItem('ownedCharacters');
            if (storedCharacters) {
                this.ownedCharacters = JSON.parse(storedCharacters);
            }
        },
    },
    mounted() {
        this.loadOwnedCharacters();
    },
};
</script>

<style>
#app {
    font-family: Arial, sans-serif;
    text-align: center;
    margin: 20px;
}

.headline-image {
    height: auto;
    width: 30em;
}

@media only screen and (max-width: 600px) {
    .headline-image {
        width: 15em;
    }
}

select option[disabled] {
    color: gray;
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
.element-buttons {
    margin: 20px 0;
}

.element-buttons p {
    margin-bottom: 10px;
}

.element-buttons button.element-btn {
    width: 50px;
    height: 50px;
    margin: 5px;
    padding: 0;
    border: none;
    background-color: transparent;
    cursor: pointer;
}

.element-buttons button.element-btn img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

</style>
