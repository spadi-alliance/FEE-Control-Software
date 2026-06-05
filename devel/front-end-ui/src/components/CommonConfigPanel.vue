<template>
  <div class="common-config-panel">
    <h4>Common Configuration</h4>
    <div class="config-grid">
      <label v-for="key in Object.keys(commonConfig)" :key="key" class="config-row">
        <span>{{ getParameterDef(key)?.description || key }}</span>
        
        <!-- Boolean select (Off/On) -->
        <select v-if="getParameterDef(key)?.type === 'boolean'" v-model="commonConfig[key]">
          <option :value="false">Off</option>
          <option :value="true">On</option>
        </select>
        
        <!-- String or Number with options (dropdown) -->
        <select v-else-if="getParameterDef(key)?.options" v-model="commonConfig[key]">
          <option v-for="(option, index) in getParameterDef(key).options" :key="option" :value="option">
            {{ index + 1 }} ({{ option }}{{ getParameterDef(key)?.unit ? ' ' + getParameterDef(key).unit : '' }})
          </option>
        </select>
        
        <!-- Number input (range) -->
        <input 
          v-else-if="getParameterDef(key)?.type === 'number'"
          type="number" 
          v-model.number="commonConfig[key]"
          :min="getParameterDef(key)?.min"
          :max="getParameterDef(key)?.max"
          :step="getParameterDef(key)?.step"
        />
      </label>
    </div>
    <button class="btn btn-primary" @click="$emit('applyCommon')">Apply to All Channels</button>
  </div>
</template>

<script setup>
const props = defineProps({
  commonConfig: Object,
  getParameterOptions: Function,
  getParameterDef: Function
})

defineEmits(['applyCommon'])
</script>

<style scoped>
.common-config-panel {
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: var(--shadow);
  background: var(--panel);
  padding: 16px;
  margin-bottom: 16px;
}

.common-config-panel h4 {
  margin: 0 0 16px 0;
  font-size: 18px;
  font-weight: 600;
  color: var(--accent);
}

.config-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
  margin-bottom: 16px;
}

.config-row {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.config-row span {
  font-weight: 500;
  font-size: 14px;
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
}

.config-row select {
  cursor: pointer;
}

.config-row select:focus,
.config-row input:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.1);
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: linear-gradient(90deg, var(--accent), color-mix(in srgb, var(--accent) 70%, black));
  color: white;
  border: none;
}

.btn-primary:hover {
  opacity: 0.9;
}
</style>
