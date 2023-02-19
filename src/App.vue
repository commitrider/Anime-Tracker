<script setup>

import { ref, computed, onMounted } from 'vue'

const searchQuery = ref('')
const myAnime = ref([])
const searchResults = ref([])

// Order anime list by ascending order
const myAnimeAscending = computed(() => {
    return myAnime.value.sort((a,b) => {
        return a.title.localeCompare(b.title)
    })
})

// Search anime
const searchAnime = () => {
    const url = `https://api.jikan.moe/v4/anime?q=${searchQuery.value}`
    fetch( url )
        .then( res => res.json() )
        .then( res => {
            searchResults.value = res.data
        })
}

// Search bar functionality
const handleInput = e => {
    if ( !e.target.value ) {
        searchResults.value = []
    }
}

// Add anime to our list
const addAnime = anime => {
    searchResults.value = []
    searchQuery.value = ''

    myAnime.value.push({
        id: anime.mal_id,
        title: anime.title,
        image: anime.images.jpg.image_url,
        total_episodes: anime.episodes,
        watched_episodes: 0
    })

    localStorage.setItem('my_anime', JSON.stringify(myAnime.value))
}

// Increase episode count
const increaseWatch = anime => {
    anime.watched_episodes++
    localStorage.setItem('my_anime', JSON.stringify(myAnime.value))
}

// Decrease episode count
const decreaseWatch = anime => {
    anime.watched_episodes--
    localStorage.setItem('my_anime', JSON.stringify(myAnime.value))
}

// Obtain anime list from localStorage on onMounted
onMounted(() => {
    myAnime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})

</script>


<template>

    <main>
        <h1>My Anime Tracker</h1>

        <form @submit.prevent="searchAnime">
            <input type="text"
                   placeholder="Search for an anime..."
                   v-model="searchQuery"
                   @input="handleInput"
            >

            <button type="submit" class="button search">Search</button>
        </form>

        <div v-if="searchResults.length > 0" class="results">
            <div class="result" v-for="anime in searchResults" :key="anime.mal_id">
                <img :src="anime.images.jpg.image_url" alt="anime.title">

                <div class="details">
                    <h3>{{ anime.title }}</h3>

                    <p v-if="anime.synopsis" :title="anime.synopsis">
                        {{ anime.synopsis.slice(0,120) }}
                    </p>

                    <span class="flex-1"></span>
                    <button @click="addAnime(anime)" class="button">Add to my anime</button>
                </div>

            </div>
        </div>

        <div v-if="myAnime.length > 0" class="myanime">
            <h2>My Anime</h2>

            <div v-for="anime in myAnimeAscending" class="anime">
                <img :src="anime.image" alt="anime.title">

                <h3>{{ anime.title }}</h3>
                <div class="flex-1"></div>
                <span class="episodes">
                    {{ anime.watched_episodes }} / {{ anime.total_episodes }}
                </span>

                <button v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)" class="button">-</button>
                <button v-if="anime.total_episodes !== anime.watched_episodes" @click="increaseWatch(anime)" class="button">+</button>
            </div>
        </div>

    </main>

</template>


<style>
* {
	box-sizing: border-box;
	font-family: 'Fira Sans', sans-serif;
	margin: 0;
	padding: 0;
}

body { background-color: #fff; }

main {
	margin: 0 auto;
	max-width: 768px;
	padding: 1.5rem;
}

h1 {
	margin-bottom: 1.5rem;
	text-align: center;
}

form {
	display: flex;
	margin: 0 auto 1.5rem;
	max-width: 480px;
}
form input {
	appearance: none;
	background: #ecebeb;
	border: none;
	color: #616161;
	display: block;
	font-size: 1.125rem;
	outline: none;
	padding: 0.5rem 1rem;
	width: 100%;
}

.button {
	appearance: none;
    background: none;
	background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
	background-size: 200%;
	border: none;
	color: white;
	cursor: pointer;
	display: block;
	font-size: 1.125rem;
	font-weight: bold;
	outline: none;
	padding: 0.5rem 1rem;
	text-transform: uppercase;
	transition: 0.4s;
}
.button.search {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
}
.button:hover { background-position: right; }

.results {
	background-color: #fff;
	border-radius: 0.5rem;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	margin-bottom: 1.5rem;
	max-height: 480px;
	overflow-y: scroll;
}

.result {
    border: 1px solid #ccc;
	border-radius: 5px;
	display: flex;
	margin: 1rem;
	padding: 1rem;
	transition: 0.4s;
}
.result img {
	border-radius: 1rem;
	height: 72px;
	margin-right: 1rem;
	object-fit: cover;
    width: 72px;
}

.details {
	display: flex;
	align-items: flex-start;
    flex: 1 1 0%;
	flex-direction: column;
}
.details h3 {
	font-size: 1.25rem;
	margin-bottom: 0.5rem;
}
.details p {
	font-size: 0.875rem;
	margin-bottom: 1rem;
    text-align: left;
}
.details .button { margin-left: auto; }

.flex-1 {
	display: block;
	flex: 1 1 0%;
}

.myanime h2 {
	color: #626161;
	font-weight: 400;
	margin-bottom: 1.5rem;
}

.myanime .anime {
	display: flex;
	align-items: center;
	background-color: #FFF;
	border-radius: 0.5rem;
	box-shadow: 0px 2px 7px rgb(0 0 0 / 40%);
	margin-bottom: 1.5rem;
	padding: 1rem;
}

.anime img {
	width: 72px;
	height: 72px;
	object-fit: cover;
	border-radius: 1rem;
	margin-right: 1rem;
}
.anime h3 {
	color: #626161;
	font-size: 1.125rem;
}
.anime .episodes {
	color: ##626161;
	margin: 0 1rem;
    white-space: nowrap;
}
.anime .button:first-of-type { margin-right: 0.5rem; }
.anime .button:last-of-type { margin-right: 0; }

</style>
