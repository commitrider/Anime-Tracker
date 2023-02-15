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

            <button type="submit">Search</button>
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
                    <button @click="addAnime(anime)">Add to my anime</button>
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

                <button v-if="anime.total_episodes !== anime.watched_episodes" @click="increaseWatch(anime)">+</button>
                <button v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)">-</button>
            </div>
        </div>

    </main>

</template>


<style>
body {
    background-color: #fff;
}
</style>
