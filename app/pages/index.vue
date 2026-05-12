<template>
    <div class="p-6 space-y-6">
        <div v-if="!res">
            <div
                class="bg-gradient-to-r from-green-50 to-green-100 border border-green-100 rounded-2xl p-6 sm:p-8 flex flex-col sm:flex-row items-center justify-between gap-5 shadow-sm">
                <div>
                    <p class="text-xl font-semibold text-gray-900">
                        Чтобы начать пользоваться приложением, пройдите короткий опрос
                    </p>

                    <p class="text-sm text-gray-500 mt-1">
                        Это займет меньше минуты
                    </p>
                </div>

                <NuxtLink to="/bio-info"
                    class="px-6 py-3 bg-orange-500 hover:bg-orange-600 text-white text-sm font-semibold rounded-xl transition-all hover:scale-105 shadow-md whitespace-nowrap">
                    Начать →
                </NuxtLink>
            </div>
            <img src="/donut-cat.png" alt="Cute cat" class="mx-auto w-128 h-auto" />
        </div>

        <div v-else class="space-y-6">
            <div>
                <h1 class="text-2xl font-bold text-gray-900">
                    Добро пожаловать, {{ res.Name }} 👋
                </h1>

                <p class="text-gray-500">
                    Сегодня отличный день держать курс
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">

                <StatCard title="Калории сегодня" :value="`${caloriesToday} / ${res.TDEE}`" suffix="ккал"
                    :description="`Осталось ${caloriesLeft} ккал`" :progress="caloriesProgress"
                    progressColor="bg-green-500" />

                <div class="bg-white border border-gray-300 rounded-lg p-5">

                    <div class="flex items-center justify-between mb-4">
                        <p class="text-sm text-gray-500 font-medium">
                            Ваш профиль
                        </p>

                        <NuxtLink to="/bio-info" class="text-xs text-blue-500 hover:text-blue-600">
                            Изменить (перепройти тест)
                        </NuxtLink>
                    </div>

                    <div class="grid grid-cols-2 gap-y-3 text-sm">

                        <div>
                            <p class="text-gray-400">Вес</p>
                            <p class="font-semibold">{{ res.Weight }} кг</p>
                        </div>

                        <div>
                            <p class="text-gray-400">Рост</p>
                            <p class="font-semibold">{{ res.Height }} см</p>
                        </div>

                        <div>
                            <p class="text-gray-400">Возраст</p>
                            <p class="font-semibold">{{ res.Age }}</p>
                        </div>

                        <div>
                            <p class="text-gray-400">BMI</p>
                            <p class="font-semibold">{{ res.BMI }}</p>
                        </div>

                        <div>
                            <p class="text-gray-400">BMR</p>
                            <p class="font-semibold">{{ res.BMR }}</p>
                        </div>

                        <div>
                            <p class="text-gray-400">TDEE</p>
                            <p class="font-semibold">{{ res.TDEE }}</p>
                        </div>
                    </div>

                </div>


            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">

                <NuxtLink to="/meals-diary" class="bg-white border rounded-xl p-5 hover:shadow-md transition">
                    <p class="font-semibold text-gray-900">Дневник питания</p>
                    <p class="text-sm text-gray-500 mt-1">
                        Все приемы пищи и статистика
                    </p>
                </NuxtLink>

                <NuxtLink to="/deficit-quiz" class="bg-white border rounded-xl p-5 hover:shadow-md transition">
                    <p class="font-semibold text-gray-900">Тест витаминов</p>
                    <p class="text-sm text-gray-500 mt-1">
                        Проверить возможные дефициты
                    </p>
                </NuxtLink>
            </div>

            <div class="space-y-3">

                <div class="flex items-center justify-between">
                    <h2 class="text-lg font-semibold text-gray-900">
                        Последние приемы пищи
                    </h2>

                    <button
                        class="px-4 py-2 bg-green-500 hover:bg-green-600 text-white rounded-xl text-sm font-medium transition"
                        @click="toDiary">
                        Открыть
                    </button>
                </div>

                <RecentMeals />

            </div>

        </div>

    </div>
</template>

<script setup>
import { computed } from "vue"
import { useLocalStorage } from "@vueuse/core"

const raw = useLocalStorage("biodata", null)
const meals = useLocalStorage("foodDiary", {})

const res = computed(() => {
    try {
        return JSON.parse(raw.value)
    } catch {
        return null
    }
})

function toDiary() {
    navigateTo("/meals-diary");
}


const today = new Date().toISOString().split("T")[0];

const todayMeals = computed(() => {
    console.log(meals);
    console.log(today);
    console.log(meals.value?.[today] || []);
    return meals.value?.[today] || []
})

const caloriesToday = computed(() => {
    return todayMeals.value.reduce((sum, item) => {
        return sum + Number(item.calories || 0)
    }, 0)
})

const caloriesLeft = computed(() => {
    if (!res.value?.TDEE) return 0
    return Math.max(res.value.TDEE - caloriesToday.value, 0)
})

const caloriesProgress = computed(() => {
    if (!res.value?.TDEE) return 0
    return Math.min(
        Math.round((caloriesToday.value / res.value.TDEE) * 100),
        100
    )
})
</script>