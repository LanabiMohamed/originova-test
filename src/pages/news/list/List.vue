<template>
  <v-app class="border w-100 pt-4 overflow-x-hidden">
    <Details
      :dialogCard="dialog"
      :cardData="selectedCard"
      @CloseDialog="this.dialog = false"
    />
    <v-snackbar v-model="snackbar">
      <span>{{ snackbarText }}</span>

      <template v-slot:actions>
        <v-btn color="pink" variant="text" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
    <v-main>
      <v-row>
        <v-text-field v-model="q" width="500" label="Search"></v-text-field>
        <v-select
          width="1"
          label="Language"
          :items="['en', 'ar']"
          v-model="language"
        ></v-select>
        <v-select
          :disabled="loadingSources"
          v-model="source"
          :items="sources"
          item-title="title"
          item-value="id"
          label="Source"
        ></v-select>
        <v-date-input
          v-model="from"
          prepend-icon=""
          label="From Date"
          max-width="400"
          @update:modelValue="fetchNews"
        ></v-date-input>
      </v-row>
      <v-progress-linear
        v-if="loading"
        indeterminate
        color="blue"
        height="5"
      ></v-progress-linear>
      <v-row>
        <v-col v-for="news in newsList" :key="news.id" sm="6" md="4" lg="3">
          <v-card
            @click="
              () => {
                selectedCard = news;
                dialog = true;
              }
            "
          >
            <v-img height="200px" :src="news.urlToImage" cover>
              <template v-slot:placeholder>
                <div class="d-flex align-center justify-center fill-height">
                  <v-progress-circular
                    color="grey-lighten-4"
                    indeterminate
                  ></v-progress-circular>
                </div>
              </template>
              <template v-slot:error>
                <v-img
                  class="mx-auto"
                  height="300"
                  max-width="500"
                  src="https://picsum.photos/500/300?image=232"
                ></v-img>
              </template>
            </v-img>
            <v-card-title>{{ news.title }}</v-card-title>
            <v-card-subtitle>{{ news.description }}</v-card-subtitle>
            <v-card-actions>
              <v-btn
                :icon="
                  show === news.title ? 'mdi-chevron-up' : 'mdi-chevron-down'
                "
                @click="toggleContent(news.title, $event)"
              ></v-btn>
            </v-card-actions>
            <v-expand-transition>
              <div v-show="show === news.title">
                <v-divider></v-divider>
                <v-card-text>{{ news.content }}</v-card-text>
              </div>
            </v-expand-transition>
          </v-card>
        </v-col>
      </v-row>
      <v-card-text v-if="q && newsList && newsList.length === 0 && !loading"
        >No results found</v-card-text
      >
      <v-card-text v-if="!q">Please Search for Something.</v-card-text>
      <v-pagination
        v-if="pagesNumber > 1 && !loading"
        v-model="page"
        :length="pagesNumber"
        :total-visible="5"
      ></v-pagination>
    </v-main>
  </v-app>
</template>

<script>
import Details from "./dialogs/Details.vue";
import { VDateInput } from "vuetify/labs/VDateInput";
import debounce from "lodash/debounce";

export default {
  components: {
    VDateInput,
    Details,
  },
  data() {
    return {
      snackbarText: "",
      snackbar: false,
      loadingSources: false,
      selectedCard: null,
      dialog: false,
      sources: [],
      source: null,
      from: null,
      pagesNumber: 0,
      page: 1,
      pageSize: 6,
      ApiUrl:
        "https://newsapi.org/v2/everything?apiKey=dd43e5d652464a98b911bd22631f609c",
      q: "",
      newsList: [],
      loading: true,
      show: null,
      language: "en",
    };
  },
  created() {
    this.fetchNews();
    this.getSources();
  },
  methods: {
    fetchNews() {
      if (this.q === "") {
        this.newsList = [];
        this.pagesNumber = 0;
        this.loading = false;
        return;
      }

      const fromDate = this.from ? this.from.toISOString().split("T")[0] : "";
      this.loading = true;
      fetch(
        `${this.ApiUrl}&page=${this.page}&pageSize=${this.pageSize}&q=${this.q}&language=${this.language}&from=${fromDate}&source=${this.source}`
      )
        .then((response) => response.json())
        .then((json) => {
          this.pagesNumber = Math.ceil(json.totalResults / this.pageSize);
          this.newsList = json.articles;
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
          this.snackbarText = "Error fetching news";
          this.snackbar = true;
        });
    },
    getSources() {
      const Url =
        "https://newsapi.org/v2/top-headlines/sources?apiKey=dd43e5d652464a98b911bd22631f609c";
      fetch(Url)
        .then((response) => response.json())
        .then((json) => {
          this.sources = json.sources.map((item) => ({
            title: item.name,
            id: item.id,
          }));
          this.loadingSources = false;
        })
        .catch((error) => {
          this.loadingSources = false;
          this.snackbarText = "Error fetching sources";
          this.snackbar = true;
        });
    },
    toggleContent(title, event) {
      event.stopPropagation();
      this.show = this.show === title ? null : title;
    },
    debouncedFetchNews: debounce(function () {
      this.fetchNews();
    }, 600),
  },
  watch: {
    page() {
      this.fetchNews();
    },
    language() {
      this.page = 1;
      this.fetchNews();
    },
    source() {
      this.page = 1;
      this.fetchNews();
    },
    q() {
      this.page = 1;
      this.debouncedFetchNews();
    },
  },
};
</script>
