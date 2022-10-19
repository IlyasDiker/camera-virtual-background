<script setup>
import HeroHeader from './components/HeroHeader.vue';
import VideoPreview from './components/VideoPreview.vue';
import BifoliaLogo from './components/BifoliaLogo.vue';

</script>

<template>
  <HeroHeader/>
  <VideoPreview/>
  <article data-theme="dark" style="margin: 0; border-radius: 0;">
    <nav class="container">
      <a href="https://bifolia.ma/"><BifoliaLogo height="20px"/></a>
    </nav>
  </article>
</template>

<style scoped>

</style>
