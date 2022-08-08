<template>
  <div>
    <v-subheader>
      {{ title }}
    </v-subheader>
    <v-row v-for="(item, k) in items" :key="k">
      {{ item }}
    </v-row>
  </div>
</template>

<script>
export default {
  name: "PopulationDisplay",

  props: {
    items: {
      type: Array,
      default: ()=> [],
    },
    title: {
      type: String,
      default: "",
    },
  },

  data: () => ({})
}
</script>