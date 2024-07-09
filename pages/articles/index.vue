<template>
  <main class="min-h-screen">
    <AppHeader class="mb-16" title="Articles" :description="description" />
    <ul class="space-y-16">
      <li v-for="(article, id) in articles" :key="id">
        <AppArticleCard :article="article" />
      </li>
    </ul>
  </main>
</template>

<script setup>
const description = "";
useSeoMeta({
  title: "Articles | Yoshiki Masubuchi",
  description,
});

const { data: articles } = await useAsyncData("all-articles", () =>
  queryContent("/articles")
    .where({ published: { $exists: true } })
    .sort({ published: -1 })
    .find()
);
</script>
