<template>
  <div class="mt-2 p-6">
    <h1 class="text-2xl font-bold text-center text-gray-800 mb-6">Заполните информацию о себе</h1>
    <div сlass="flex gap-4 mb-6">
      <div class="bg-white border border-gray-300 rounded-lg p-4">
        <div>
          <label class="block text-gray-900 font-medium mb-1">Как вас зовут?</label>
          <input type="text" v-model.number="name"
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none"
            placeholder="Все начинается с знакомства" />
        </div>
        <div class="mt-6">
          <label class="block text-gray-900 font-medium mb-1">Пол</label>
          <div class="flex gap-4">
            <label class="inline-flex items-center gap-2">
              <input type="radio" name="gender" value="male" v-model="gender" class="w-4 h-4 text-green-600" />
              <span>Мужской</span>
            </label>

            <label class="inline-flex items-center gap-2">
              <input type="radio" name="gender" value="female" v-model="gender" class="w-4 h-4 text-green-600" />
              <span>Женский</span>
            </label>
          </div>
        </div>
        <div class="mt-6">
          <label class="block text-gray-900 font-medium mb-1">Возраст (лет)</label>
          <input type="number" v-model.number="age"
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none"
            placeholder="Введите возраст" />
        </div>
        <div class="mt-6">
          <label class="block text-gray-900 font-medium mb-1">Рост (см)</label>
          <input type="number" v-model.number="height"
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none"
            placeholder="Введите рост в см" />
        </div>
        <div class="mt-6">
          <label class="block text-gray-900 font-medium mb-1">Вес (кг)</label>
          <input type="number" v-model.number="weight"
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:outline-none"
            placeholder="Введите вес в кг" />
        </div>

      </div>
    </div>
    <div сlass="flex gap-4 mb-6">
      <h3 class="text-l font-bold text-center text-gray-800 mb-6 mt-8">Ответьте на 3 вопроса, чтобы определить Ваш
        уровень активности</h3>

      <QuestionCard v-model="q1" :question-number="1"
        question-text="Как часто вы занимаетесь спортом или физическими упражнениями?" :options="[
          { value: '1', label: 'Практически никогда' },
          { value: '2', label: '1-2 раза в неделю (лёгкие прогулки, зарядка)' },
          { value: '3', label: '3-4 раза в неделю (фитнес, бег, велосипед)' },
          { value: '4', label: '5-6 раз в неделю или каждый день' }
        ]" />

      <QuestionCard v-model="q2" :question-number="2" question-text="Сколько шагов вы проходите в день в среднем?"
        :options="[
          { value: '1', label: 'Менее 3 000 шагов (сидячая работа)' },
          { value: '2', label: '3 000 - 7 000 шагов' },
          { value: '3', label: '8 000 - 12 000 шагов' },
          { value: '4', label: 'Более 15 000 шагов' }
        ]" />

      <QuestionCard v-model="q3" :question-number="3" question-text="Какой у вас образ жизни и работа?" :options="[
        { value: '1', label: 'Сидячая работа, передвижение на авто/транспорте' },
        { value: '2', label: 'Работа стоя или активное движение по дому' },
        { value: '3', label: 'Активная работа / учёба с движением' },
        { value: '4', label: 'Тяжёлая физическая работа или профессиональный спорт' }
      ]" />

    </div>
    <button @click="process"
      class="w-full bg-orange-300 hover:bg-orange-400 text-white font-bold py-2 px-4 rounded-lg transition mt-2 font-medium">
      Завершить опрос
    </button>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { useLocalStorage } from "@vueuse/core";

const height = ref(null);
const weight = ref(null);
const sex = ref("male");
const age = ref(null);
const name = ref(null);

const bmi = ref(null);
const bmr = ref(null);
const tdee = ref(null);

const q1 = ref(null);
const q2 = ref(null);
const q3 = ref(null);
const activityResult = ref(null);
const activityError = ref('');

function process() {
  calculateBMI();
  calculateActivity();
  calculateTDEE();
  saveStatsInLocalStorage();
  navigateTo("/");
}

