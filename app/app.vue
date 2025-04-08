<script setup>
useHead({
  htmlAttrs: {
    lang: 'en',
  },

  link: [
    { rel: 'icon', href: '/favicon.png' },
    { rel: 'icon', type: 'image/svg+xml', href: '/favicon.svg' },
    { rel: 'stylesheet', href: '/reset.css' },
    { rel: 'stylesheet', href: 'https://rsms.me/inter/inter.css' },
  ],
})
</script>

<template>
  <NuxtPage />

  <ScrollToTop class="border shadow-md" />
</template>

<style lang="postcss">
body {
  @apply min-h-screen bg-white dark:bg-gray-900 text-gray-700 dark:text-gray-200 font-sans;
}
</style>
