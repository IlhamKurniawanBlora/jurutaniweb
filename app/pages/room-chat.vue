<script lang="ts" setup>
useSeoMeta({
  title: 'Room Chat - Juru Tani',
  description: 'Stay updated with the latest Room Chat from Juru Tani.',
  ogTitle: 'Room Chat - Juru Tani',
  ogDescription: 'Stay updated with the latest Room Chat from Juru Tani.',
})
definePageMeta({
  prerender: false,
  middleware: ['auth']
});
</script>

<template>
  <div>
    <NuxtPage />
  </div>
</template>