const calculateBMI = () => {
  let bmiValue = null;
  const w = weight.value;
  const h = height.value / 100;
  if (w > 0 && h > 0) {
    bmiValue = w / (h * h);
  }
  bmi.value = bmiValue ? bmiValue.toFixed(1) : null;
};

function saveStatsInLocalStorage() {
  const res = useLocalStorage('biodata', {
    Name: null,
    BMI: null,
    Height: null,
    Weight: null,
    Age: null,
    Sex: null,
    Activity: null,
    BMR: null,
    TDEE: null
  });

  res.value = {
    Name: name,
    BMI: bmi.value,
    Height: height.value,
    Weight: weight.value,
    Age: age,
    Sex: sex,
    Activity: activityResult.value,
    BMR: bmr.value,
    TDEE: tdee.value
  };
}

function calculateTDEE() {
  if (!weight.value || !height.value || !age.value || !activityResult.value) {
    return;
  }

  let bmrValue;

  if (sex.value === "male") {
    bmrValue = 10 * weight.value + 6.25 * height.value - 5 * age.value + 5;
  } else {
    bmrValue = 10 * weight.value + 6.25 * height.value - 5 * age.value - 161;
  }

  bmr.value = Math.round(bmrValue);

  const pal = activityResult.value.pal;
  tdee.value = Math.round(bmrValue * pal);
}

const bmiColorClass = computed(() => {
  const val = parseFloat(bmi.value);
  if (isNaN(val)) return "";
  if (val < 18.5) return "bg-yellow-100 text-yellow-800";
  if (val < 25) return "bg-green-100 text-green-800";
  if (val < 30) return "bg-orange-100 text-orange-800";
  return "bg-red-100 text-red-800";
});

function getPALFromScore(score) {
  const map = { 1: 1.2, 2: 1.4, 3: 1.7, 4: 2.0 };
  return map[score] || 1.2;
}

function calculateActivity() {
  activityError.value = '';
  activityResult.value = null;

  if (q1.value === null || q2.value === null || q3.value === null) {
    activityError.value = 'Пожалуйста, ответьте на все 3 вопроса.';
    return;
  }

  const v1 = Number(q1.value);
  const v2 = Number(q2.value);
  const v3 = Number(q3.value);

  const pal1 = getPALFromScore(v1);
  const pal2 = getPALFromScore(v2);
  const pal3 = getPALFromScore(v3);

  let pal = (pal1 + pal2 + pal3) / 3;
  pal = Math.round(pal * 100) / 100;

  if (v3 === 4 && v1 >= 3 && pal < 1.9) {
    pal = Math.min(2.2, pal + 0.1);
    pal = Math.round(pal * 100) / 100;
  }
  if (v3 === 1 && v1 === 1 && v2 === 1 && pal > 1.25) {
    pal = 1.2;
  }
  if (v2 === 4 && v3 >= 3 && pal > 1.9) {
    pal = Math.min(2.3, pal + 0.05);
    pal = Math.round(pal * 100) / 100;
  }
  if (pal < 1.2) pal = 1.2;
  if (pal > 2.5) pal = 2.5;

  let description = '';
  if (pal < 1.4) description = 'Низкая активность (сидячий образ жизни). Рекомендуется больше ходьбы и лёгкая зарядка.';
  else if (pal < 1.6) description = 'Ниже среднего (лёгкая активность, редкие прогулки). Для поддержания формы увеличьте число шагов.';
  else if (pal < 1.8) description = 'Умеренная активность (регулярные прогулки, спорт 1-3 раза в неделю). Хороший баланс.';
  else if (pal < 2.0) description = 'Выше среднего (активный образ жизни, регулярный фитнес/бег). Отличная физическая форма.';
  else description = 'Высокая / экстремальная активность (ежедневные тренировки, физическая работа или спорт высоких достижений).';

  if (v1 === 4 && v2 === 4 && v3 === 4) description += ' — Максимальный уровень!';
  else if (v1 === 1 && v2 === 1 && v3 === 1) description += ' — Минимальная активность, рекомендуется больше движения.';

  activityResult.value = { pal, description };
}


</script>