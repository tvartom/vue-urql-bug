<script setup>
import { gql, useQuery } from '@urql/vue';
import { ref } from 'vue';
// https://studio.apollographql.com/public/countries/variant/current/home

const code = ref(null);
const { data } = useQuery({
  query: gql`
  query ($code: ID!) {
  country(code: $code) {
    name
  }

}`,
  variables: { code },
});

const contryCodes = ['SE', 'AQ'];
</script>

<template>
  <div>
    <button
      v-for="countryCode of contryCodes"
      :key="countryCode"
      @click="code = countryCode"
    >
      {{ countryCode }}
    </button>
    <pre>{{ JSON.stringify(data) }}</pre>
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
