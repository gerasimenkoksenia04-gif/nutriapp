<template>
  <div class="bg-white border border-gray-300 rounded-xl overflow-hidden mb-6">
    <div class="bg-gradient-to-r from-green-600 to-green-700 px-6 py-4">
      <div class="flex items-center justify-between">
        <div class="flex items-center space-x-3">
          <span class="text-white font-semibold text-lg">Вопрос {{ questionNumber }}</span>
        </div>
        <div class="bg-red-900/20 px-3 py-1 rounded-full">
          <span class="text-white text-xs font-medium">Обязательный вопрос</span>
        </div>
      </div>
    </div>

    <div class="p-4">
      <p class="font-medium text-gray-900 mb-3">{{ questionText }}</p>
      <div class="space-y-2">
        <label v-for="(option, index) in options" :key="index"
          class="flex items-center p-2 hover:bg-gray-50 rounded-lg cursor-pointer transition-colors">
          <input type="radio" :value="option.value" v-model="selectedValue" :name="`q${questionNumber}`"
            class="w-4 h-4 text-green-600 focus:ring-green-500">
          <span class="ml-3 text-sm text-gray-700">{{ option.label }}</span>
        </label>
      </div>
    </div>

    <div v-if="selectedValue" class="bg-green-50 px-6 py-3 border-t border-green-100">
      <div class="flex items-center text-sm">
        <svg class="w-4 h-4 text-green-600 mr-2" fill="currentColor" viewBox="0 0 20 20">
          <path fill-rule="evenodd"
            d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
            clip-rule="evenodd" />
        </svg>
        <span class="text-green-800">Ответ выбран</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'QuestionCard',
  props: {
    questionNumber: {
      type: Number,
      required: true
    },
    questionText: {
      type: String,
      required: true
    },
    options: {
      type: Array,
      required: true,
      validator: (value) => {
        return value.every(opt => opt.hasOwnProperty('value') && opt.hasOwnProperty('label'))
      }
    },
    modelValue: {
      type: [String, Number],
      default: null
    }
  },
  computed: {
    selectedValue: {
      get() {
        return this.modelValue
      },
      set(value) {
        this.$emit('update:modelValue', value)
      }
    }
  }
}
</script>