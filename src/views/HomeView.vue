<script setup lang="ts">
import TipTap from '@/components/TipTap.vue';
</script>

<template>
  <main class="container">
    <div class="header">
      <h1>Enhanced TipTap Editor</h1>
    </div>

    <div class="editor-section">
      <div class="editor-container">
        <TipTap />
      </div>
    </div>
  </main>
</template>

<style>
.container {
  max-width: 960px;
  margin: 0 auto;
  padding: 2rem;
}

.header {
  text-align: center;
  margin-bottom: 4rem;
}

.editor-section {
  display: flex;
  gap: 2rem;
}

.editor-container {
  flex: 1;
}
</style>
