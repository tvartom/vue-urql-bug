<script setup>
import { gql, useQuery } from '@urql/vue';
// https://studio.apollographql.com/public/countries/variant/current/home
const props = defineProps(['code']);

const code = props.code;

console.log(`${code}: Start waiting for useQuery.`);

const { data } = await useQuery({
  query: gql`
  query ($code: ID!) {
  continent(code: $code) {
    name
    countries {
      code
      name
      emoji
    }
  }

}`,
  variables: { code },
});
console.log(`${code}: useQuery resolved!`);

// Delay to make reload visible on fast networks:
await new Promise((resolve) => setTimeout(() => resolve(), 500));
</script>

<template>
  <div>
    <h2>
      {{ data.continent.countries.length }} flags in {{ data.continent.name }}
    </h2>
    <ul>
      <li
        v-for="country of data.continent.countries"
        :key="country.code"
        :title="`${country.name} (${country.code})`"
      >
        {{ country.emoji }}
      </li>
    </ul>
  </div>
</template>

<style scoped>
ul {
  display: flex;
  gap: 10px 20px;
  width: 100%;
  flex-wrap: wrap;
}
li {
  list-style: none;
}
</style>
