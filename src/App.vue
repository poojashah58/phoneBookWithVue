<template>
  <v-app>
    <v-main id="phoneBookMain">
      <PhoneBook/>
    </v-main>
  </v-app>
</template>

<script>
import PhoneBook from './components/PhoneBook';

export default {
  name: 'App',

  components: {
    PhoneBook
  },

  data: () => ({
    //
  }),
};
</script>
