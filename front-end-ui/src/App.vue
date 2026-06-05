<template>
  <div class="container" id="app" :data-theme="theme">
    <BoardConfigPanel
      :boardConfigs="boardConfigs"
      :selectedBoard="selectedBoard"
      :channelCount="channelCount"
      :theme="theme"
      @toggleTheme="toggleTheme"
      @boardChanged="loadBoardConfig"
    />

    <CommonConfigPanel
      v-if="selectedBoard"
      :commonConfig="commonConfig"
      :getParameterOptions="getParameterOptions"
      :getParameterDef="getParameterDef"
      @applyCommon="applyCommonConfig"
    />

    <ChannelSidebar
      v-if="selectedBoard"
      :channels="channels"
      :selectedCount="selectedCount"
      :status="status"
      @enableAll="enableAll"
      @disableAll="disableAll"
      @applyConfig="applyConfig"
    />

    <ChannelPanels
      v-if="selectedBoard"
      :selected="selected"
      :getParameterOptions="getParameterOptions"
      :getParameterDef="getParameterDef"
    />
  </div>
</template>

<script setup>
import { reactive, computed, ref, watch, onMounted } from 'vue'
import BoardConfigPanel from './components/BoardConfigPanel.vue'
import CommonConfigPanel from './components/CommonConfigPanel.vue'
import ChannelSidebar from './components/ChannelSidebar.vue'
import ChannelPanels from './components/ChannelPanels.vue'

const status = ref('idle')
const files = import.meta.glob('/public/FEE-Config/*')
const knownFiles = Object.keys(files).map((path) => path.split('/').pop())

// Board configuration state
const boardConfigs = ref([])
const selectedBoard = ref('')
const currentBoardConfig = ref(null)
const asicConfig = ref(null)

// channel count and dynamic channels array
const channelCount = ref(16)
const channels = reactive([])

function createChannel(i, asicParams = []) {
  const channel = {
    id: i + 1,
    enabled: false
  }
  // Dynamically add properties from ASIC parameters
  asicParams.forEach(param => {
    channel[param.name] = param.defaultValue
  })
  return channel
}

function initChannels(n, asicParams = []) {
  const next = Array.from({ length: n }, (_, i) => createChannel(i, asicParams))
  channels.splice(0, channels.length, ...next)
}

const selected = computed(() => channels.filter(c => c.enabled))
const selectedCount = computed(() => selected.value.length)

// common configuration for all channels
const commonConfig = reactive({})

// theme state
const theme = ref('light')
function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

function enableAll() {
  channels.forEach(c => (c.enabled = true))
}
function disableAll() {
  channels.forEach(c => (c.enabled = false))
}

function applyCommonConfig() {
  channels.forEach(ch => {
    Object.keys(commonConfig).forEach(key => {
      ch[key] = commonConfig[key]
    })
  })
  status.value = `Applied common config to all ${channels.length} channels`
}

// Load board configurations on mount
async function loadBoardConfigs() {
  try {
    console.log('Loading board configs...')
    const configs = []
    
    for (const file of knownFiles) {
      try {
        console.log(`Trying to load ${file}...`)
        const res = await fetch(`FEE-Config/${file}`)
        if (res.ok) {
          const config = await res.json()
          console.log(`Loaded config for ${file}:`, config)
          configs.push({ file, name: config.Name, ...config })
        } else {
          console.warn(`Failed to load ${file}: ${res.status}`)
        }
      } catch (e) {
        console.warn(`Failed to load ${file}:`, e)
      }
    }
    console.log('Final configs:', configs)
    boardConfigs.value = configs
  } catch (e) {
    console.error('Failed to load board configurations:', e)
    status.value = 'Error loading board configs'
  }
}

// Load ASIC configuration
async function loadAsicConfig(asicName) {
  try {
    const response = await fetch(`FEE-Config/ASIC-Config/${asicName}.json`)
    if (!response.ok) throw new Error(`Failed to load ASIC config: ${asicName}`)
    
    const config = await response.json()
    asicConfig.value = config
    return config
  } catch (e) {
    console.error('Failed to load ASIC configuration:', e)
    status.value = `Error loading ASIC config: ${asicName}`
    throw e
  }
}

