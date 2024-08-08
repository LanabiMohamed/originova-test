<template>
  <v-app theme="dark">
    <main class="main--container">
      <aside class="dashboard">
        <v-list-item
          v-for="n in [
            { title: 'Originova', link: '/' },
            { title: 'Posts', link: '/posts' },
            { title: 'News', link: '/news' },
            { title: 'About', link: '/about' },
          ]"
          :key="n.title"
          :title="n.title"
          :href="n.link"
        ></v-list-item>
      </aside>
      <router-view />
    </main>
  </v-app>
</template>

<style scoped>
.main--container {
  display: flex;
  height: 100svh;
}
.dashboard {
  padding: 1rem;
  color: black;
  background-color: #b9babb;
  width: 15rem;
  height: 100%;
  border-radius: 0px 30px 30px 0px;
  box-shadow: 0px 0px 5px 0px #2f2f2f;
}
.dashboard > * {
  display: block;
  text-decoration: none;
}
</style>
