<template>
  <div>
    <h1>Configuration Editor</h1>
    <div v-for="(group, groupName) in groupedConfig" :key="groupName">
      <h2>{{ groupName }}</h2>
      <div v-for="(item, key) in group" :key="key">
        <label style="padding-right: 1vw">{{ key }}</label>
        <!-- Use separate inputs for different types to avoid dynamic type binding issues -->
        <input
          v-if="isBoolean(item.value)"
          type="checkbox"
          :checked="item.value === 'true'"
          @change="updateBoolean(key, groupName, $event.target.checked)"
        />
        <input
          v-else-if="isNumber(item.value)"
          type="number"
          :value="item.value"
          @input="updateValue(key, groupName, $event.target.value)"
        />
        <select
          v-else-if="isOptType(item?.parameters?.Type)"
          :value="item.value"
          @input="updateValue(key, groupName, $event.target.value)"
        >
          <option
            v-for="(option, index) in item?.parameters?.Opts?.split('|')"
            :key="index"
            :value="option"
          >
            {{ option }}
          </option>
        </select>
        <input
          v-else
          type="text"
          :value="item.value"
          @input="updateValue(key, groupName, $event.target.value)"
        />
        <!-- Display parameters if they exist -->
        <div v-if="item.parameters">
          <div v-for="(paramValue, paramKey) in item.parameters" :key="paramKey">
            <label>{{ paramKey }}</label>
            <input type="text" :value="paramValue" readonly />
          </div>
        </div>
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
      let parameters = null
      lines.forEach((line) => {
        if (line.startsWith('# C[')) {
          // Parse parameters from the comment
          parameters = this.parseParameters(line)
        } else if (line.includes('=')) {
          const [key, value] = line.split('=')
          const groupName = key.trim().split('_')[0]
          if (!this.groupedConfig[groupName]) {
            this.groupedConfig[groupName] = {}
          }
          // Store the value and parameters
          this.groupedConfig[groupName][key.trim()] = {
            value: value.trim().replace(/^"|"$/g, ''),
            parameters: parameters
          }
          // Reset parameters for the next line
          parameters = null
        }
      })
    },
    parseParameters(comment) {
      // Extract parameters from the comment
      const match = comment.match(/# C\[(.*?)\]R/)
      if (match) {
        const params = match[1].split(',').reduce((acc, param) => {
          const [key, value] = param.split(':').map((s) => s.trim())
          acc[key] = value
          return acc
        }, {})
        return params
      }
      return null
    },
    isBoolean(value) {
      // Check without quotes
      return value === 'true' || value === 'false'
    },
    isNumber(value) {
      // Check if the value is a number
      return !isNaN(parseFloat(value)) && isFinite(value)
    },
    isOptType(type) {
      // Check if the type is 'Opt'
      return type === 'Opt'
    },
    updateValue(key, groupName, newValue) {
      // Update the value directly
      this.groupedConfig[groupName][key].value = newValue
    },
    updateBoolean(key, groupName, isChecked) {
      // Convert boolean to string representation
      this.groupedConfig[groupName][key].value = isChecked ? 'true' : 'false'
    }
  }
}
</script>