// Load board configuration and update UI
async function loadBoardConfig(boardFile) {
  selectedBoard.value = boardFile
  if (!selectedBoard.value) {
    currentBoardConfig.value = null
    asicConfig.value = null
    channelCount.value = 16
    initChannels(16)
    return
  }
  
  try {
    const boardConfig = boardConfigs.value.find(b => b.file === selectedBoard.value)
    if (!boardConfig) throw new Error('Board config not found')
    
    currentBoardConfig.value = boardConfig
    
    // Load ASIC config
    const asicData = await loadAsicConfig(boardConfig.ASIC)
    
    // Update channel count
    channelCount.value = boardConfig.input_channels
    
    // Initialize channels with new count and ASIC parameters
    const asicParams = asicData.channelParameters || []
    initChannels(channelCount.value, asicParams)
    
    // Before updating common config with parameter defaults, clear old properties first
    Object.keys(commonConfig).forEach(key => delete commonConfig[key])
    // Add new properties from ASIC config
    asicParams.forEach(param => {
      commonConfig[param.name] = param.defaultValue
    })
    
    status.value = `Loaded ${boardConfig.Name} with ${channelCount.value} channels`
  } catch (e) {
    console.error('Failed to load board config:', e)
    status.value = 'Error loading board configuration'
  }
}

// Get full parameter definition from ASIC config
function getParameterDef(paramName) {
  if (!asicConfig.value?.channelParameters) return null
  return asicConfig.value.channelParameters.find(p => p.name === paramName) || null
}

// Get parameter options from ASIC config
function getParameterOptions(paramName) {
  if (!asicConfig.value?.channelParameters) return []
  
  const param = asicConfig.value.channelParameters.find(p => p.name === paramName)
  if (!param?.options) return []
  
  return param.options.map(value => ({ value }))
}

async function applyConfig() {
  status.value = 'Sending...'
  try {
    const payload = selected.value.map(c => {
      const obj = { id: c.id }
      // Dynamically include all properties except id and enabled
      Object.keys(c).forEach(key => {
        if (key !== 'id' && key !== 'enabled') {
          obj[key] = c[key]
        }
      })
      return obj
    })

    const res = await fetch('/api/channels/config', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ channels: payload })
    })
    if (!res.ok) {
      const text = await res.text()
      throw new Error(text || res.statusText)
    }

    const data = await res.json()
    status.value = data.message ?? `sent ${payload.length} channels`
  } catch (e) {
    status.value = 'error: ' + (e.message ?? e)
  }
}

// Initialize
onMounted(() => {
  console.log('Component mounted, loading board configs...')
  loadBoardConfigs()
})

// Watch for boardConfigs changes
watch(boardConfigs, (newConfigs) => {
  console.log('boardConfigs changed:', newConfigs)
})
</script>

<style>
body {
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  margin: 20px;
}

/* Theme variables */
.container {
  /* --bg: #ffffff; */
  --panel: #fafafa;
  --muted: #6b7280;
  --text: #0f172a;
  --accent: #2563eb;
  --border: #3668a0;
  --shadow: 0 6px 18px rgba(16,24,40,0.06);
  background: var(--bg);
  color: var(--text);
  /* display: flex; */
  flex-direction: column;
  /* gap: 16px;
  padding: 16px; */
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  position: static;
  /* width: 100%; */
  /* max-width: 1400px; */
  margin: 0 auto;
  box-sizing: border-box;
  overflow: visible;
  border-radius: 12px;
}

/* Dark theme overrides */
.container[data-theme="dark"] {
  /* --bg: #0b1220; */
  --panel: #071228;
  --muted: #9aa4b2;
  --text: #e6eef8;
  --accent: #ea8d29ff;
  --border: rgba(255, 255, 255, 0.231);
  --shadow: 0 8px 24px rgba(2,6,23,0.6);
}

</style>
