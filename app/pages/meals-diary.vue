<template>
    <div class="mt-2 p-6 space-y-6">
        <div class="grid grid-cols-2 items-center">
            <h1 class="text-2xl font-bold text-gray-800 dark:text-white">
                Дневник питания
            </h1>

            <div class="flex justify-end">
                <button @click="goBack"
                    class="px-3 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-white transition">
                    Назад
                </button>
            </div>
        </div>

        <div class="flex items-center gap-3">
            <label class="text-sm text-gray-600 dark:text-gray-300">Дата:</label>
            <input type="date" v-model="selectedDate" :max="today"
                class="border rounded-lg px-3 py-2 dark:bg-gray-800 dark:border-gray-700 dark:text-white" />

            <span v-if="isToday" class="text-xs px-2 py-1 bg-green-100 text-green-700 rounded-full">
                Сегодня
            </span>

            <span v-else class="text-xs px-2 py-1 bg-gray-100 text-gray-600 rounded-full">
                Архив
            </span>
        </div>

        <div class="relative w-56 h-28 mx-auto overflow-hidden">
            <svg class="w-full h-full" viewBox="0 0 100 50">
                <defs>
                    <linearGradient id="fuelGradient" x1="0%" y1="0%" x2="100%" y2="0%">
                        <stop offset="0%" stop-color="#f97316" />
                        <stop offset="50%" stop-color="#f97316" />
                        <stop offset="100%" stop-color="#22c55e" />
                    </linearGradient>
                </defs>

                <path d="M 10 50 A 40 40 0 0 1 90 50" fill="none" class="stroke-gray-200 dark:stroke-gray-700"
                    stroke-width="8" stroke-linecap="round" />

                <path d="M 10 50 A 40 40 0 0 1 90 50" fill="none" stroke="url(#fuelGradient)" stroke-width="8"
                    stroke-linecap="round" class="transition-all duration-500"
                    :stroke-dasharray="`${progressValue}, 126`" />
            </svg>

            <div class="absolute bottom-0 left-0 right-0 flex flex-col items-center">
                <span class="text-2xl font-bold text-gray-800 dark:text-white">
                    {{ totalCalories }} / {{ calorieGoal }}
                </span>
                <span class="text-xs text-gray-500 uppercase">Калорий</span>
            </div>
        </div>

        <div
            class="grid grid-cols-2 md:grid-cols-4 gap-3 bg-white dark:bg-gray-800 rounded-2xl p-4 border border-gray-200 dark:border-gray-700">
            <div>
                <p class="text-xs text-gray-500">Калории</p>
                <p class="font-bold text-lg">{{ totalCalories }}</p>
            </div>

            <div>
                <p class="text-xs text-gray-500">Белки</p>
                <p class="font-bold text-lg">{{ totalProtein }} г</p>
            </div>

            <div>
                <p class="text-xs text-gray-500">Жиры</p>
                <p class="font-bold text-lg">{{ totalFat }} г</p>
            </div>

            <div>
                <p class="text-xs text-gray-500">Углеводы</p>
                <p class="font-bold text-lg">{{ totalCarbs }} г</p>
            </div>
        </div>

        <div class="space-y-3">
            <h2 class="font-semibold text-gray-800 dark:text-white">
                Продукты за день
            </h2>

            <div v-if="foods.length === 0" class="text-sm text-gray-400 bg-gray-50 dark:bg-gray-800 rounded-xl p-4">
                Пока ничего не добавлено
            </div>

            <div v-for="(item, index) in foods" :key="index"
                class="bg-white dark:bg-gray-800 rounded-2xl p-4 border border-gray-200 dark:border-gray-700">
                <div class="flex justify-between items-start gap-4">
                    <div class="flex-1 min-w-0">
                        <p class="text-base font-semibold text-gray-800 dark:text-white truncate">
                            {{ item.name }}
                        </p>

                        <div class="flex flex-wrap gap-2 mt-3 text-xs">

                            <span
                                class="px-2 py-1 rounded-full border border-blue-200 text-blue-600 dark:border-blue-700 dark:text-blue-300">
                                Белки {{ item.protein }} г
                            </span>

                            <span
                                class="px-2 py-1 rounded-full border border-yellow-200 text-yellow-700 dark:border-yellow-700 dark:text-yellow-300">
                                Жиры {{ item.fat }} г
                            </span>

                            <span
                                class="px-2 py-1 rounded-full border border-green-200 text-green-700 dark:border-green-700 dark:text-green-300">
                                Углеводы {{ item.carbs }} г
                            </span>

                        </div>
                    </div>

                    <div class="text-right shrink-0">
                        <p class="text-lg font-bold text-gray-900 dark:text-white">
                            {{ item.calories }} ккал
                        </p>

                        <button v-if="isToday" @click="removeFood(index)"
                            class="mt-3 text-xs px-3 py-1 rounded-lg border border-red-200 text-red-500 hover:bg-red-50 dark:border-red-700 dark:text-red-400">
                            Удалить
                        </button>
                    </div>

                </div>
            </div>
        </div>

        <button v-if="isToday" @click="showModal = true"
            class="w-full bg-orange-400 hover:bg-orange-500 text-white py-3 rounded-xl font-semibold">
            + Добавить продукт
        </button>

        <div v-if="showModal" class="fixed inset-0 bg-black/40 flex items-center justify-center z-50 px-4">
            <div class="bg-white dark:bg-gray-900 rounded-2xl p-6 w-full max-w-md space-y-5 shadow-xl">
                <h3 class="text-xl font-bold text-gray-800 dark:text-white">
                    Добавить продукт
                </h3>

                <div class="space-y-1">
                    <div class="space-y-1">
                        <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                            Поиск продукта
                        </label>

                        <input v-model="searchQuery" type="text" placeholder="Начни вводить..."
                            class="w-full px-4 py-2 border border-gray-300 rounded-lg dark:bg-gray-800 dark:border-gray-700 dark:text-white" />

                        <div v-if="searchResults.length"
                            class="border rounded-lg mt-2 max-h-40 overflow-auto bg-white dark:bg-gray-800">

                            <div v-for="item in searchResults" :key="item.name" @click="selectFood(item)"
                                class="px-3 py-2 hover:bg-gray-100 dark:hover:bg-gray-700 cursor-pointer">

                                {{ item.name }}
                            </div>
                        </div>
                    </div>

                    <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                        Название продукта
                    </label>

                    <input v-model="newFood.name" type="text" placeholder="Введите название продукта"
                        class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                </div>

                <div class="space-y-1">
                    <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                        Калории
                    </label>

                    <input v-model.number="newFood.calories" type="number" placeholder="0"
                        class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                </div>

                <div class="space-y-1">
                    <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                        Граммы
                    </label>

                    <input v-model.number="grams" type="number" placeholder="100"
                        class="w-full px-4 py-2 border border-gray-300 rounded-lg dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                </div>

                <div class="grid grid-cols-3 gap-3">
                    <div class="space-y-1">
                        <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                            Белки
                        </label>

                        <input v-model.number="newFood.protein" type="number" placeholder="0"
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                    </div>

                    <div class="space-y-1">
                        <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                            Жиры
                        </label>

                        <input v-model.number="newFood.fat" type="number" placeholder="0"
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                    </div>

                    <div class="space-y-1">
                        <label class="text-sm font-medium text-gray-700 dark:text-gray-300">
                            Углеводы
                        </label>

                        <input v-model.number="newFood.carbs" type="number" placeholder="0"
                            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none dark:bg-gray-800 dark:border-gray-700 dark:text-white" />
                    </div>
                </div>

                <div class="text-sm text-gray-600 dark:text-gray-300 space-y-1">
                    <p>Калории: {{ calculatedFood.calories }}</p>
                    <p>Белки: {{ calculatedFood.protein }} г</p>
                    <p>Жиры: {{ calculatedFood.fat }} г</p>
                    <p>Углеводы: {{ calculatedFood.carbs }} г</p>
                </div>

                <div class="flex gap-3 pt-2">
                    <button @click="showModal = false"
                        class="flex-1 py-2.5 border border-gray-300 rounded-xl hover:bg-gray-50 dark:border-gray-700 dark:text-white dark:hover:bg-gray-800">
                        Отмена
                    </button>

                    <button @click="addFood"
                        class="flex-1 py-2.5 bg-orange-400 hover:bg-orange-500 text-white rounded-xl font-medium">
                        Добавить
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>
<script setup>
import { ref, computed } from "vue";
import { useLocalStorage, useFetch } from "@vueuse/core";

