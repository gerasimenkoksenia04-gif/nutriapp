<script setup>
import { computed } from "vue";
import { useLocalStorage } from "@vueuse/core";

const today = new Date().toISOString().split("T")[0];

const foodDiary = useLocalStorage("foodDiary", {});

const mealsToday = computed(() => foodDiary.value[today] || []);

const recentMeals = computed(() =>
    [...mealsToday.value].reverse().slice(0, 5)
);
</script>

<template>
    <div v-if="recentMeals.length" class="space-y-3">
        <div v-for="(item, index) in recentMeals" :key="index" class="bg-white rounded-xl p-4 border border-gray-200">
            <div class="flex justify-between items-start gap-4">
                <div class="flex-1 min-w-0">
                    <p class="text-base font-semibold text-gray-800 truncate">
                        {{ item.name }}
                    </p>

                    <div class="flex flex-wrap gap-2 mt-3 text-xs">

                        <span class="px-2 py-1 rounded-full border border-blue-200 text-blue-600">
                            Белки {{ item.protein }} г
                        </span>

                        <span class="px-2 py-1 rounded-full border border-yellow-200 text-yellow-700">
                            Жиры {{ item.fat }} г
                        </span>

                        <span class="px-2 py-1 rounded-full border border-green-200 text-green-700">
                            Углеводы {{ item.carbs }} г
                        </span>

                    </div>
                </div>

                <div class="text-right shrink-0">
                    <p class="text-lg font-bold text-gray-900">
                        {{ item.calories }} ккал
                    </p>
                </div>
            </div>
        </div>

    </div>

    <div v-else class="text-gray-400 text-sm">
        За сегодня записей нет
    </div>
</template>