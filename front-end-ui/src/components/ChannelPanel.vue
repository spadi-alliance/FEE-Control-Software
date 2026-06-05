<template>
  <div class="channel-panel">
    <div class="panel-header">
      <h4>Channel {{ channel.id }}</h4>
    </div>

    <div class="panel-body">
      <label v-for="key in parameterKeys" :key="key" class="config-row">
        <span>{{ getParameterDef(key)?.description || key }}</span>
        
        <!-- Boolean select (Off/On) -->
        <select v-if="getParameterDef(key)?.type === 'boolean'" v-model="channel[key]">
          <option :value="false">Off</option>
          <option :value="true">On</option>
        </select>
        
        <!-- String or Number with options (dropdown) -->
        <select v-else-if="getParameterDef(key)?.options" v-model="channel[key]">
          <option v-for="(option, index) in getParameterDef(key).options" :key="option" :value="option">
            {{ index + 1 }} ({{ option }}{{ getParameterDef(key)?.unit ? ' ' + getParameterDef(key).unit : '' }})
          </option>
        </select>
        
        <!-- Number input (range) -->
        <input 
          v-else-if="getParameterDef(key)?.type === 'number'"
          type="number" 
          v-model.number="channel[key]"
          :min="getParameterDef(key)?.min"
          :max="getParameterDef(key)?.max"
          :step="getParameterDef(key)?.step"
        />
      </label>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  channel: {
    type: Object,
    required: true
  },
  getParameterOptions: {
    type: Function,
    required: true
  },
  getParameterDef: {
    type: Function,
    required: true
  }
})

// Get all parameter keys (excluding id and enabled)
const parameterKeys = computed(() => {
  return Object.keys(props.channel).filter(key => key !== 'id' && key !== 'enabled')
})
</script>

<style scoped>
/* Channel panel styles - copied from the original */
.channel-panel {
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: var(--shadow);
  background: var(--panel);
  margin-bottom: 16px;
  margin-left: 10px;
  padding: 8px 8px;
  float: left;
}

.panel-header {
  padding: 12px 16px;
  border-bottom: 1px solid var(--border);
  background: linear-gradient(90deg, var(--accent), color-mix(in srgb, var(--accent) 70%, black));
  color: white;
  border-radius: 12px 12px 0 0;
}

.panel-header h4 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
}

.panel-body {
  padding: 16px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.config-row {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.config-row span {
  font-weight: 500;
  font-size: 13px;
  color: var(--text);
}

.config-row select,
.config-row input {
  padding: 8px 12px;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: var(--panel);
  color: var(--text);
  font-size: 14px;
  font-weight: 500;
  min-width: 140px;
}

.config-row select {
  cursor: pointer;
}

.config-row input:focus,
.config-row select:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.1);
}

.full {
  width: 100%;
  border: none;
  border-top: 1px solid var(--border);
  margin: 12px 0;
}
</style>
