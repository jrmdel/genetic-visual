<template>
  <div>
    <v-subheader>
      {{ text }}
    </v-subheader>
    <v-slider
      v-model="value" thumb-label
      :min="min" :max="max" :step="steps"
    >
    </v-slider>
  </div>
</template>

<script>
export default {
  name: "CustomSlider",

  props: {
    text: {
      type: String,
      default: "",
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100,
    },
    steps: {
      type: Number,
      default: 1,
    }
  },

  watch: {
    value: {
      handler(v) {
        this.$emit("valueChange", v)
      }
    }
  },

  data: () => ({
    value: 0,
  }),
}
</script>