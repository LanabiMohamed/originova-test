<template>
  <v-dialog v-model="dialog" width="auto">
    <v-card width="700" :title="selectedCard.author">
      <v-card-title>{{ selectedCard.title }}</v-card-title>
      <v-img height="200px" :src="selectedCard.urlToImage" cover></v-img>
      <h3>Description:</h3>
      <v-card-text>{{ selectedCard.description }}</v-card-text>

      <v-card-text>{{ selectedCard.content }}</v-card-text>

      <v-divider></v-divider>
      <v-card-subtitle
        >Published At:
        {{
          new Date(selectedCard.publishedAt)
            .toISOString()
            .split("T")[0]
            .split("-")
            .join(" ")
        }}</v-card-subtitle
      >
      <v-card-subtitle>Source: {{ selectedCard.source.name }}</v-card-subtitle>

      <template v-slot:actions>
        <v-btn class="ms-auto" text="Close" @click="dialog = false"></v-btn>
        <a :href="selectedCard.url" target="_blank">
          <v-btn text>Read More</v-btn>
        </a>
      </template>
    </v-card>
  </v-dialog>
  <v-col>
    <h1>News</h1>
    <v-row>
      <v-text-field
        v-model="q"
        width="500"
        label="Search"
        class="mr-2"
      ></v-text-field>

      <v-select
        width="1"
        label="Language"
        :items="['en', 'ar']"
        v-model="language"
        class="mr-2"
      ></v-select>

      <v-select
        :disabled="loadingSources"
        v-model="source"
        :items="sources"
        item-title="title"
        item-value="id"
        label="From Date"
        class="mr-2"
      ></v-select>

      <v-date-input
        v-model="from"
        prepend-icon=""
        label="From Date"
        max-width="400"
        @update:modelValue="() => fetchNews()"
        class="mr-2"
      ></v-date-input>
    </v-row>

    <v-progress-linear
      v-if="loading"
      indeterminate
      color="blue"
      height="5"
    ></v-progress-linear>

    <v-row>
      <v-col v-for="news in newsList" :key="news.id">
        <v-card
          max-width="399"
          @click="
            () => {
              selectedCard = news;
              dialog = true;
            }
          "
        >
          <v-img height="200px" :src="news.urlToImage" cover></v-img>

          <v-card-title>{{ news.title }}</v-card-title>

          <v-card-subtitle>{{ news.description }}</v-card-subtitle>

          <v-card-actions>
            <v-btn
              :icon="
                show === news.title ? 'mdi-chevron-up' : 'mdi-chevron-down'
              "
              @click="
                (event) => {
                  event.stopPropagation();
                  show = show === news.title ? null : news.title;
                }
              "
            ></v-btn>
          </v-card-actions>

          <v-expand-transition>
            <div v-show="show === news.title">
              <v-divider></v-divider>

              <v-card-text>
                {{ news.content }}
              </v-card-text>
            </div>
          </v-expand-transition>
        </v-card>
      </v-col>
    </v-row>
    <v-card-text v-if="newsList && newsList.length === 0 && !loading"
      >no Resuts founded</v-card-text
    >
    <v-pagination
      v-if="pagesNumber > 1 && !loading"
      v-model="page"
      :length="pagesNumber"
      :total-visible="5"
    />
  </v-col>
</template>

<script>
import { VDateInput } from "vuetify/labs/VDateInput";
import debounce from "lodash/debounce";
export default {
  components: {
    VDateInput,
  },
  data() {
    return {
      loadingSources: false,
      selectedCard: null,
      dialog: false,
      sources: [],
      source: null,
      loadingSources: false,
      from: null,
      toggleFilter: false,
      pagesNumber: 0,
      page: 1,
      pageSize: 6,
      ApiUrl:
        "https://newsapi.org/v2/everything?apiKey=dd43e5d652464a98b911bd22631f609c",
      q: "bitcoin",
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
          this.pagesNumber = Math.floor(json.totalResults / this.pageSize);

          this.newsList = json.articles;
          this.loading = false;
        })
        .catch((error) => {
          console.error(error);
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
        });
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