const today = new Date().toISOString().split("T")[0];
const selectedDate = ref(today);

const foodDiary = useLocalStorage("foodDiary", {});
const raw = useLocalStorage("biodata", "{}");

const { data: questions } = useFetch('/foods.json').json();

const res = computed(() => {
    try {
        return JSON.parse(raw.value);
    } catch {
        return {};
    }
});

const calorieGoal = computed(() => res.value?.TDEE ?? 0);

const foods = computed(() => {
    return foodDiary.value[selectedDate.value] || [];
});

const showModal = ref(false);

const newFood = ref({
    name: "",
    calories: null,
    protein: null,
    fat: null,
    carbs: null,
});

const searchQuery = ref("");

const searchResults = computed(() => {
    if (!searchQuery.value.trim()) return [];

    const list = questions.value || [];

    return list
        .filter(item =>
            (item.name || item.title || "")
                .toLowerCase()
                .includes(searchQuery.value.toLowerCase())
        )
        .slice(0, 5);
});

const grams = ref(100);

function selectFood(food) {
    newFood.value.name = food.name || food.title || "";

    newFood.value.calories = Number(
        food.calories ?? food.kcal ?? 0
    );

    newFood.value.protein = Number(
        food.proteins ?? food.b ?? 0
    );

    newFood.value.fat = Number(
        food.fats ?? food.f ?? 0
    );

    newFood.value.carbs = Number(
        food.carbohydrates ?? food.c ?? 0
    );

    searchQuery.value = "";
}
const calculatedFood = computed(() => {
    const factor = grams.value > 0 ? grams.value / 100 : 1;

    return {
        calories: Math.round(Number(newFood.value.calories || 0) * factor),
        protein: Math.round(Number(newFood.value.protein || 0) * factor),
        fat: Math.round(Number(newFood.value.fat || 0) * factor),
        carbs: Math.round(Number(newFood.value.carbs || 0) * factor),
    };
});

