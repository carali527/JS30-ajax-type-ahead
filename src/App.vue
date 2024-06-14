<template>
  <form class="search-form">
    <input type="text" v-model="searchQuery" @input="displayMatches" class="search" placeholder="Search for a city or state" />
    <ul class="suggestions">
      <li v-for="place in matches" :key="place.city + place.state">
        <span>
          <span class="name" v-html="highlightMatch(place.city, searchQuery)"></span>,
          <span class="name" v-html="highlightMatch(place.state, searchQuery)"></span>
        </span>
        <span class="population">{{ numberWithCommas(place.population) }}</span>
      </li>
      <li v-show="searchQuery && matches.length === 0">No matches found</li>
    </ul>
  </form>
</template>

<script>
import { ref, onMounted } from 'vue';
import debounce from 'lodash/debounce';

export default {
  setup() {
    const endpoint = '/js30-ajax-type-ahead/data/data.json';
    const cities = ref([]);
    const searchQuery = ref('');
    const matches = ref([]);

    onMounted(() => {
      fetch(endpoint)
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok ' + response.statusText);
          }
          return response.json();
        })
        .then(data => {
          cities.value = data;
        })
        .catch(error => {
          console.error('Error fetching data:', error);
        });
    });

    function findMatches(wordToMatch) {
      return cities.value.filter(place => {
        const regex = new RegExp(wordToMatch, 'gi');
        return place.city.match(regex) || place.state.match(regex);
      });
    }

    function numberWithCommas(x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    }

    function highlightMatch(text, query) {
      const regex = new RegExp(query, 'gi');
      return text.replace(regex, match => `<span>${match}</span>`);
    }

    const displayMatches = debounce(() => {
      if (searchQuery.value.trim() === '') {
        matches.value = [];
      } else {
        matches.value = findMatches(searchQuery.value);
      }
    }, 300);

    return {
      searchQuery,
      matches,
      numberWithCommas,
      highlightMatch,
      displayMatches
    };
  }
};
</script>

<style lang="scss" scoped>
input {
  width: 100%;
  padding: 20px;
}

.search-form {
  max-width: 400px;
  margin: auto;
  margin-top: 50px;
}

input.search {
  box-sizing: border-box;
  margin: 0;
  text-align: center;
  outline: 0;
  border: 10px solid #F7F7F7;
  position: relative;
  top: 10px;
  z-index: 2;
  border-radius: 5px;
  font-size: 40px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.14);
}

.suggestions {
  margin: 0;
  padding: 0;
  position: relative;
  li {
    background: white;
    list-style: none;
    border-bottom: 1px solid #D8D8D8;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.14);
    margin: 0;
    padding: 20px;
    transition: background 0.2s;
    display: flex;
    justify-content: space-between;
    text-transform: capitalize;
    right: -5px;
    position: relative;
    width: calc(100% - 10px);
    box-sizing: border-box;

    &:nth-child(even) {
      transform: perspective(100px) rotateX(3deg) translateY(2px) scale(1.001);
      background: linear-gradient(to bottom, #ffffff 0%, #EFEFEF 100%);
    }

    &:nth-child(odd) {
      transform: perspective(100px) rotateX(-3deg) translateY(3px);
      background: linear-gradient(to top, #ffffff 0%, #EFEFEF 100%);
    }
  }
}

span.population {
  font-size: 15px;
}
</style>
