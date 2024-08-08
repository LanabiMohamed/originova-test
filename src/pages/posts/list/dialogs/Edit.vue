<template>
  <v-snackbar v-model="snackbar">
    <span>{{ snackbarText }}</span>

    <template v-slot:actions>
      <v-btn color="pink" variant="text" @click="snackbar = false">
        Close
      </v-btn>
    </template>
  </v-snackbar>
  <v-dialog v-model="dialog" max-width="1200">
    <template v-slot:default="{ isActive }">
      <v-card>
        <v-card-title>
          <span class="text-h5">Edit item</span>
        </v-card-title>
        <v-form v-model="valid" @submit.prevent="handleEditItem">
          <v-row>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="newItem.title"
                label="Title"
                :rules="[rules.required]"
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
              <v-textarea
                v-model="newItem.body"
                label="Body"
                :rules="[rules.required]"
              ></v-textarea>
            </v-col>
          </v-row>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              class="ml-auto"
              text="Close"
              id="closeEditItem"
              @click="isActive.value = false"
            ></v-btn>

            <v-btn
              color="blue-darken-1"
              variant="text"
              :loading="loading"
              :disabled="loading || !valid || !isChanged"
              type="submit"
            >
              Edit
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card>
    </template>
  </v-dialog>
</template>

<script>
import { computed } from "vue";
import { isEqual } from "lodash";

export default {
  props: {
    dialogEdit: Boolean,
    editedItem: Object,
  },
  data() {
    return {
      snackbarText: "",
      snackbar: false,
      newItem: {
        title: "",
        body: "",
      },
      valid: false,
      loading: false,
      rules: {
        required: (value) => !!value || "Field is required",
      },
      dialog: null,
    };
  },
  methods: {
    close() {
      this.dialog = false;
    },
    handleEditItem() {
      this.loading = true;
      fetch(
        `https://jsonplaceholder.typicode.com/posts/${this.editedItem.id}`,
        {
          method: "PATCH",
          body: JSON.stringify(this.newItem),
          headers: {
            "Content-Type": "application/json",
          },
        }
      )
        .then((response) => {
          if (!response.ok) {
            throw new Error("Error updating item");
          }
          this.$emit("HanldeUpdate", this.newItem);
          this.loading = false;
          this.close();
        })
        .catch((error) => {
          this.snackbarText = "Error updating item";
          this.snackbar = true;
          this.loading = false;
        });
    },
  },
  watch: {
    editedItem: {
      handler(newVal) {
        this.newItem = { ...newVal };
      },
    },
    dialogEdit: {
      handler(newVal) {
        this.dialog = newVal;
      },
    },
    dialog: {
      handler(newVal) {
        if (!newVal) this.$emit("CloseDialogEdit", newVal);
      },
    },
  },
  computed: {
    isChanged() {
      return !isEqual(this.newItem, this.editedItem);
    },
  },
};
</script>
