<template>
  <div>
    <h1>Configuration Editor</h1>
    <div v-for="(group, groupName) in groupedConfig" :key="groupName">
      <h2>{{ groupName }}</h2>
      <div v-for="(value, key) in group" :key="key">
        <label>{{ key }}</label>
        <!-- Use separate inputs for different types to avoid dynamic type binding issues -->
        <input
          v-if="isBoolean(value)"
          type="checkbox"
          :checked="value === 'true'"
          @change="updateBoolean(key, groupName, $event.target.checked)"
        />
        <input
          v-else-if="isNumber(value)"
          type="number"
          :value="value"
          @input="updateValue(key, groupName, $event.target.value)"
        />
        <input
          v-else
          type="text"
          :value="value"
          @input="updateValue(key, groupName, $event.target.value)"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      groupedConfig: {}
    }
  },
  async mounted() {
    const response = await fetch('/retroarch.cfg')
    const text = await response.text()
    this.parseConfig(text)
  },
  methods: {
    parseConfig(text) {
      const lines = text.split('\n')
      lines.forEach((line) => {
        if (line.includes('=')) {
          const [key, value] = line.split('=')
          const groupName = key.trim().split('_')[0]
          if (!this.groupedConfig[groupName]) {
            this.groupedConfig[groupName] = {}
          }
          // Directly store the trimmed value
          this.groupedConfig[groupName][key.trim()] = value.trim().replace(/^"|"$/g, '') // Remove quotes here for simplicity
        }
      })
    },
    isBoolean(value) {
      // Check without quotes
      return value === 'true' || value === 'false'
    },
    isNumber(value) {
      // Check if the value is a number
      return !isNaN(parseFloat(value)) && isFinite(value)
    },
    updateValue(key, groupName, newValue) {
      // Update the value directly
      this.groupedConfig[groupName][key] = newValue
    },
    updateBoolean(key, groupName, isChecked) {
      // Convert boolean to string representation
      this.groupedConfig[groupName][key] = isChecked ? 'true' : 'false'
    }
  }
}
</script>
