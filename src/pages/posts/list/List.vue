<template>
  <v-data-table :headers="headers" :items="serverItems" :loading="loading">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <Add @HanldeSave="(item) => serverItems.unshift(item)" />
        <Edit
          :dialogEdit="dialogEdit"
          :editedItem="editedItem"
          @CloseDialogEdit="this.dialogEdit = false"
          @HanldeUpdate="
            (updatedItem) => {
              const index = serverItems.findIndex(
                (item) => item.id === updatedItem.id
              );
              serverItems.splice(index, 1, updatedItem);
            }
          "
        />
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn
                color="blue-darken-1"
                variant="text"
                :disabled="loadingDelete"
                :loading="loadingDelete"
                @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <template v-slot:item.actions="{ item, index }">
      <v-icon class="me-2" size="small" @click="openEditDialog(item)">
        mdi-pencil
      </v-icon>
      <v-icon size="small" @click="deleteItem(item)">mdi-delete</v-icon>
    </template>
  </v-data-table>
</template>

<script>
import Add from "./dialogs/Add.vue";
import Edit from "./dialogs/Edit.vue";

export default {
  components: {
    Add,
    Edit,
  },
  data: () => ({
    dialogEdit: false,
    dialogDelete: false,
    loadingDelete: false,
    headers: [
      { title: "Title", align: "start", sortable: false, key: "title" },
      { title: "Body", align: "start", sortable: false, key: "body" },
      { title: "Actions", key: "actions", sortable: false },
    ],
    itemsPerPage: 5,
    serverItems: [],
    loading: true,
    totalItems: 0,
    editedItem: {
      id: 0,
      title: "",
      body: "",
    },
  }),

  methods: {
    openEditDialog(item) {
      this.editedItem = { ...item };
      this.dialogEdit = true;
    },
    deleteItem(item) {
      this.editedIndex = this.serverItems.indexOf(item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.loadingDelete = true;
      fetch(
        `https://jsonplaceholder.typicode.com/posts/${this.editedItem.id}`,
        {
          method: "DELETE",
        }
      )
        .then(() => {
          this.serverItems.splice(this.editedIndex, 1);
          this.closeDelete();
        })
        .finally(() => {
          this.loadingDelete = false;
        })
        .catch(() => {
          alert("An error occurred. Please try again later.");
        });
    },

    closeDelete() {
      this.dialogDelete = false;
    },

    loadItems() {
      this.loading = true;
      fetch("https://jsonplaceholder.typicode.com/posts")
        .then((response) => response.json())
        .then((json) => {
          this.serverItems = json;
          this.loading = false;
        });
    },
  },

  mounted() {
    this.loadItems();
  },
};
</script>
