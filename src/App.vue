<script setup>
import { ref, reactive } from 'vue';
import AsyncFlags from './components/AsyncFlags.vue';
import MyOtherComponent from './components/MyOtherComponent.vue';
import { Client, provideClient, fetchExchange } from '@urql/vue';
import { authExchange } from '@urql/exchange-auth';

const ttlToken = 10 * 1000;
// Dummy token:
const token = reactive({ expire: Date.now() + ttlToken });

// This authExchange should refresh the token with new expire-time before
// new requests.
const auth = authExchange(async (utils) => ({
  willAuthError: (operation) => {
    const result = token.expire < Date.now();
    console.log('willAuthError:', result);
    return result;
  },
  addAuthToOperation: (operation) => operation,
  didAuthError: (error) => false,
  async refreshAuth() {
    console.log('refreshAuth start');
    token.expire = await new Promise((resolve) =>
      setTimeout(() => resolve(Date.now() + ttlToken), 300)
    );
    console.log('refreshAuth end');
  },
}));

const client = new Client({
  url: 'https://countries.trevorblades.com/graphql',
  exchanges: [auth, fetchExchange],
});

provideClient(client);

const continents = {
  AF: 'Africa',
  AN: 'Antarctica',
  AS: 'Asia',
  EU: 'Europe',
  NA: 'North America',
  OC: 'Oceania',
  SA: 'South America',
};
const choosen = ref('AF');

const timeLeftSeconds = ref(ttlToken / 1000);

setInterval(() => {
  const timeLeft = token.expire - Date.now();
  timeLeftSeconds.value = timeLeft > 0 ? Math.ceil(timeLeft / 1000) : 'Expired';
}, 1000);
</script>

<template>
  <div>
    <h1>Demo of async component failing when token needs to be refreshed</h1>
    <div class="token">
      Token time to live: <strong>{{ timeLeftSeconds }}</strong>
    </div>
    <section>
      <p>
        Click on a continent to load an async component with flags. (Open
        console to see logging.) This does not work if token has expired.
      </p>
      <button
        v-for="(value, key) of continents"
        :key="key"
        @click="choosen = key"
      >
        {{ value }}
      </button>
    </section>
    <!-- :key force component to rerender, to demonstrate problem with async components. -->
    <Suspense :key="choosen">
      <template #default><AsyncFlags :code="choosen" /></template>
      <template #fallback>
        Loading flags for {{ continents[choosen] }}... Should take less than a
        second.</template
      >
    </Suspense>
    <hr />
    <p>
      Click a button bellow to load data in a none async component. This works,
      even if token has expired.
    </p>
    <MyOtherComponent />
  </div>
</template>

<style scoped>
li {
  list-style: none;
}
.token {
  border: 1px solid;
}
</style>
