<template>
  <v-dialog v-model="dialog" width="auto">
    <v-card width="700" :title="cardData.author">
      <v-card-title>{{ cardData.title }}</v-card-title>
      <v-img height="200px" :src="cardData.urlToImage" cover></v-img>
      <h3>Description:</h3>
      <v-card-text>{{ cardData.description }}</v-card-text>

      <v-card-text>{{ cardData.content }}</v-card-text>

      <v-divider></v-divider>
      <v-card-subtitle
        >Published At:
        {{
          new Date(cardData.publishedAt)
            .toISOString()
            .split("T")[0]
            .split("-")
            .join(" ")
        }}</v-card-subtitle
      >
      <v-card-subtitle>Source: {{ cardData.source.name }}</v-card-subtitle>

      <template v-slot:actions>
        <v-btn class="ms-auto" text="Close" @click="dialog = false"></v-btn>
        <a :href="cardData.url" target="_blank">
          <v-btn text>Read More</v-btn>
        </a>
      </template>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: {
    dialogCard: Boolean,
    cardData: Object,
  },
  data() {
    return {
      dialog: null,
    };
  },
  methods: {
    close() {
      this.dialog = false;
    },
  },
  watch: {
    dialogCard: {
      handler(newVal) {
        this.dialog = newVal;
      },
    },
    dialog: {
      handler(newVal) {
        if (!newVal) this.$emit("CloseDialog", newVal);
      },
    },
  },
};
</script>
