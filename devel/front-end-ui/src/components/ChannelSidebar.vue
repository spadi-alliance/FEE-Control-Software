<template>
  <div class="sidebar">
    <div class="sidebar-top">
      <h3>Channels</h3>
      <span class="badge">{{ selectedCount }} selected</span>
    </div>

    <div class="channels-grid">
      <label v-for="ch in channels" :key="ch.id" class="ch-checkbox">
        <div class="switch">
          <input type="checkbox" v-model="ch.enabled" :id="'ch'+ch.id" />
          <span class="slider"></span>
        </div>
        <span class="ch-label">Ch. {{ ch.id }}</span>
      </label>
    </div>

    <div class="controls">
      <button class="btn" @click="$emit('enableAll')">Enable All</button>
      <button class="btn" @click="$emit('disableAll')">Disable All</button>
      <button class="btn btn-primary" @click="$emit('applyConfig')">Apply Selected</button>
      <p class="status">Status: {{ status }}</p>
    </div>
  </div>
</template>

<script setup>
defineProps({
  channels: Array,
  selectedCount: Number,
  status: String
})

defineEmits(['enableAll', 'disableAll', 'applyConfig'])
</script>

<style scoped>
.sidebar {
  width: 100%;
  border-right: none;
  border-bottom: 1px solid var(--border);
  background: var(--panel);
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  flex-shrink: 0;
  margin-top: 10px;
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
  border-radius: 12px;
}

.sidebar h3 {
  font-weight: 600;
  font-size: 18px;
  margin-left: 5px;
  /* margin: 0 0 12px 0; */
  color: var(--accent);
}

.sidebar-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.channels-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 10px;
  margin-bottom: 12px;
}

.ch-checkbox {
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--panel);
  padding: 6px 8px;
  border-radius: 8px;
  border: 1px solid var(--border);
}

.ch-label {
  font-size: 14px;
  font-weight: 500;
}

.controls {
  display: flex;
  /* flex-direction: column; */
  gap: 10px;
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

.btn-primary {
  background: linear-gradient(90deg, var(--accent), color-mix(in srgb, var(--accent) 70%, black));
  color: white;
  border: none;
}

.badge {
  background: rgba(37, 99, 235, 0.12);
  color: var(--accent);
  padding: 4px 8px;
  font-size: 12px;
  border-radius: 999px;
}

.status {
  font-size: 13px;
  color: var(--muted);
  margin: 0;
}

/* Custom switch */
.switch {
  position: relative;
  width: 44px;
  height: 24px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
  position: absolute;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(180deg, rgba(0,0,0,0.03), rgba(0,0,0,0.01));
  border-radius: 999px;
  border: 1px solid var(--border);
  transition: .18s;
}

.slider::before {
  content: "";
  position: absolute;
  height: 18px;
  width: 18px;
  left: 2px;
  bottom: 2px;
  background: rgb(185, 185, 185);
  border: 1px solid var(--border);
  border-radius: 50%;
  transition: .18s;
}

.switch input:checked + .slider {
  background: var(--accent);
}

.switch input:checked + .slider::before {
  transform: translateX(20px);
}
</style>