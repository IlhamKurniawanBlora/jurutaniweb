<script setup lang="ts">
  import { heroImageUrl } from '@/utils/hero'

  definePageMeta({
    // layout: 'default',
    // name: 'index',
    // alias: 'index',
    title: 'Home',
    description: 'New Arrivals, Big Savings!',
    navOrder: 1,
    type: 'primary',
    icon: 'i-mdi-home',
    // ogImage: 'images/ogImage.png', // url or local images inside public folder, for eg, ~/public/images/ogImage.png
  })

  const { optimizeImage } = useOptimizeImage()
  const heroImageOptimized = {
    alt: `hero`,
    cover: true,
    ...optimizeImage(
      heroImageUrl,
      /* options */
      {
        // placeholder: false, // placeholder image before the actual image is fully loaded.
      },
      true /* return bgStyles */,
    ),
  }

  const heroImage = heroImageOptimized.src
  const bgStyles = heroImageOptimized.bgStyles
</script>
<template>
  <div>
    <HeroSection />
    <UContainer class="pt-8">
       <WeatherSection />
     </UContainer>
    <UContainer class="pt-8">
      <LaunchSale class="mt-8" />
      <HotDeals class="mt-8" />
    </UContainer>
    <UContainer class="pt-8">
      <FeatureSection class="mt-8" />
    </UContainer>
  </div>
</template>
<style scoped></style>
