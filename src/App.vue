<template>
  <div>
    <h1>Configinator</h1>
    <div v-for="(group, groupName) in groupedConfig" :key="groupName">
      <h2>{{ groupName }}</h2>
      <div v-for="(item, key) in group" :key="key">
        <label style="padding-right: 1vw">{{ key }}</label>
        <div v-if="isOptType(item?.parameters?.Type)">
          <input
            :list="`options-list-${groupName}-${key}`"
            :value="item.value"
            @input="updateValue(key, groupName, $event.target.value)"
          />
          <datalist :id="`options-list-${groupName}-${key}`">
            <option v-for="(option, index) in getOptions(item)" :key="index" :value="option">
              {{ option }}
            </option>
          </datalist>
        </div>

        <input
          v-else-if="isBoolean(item.value)"
          type="checkbox"
          :checked="item.value?.toLowerCase() === 'true'"
          @change="updateBoolean(key, groupName, $event.target.checked)"
        />
        <input
          v-else-if="isNumber(item.value)"
          type="number"
          :value="item.value"
          :min="item?.parameters?.Min"
          :max="item?.parameters?.Max"
          :step="item?.parameters?.Step"
          @input="updateValue(key, groupName, $event.target.value)"
        />
        <input
          v-else
          type="text"
          :value="item.value"
          @input="updateValue(key, groupName, $event.target.value)"
        />
        <!-- Display parameters if they exist -->
        <!-- <div v-if="item.parameters">
          <div v-for="(paramValue, paramKey) in item.parameters" :key="paramKey">
            <label>{{ paramKey }}</label>
            <input type="text" :value="paramValue" readonly />
          </div>
        </div> -->
      </div>
    </div>
  </div>
</template>

<script>
const optionSets = {
  Height: ['480', '576', '720', '1080', '1440', '2160', '4320']
}

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
  computed: {
    getOptions() {
      return (item) => {
        if (item?.parameters?.Opts) {
          const optsArray = item.parameters.Opts.split('|')
          if (optsArray.length === 1 && optionSets[optsArray[0]]) {
            return optionSets[optsArray[0]]
          } else {
            return optsArray
          }
        }
        return []
      }
    }
  },
  methods: {
    parseConfig(text) {
      const lines = text.split('\n')
      let parameters = null
      lines.forEach((line) => {
        if (line.startsWith('# C[')) {
          parameters = this.parseParameters(line)
        } else if (line.includes('=')) {
          const [key, value] = line.split('=')
          const groupName = key.trim().split('_')[0]
          if (!this.groupedConfig[groupName]) {
            this.groupedConfig[groupName] = {}
          }
          // Check if the Opts parameter is a single value that matches an option set
          if (parameters?.Opts && parameters.Opts.includes('|')) {
            const optsArray = parameters.Opts.split('|')
            if (optsArray.length === 1 && optionSets[optsArray[0]]) {
              parameters.Opts = optionSets[optsArray[0]].join('|')
            }
          }
          // Store the value and parameters
          this.groupedConfig[groupName][key.trim()] = {
            value: value.trim().replace(/^"|"$/g, ''),
            parameters: parameters
          }
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
      const lowerCase = value?.toLowerCase()
      return lowerCase === 'true' || lowerCase === 'false'
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
