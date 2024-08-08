<template>
  <v-dialog v-model="dialog" max-width="500px">
    <template v-slot:activator="{ props }">
      <v-btn class="mb-2" color="primary" dark v-bind="props"> New Item </v-btn>
    </template>
    <v-card>
      <v-card-title>
        <span class="text-h5">Add item</span>
      </v-card-title>
      <v-form v-model="valid">
        <v-container>
          <v-row>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="item.title"
                label="Title"
                :rules="rules"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="item.body"
                label="Body"
                :rules="rules"
                required
              ></v-text-field>
            </v-col>
          </v-row>
        </v-container>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue-darken-1" variant="text" @click="close">
            Cancel
          </v-btn>
          <v-btn
            color="blue-darken-1"
            variant="text"
            :disabled="loadingAdd || !valid"
            :loading="loadingAdd"
            @click="HandleAdd"
            type="submit"
          >
            Add
          </v-btn>
        </v-card-actions>
      </v-form>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data() {
    return {
      valid: false,
      dialog: false,
      loadingAdd: false,
      item: { title: "", body: "" },
    };
  },

  computed: {
    rules() {
      return [(v) => !!v || "Title is required"];
    },
  },

  methods: {
    HandleAdd() {
      this.loadingAdd = true;
      fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        body: JSON.stringify(this.item),
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((response) => response.json())
        .then((json) => {
          this.$emit("HanldeSave", { ...this.item, ...json });
          this.loadingAdd = false;
          this.close();
        });
    },

    close() {
      this.dialog = false;
    },
  },
};
</script>
