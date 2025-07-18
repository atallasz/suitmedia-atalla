<script setup lang="ts">
import HeaderContent from '@/components/HeaderContent.vue'
import FooterContent from '@/components/FooterContent.vue'
import IdeasBanner from '@/components/IdeasBanner.vue';
import IdeasPostList from '@/components/IdeasPostList.vue';
</script>

<template>
  <HeaderContent/>
  <main>
    <slot></slot>
  </main>
  <FooterContent/>
  <IdeasBanner />
  <IdeasPostList />
</template>
