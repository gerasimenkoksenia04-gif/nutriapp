<template>
  <div class="bg-white border border-gray-300 rounded-lg p-5">
    <div class="flex items-start justify-between">
      <div>
        <p class="text-sm text-gray-500 font-medium">
          {{ title }}
        </p>

        <h3 class="text-3xl font-bold text-gray-900 mt-2">
          {{ value }}
          <span v-if="suffix" class="text-base font-medium text-gray-400 ml-1">
            {{ suffix }}
          </span>
        </h3>
      </div>

      <div v-if="icon" class="w-11 h-11 rounded-xl bg-gray-100 flex items-center justify-center text-xl">
        {{ icon }}
      </div>
    </div>

    <div v-if="description" class="mt-4 text-sm text-gray-500">
      {{ description }}
    </div>

    <div v-if="progress !== null" class="mt-4">
      <div class="w-full h-2 bg-gray-100 rounded-full overflow-hidden">
        <div class="h-full rounded-full transition-all duration-500" :class="progressColor"
          :style="{ width: progress + '%' }"></div>
      </div>

      <p class="text-xs text-gray-400 mt-1">
        {{ progress }}%
      </p>
    </div>
  </div>
</template>

<script setup>
defineProps({
  title: String,
  value: [String, Number],
  suffix: {
    type: String,
    default: "",
  },
  icon: {
    type: String,
    default: "",
  },
  description: {
    type: String,
    default: "",
  },
  progress: {
    type: Number,
    default: null,
  },
  progressColor: {
    type: String,
    default: "bg-green-500",
  },
});
</script>