const isToday = computed(() => selectedDate.value === today);

const totalCalories = computed(() =>
    foods.value.reduce((sum, item) => sum + item.calories, 0)
);

const totalProtein = computed(() =>
    foods.value.reduce((sum, item) => sum + item.protein, 0)
);

const totalFat = computed(() =>
    foods.value.reduce((sum, item) => sum + item.fat, 0)
);

const totalCarbs = computed(() =>
    foods.value.reduce((sum, item) => sum + item.carbs, 0)
);

const progressPercent = computed(() =>
    Math.min(Math.round((totalCalories.value / calorieGoal.value) * 100), 100)
);

const progressValue = computed(() =>
    Math.min((progressPercent.value / 100) * 126, 126)
);

function addFood() {
    if (!newFood.value.name.trim()) return;

    const date = selectedDate.value;

    if (!foodDiary.value[date]) {
        foodDiary.value[date] = [];
    }

    foodDiary.value[date].push({
        name: `${newFood.value.name} (${grams.value} г)`,
        ...calculatedFood.value
    });

    newFood.value = {
        name: "",
        calories: null,
        protein: null,
        fat: null,
        carbs: null,
    };

    grams.value = 100;
    searchQuery.value = "";
    showModal.value = false;
}

function removeFood(index) {
    foodDiary.value[selectedDate.value].splice(index, 1);
}

function goBack() {
    navigateTo("/");
}
</script>