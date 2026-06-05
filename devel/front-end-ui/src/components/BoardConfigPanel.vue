<template>
  <div class="board-config-panel">
    <div class="board-header">
      <h4>Board Configuration</h4>
      <div class="board-controls">
        <span v-if="selectedBoard" class="badge">{{ channelCount }} channels</span>
        <button class="btn btn-ghost" @click="$emit('toggleTheme')">
          {{ theme === 'light' ? '🌤 Light' : '🌙 Dark' }}
        </button>
      </div>
    </div>
    <div class="config-row">
      <span>Select Board ({{ boardConfigs.length }} loaded)</span>
      <select :value="selectedBoard" @change="$emit('boardChanged', $event.target.value)">
        <option value="">Select a board...</option>
        <option v-for="board in boardConfigs" :key="board.file" :value="board.file">
          {{ board.name }}
        </option>
      </select>
    </div>
  </div>
</template>

<script setup>
defineProps({
  boardConfigs: Array,
  selectedBoard: String,
  channelCount: Number,
  theme: String
})

defineEmits(['toggleTheme', 'boardChanged'])
</script>

<style scoped>
.board-config-panel {
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: var(--shadow);
  background: var(--panel);
  padding: 16px;
  margin-bottom: 16px;
}

.board-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.board-header h4 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: var(--accent);
}

.board-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.config-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.config-row span {
  font-weight: 500;
}

.config-row select {
  flex: 1;
  padding: 8px;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: var(--bg);
  color: var(--text);
}

.config-row select:focus {
  outline: none;
  border-color: var(--accent);
}

.btn {
  padding: 8px 10px;
  border-radius: 8px;
  background: transparent;
  border: 1px solid var(--border);
  color: var(--text);
  cursor: pointer;
  transition: all .12s ease;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow);
}

.btn-ghost {
  background: transparent;
  border: none;
  color: var(--muted);
  padding: 4px 6px;
}

.badge {
  background: rgba(37, 99, 235, 0.12);
  color: var(--accent);
  padding: 4px 8px;
  font-size: 12px;
  border-radius: 999px;
}
</style>