<template>
  <div class="panels">
    <p v-if="!selected.length" class="muted">Select one or more channels to show settings</p>

    <ChannelPanel
      v-for="ch in selected"
      :key="ch.id"
      :channel="ch"
      :getParameterOptions="getParameterOptions"
      :getParameterDef="getParameterDef"
    />
  </div>
</template>

<script setup>
import ChannelPanel from './ChannelPanel.vue'

defineProps({
  selected: Array,
  getParameterOptions: Function,
  getParameterDef: Function
})
</script>

<style scoped>
.panels {
  margin-top: 16px;
}

.muted {
  color: var(--muted);
  font-style: italic;
}
</style>