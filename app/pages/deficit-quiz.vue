<template>
    <div class="mt-2 p-6">
        <div class="grid grid-cols-2 items-center mb-6">
            <div v-if="!result_for_test">
                <h1 class="text-2xl font-bold text-left text-gray-800 mb-6">
                    Тест на дефициты
                </h1>
            </div>
            <div v-else>
                <h1 class="text-2xl font-semibold">Обнаружены дефициты</h1>
                <span class="text-zinc-500 mt-2 text-sm" v-for="(deficit, index) in result_for_test.overallDeficits"
                    :key="index">
                    {{ getRightDeflectableNutrient(deficit.deficit) }}<span
                        v-if="index < result_for_test.overallDeficits.length - 1">, </span> </span>
            </div>
            <div class="flex justify-end">
                <button @click="goBack"
                    class="px-3 py-2 rounded-lg bg-gray-100 hover:bg-gray-200 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-white transition">
                    Назад
                </button>
            </div>
        </div>
        <div v-if="!questions_end">
            <QuestionCard v-if="!isLoading && questions && questions[number_of_question]" v-model="question_answer"
                :question-number="number_of_question + 1" :question-text="questions[number_of_question].text"
                :options="questions[number_of_question].options">
            </QuestionCard>

            <button v-if="question_answer" @click="nextQuestion"
                class="w-full bg-orange-300 hover:bg-orange-400 text-white font-bold py-2 px-4 rounded-lg transition mt-2 font-medium">
                Перейти к следующему вопросу
            </button>
        </div>
        <div v-if="result_for_test">
            <div class="space-y-4 mt-4">
                <div v-if="result_for_test.overallDeficits.length == 0">
                    Вы счастливчик! Никаких дефицитов витаминов у вас не обнаружено.
                </div>
                <section v-for="deficit in result_for_test.overallDeficits"
                    class="bg-white border border-gray-300 rounded-xl overflow-hidden">
                    <div class="bg-gradient-to-r from-green-600 to-green-700 px-5 py-3 mb-4">
                        <h2 class="text-white font-semibold text-lg">{{ getNutrientInfo(deficit.deficit).name }}
                        </h2>
                    </div>
                    <div class="mx-5 mb-1 p-3 rounded-lg bg-emerald-50 border border-emerald-100">
                        <p class="text-[11px] uppercase tracking-wide text-emerald-700 font-semibold mb-1">Что есть
                        </p>
                        <p class="text-zinc-800 text-sm font-medium">{{ getNutrientInfo(deficit.deficit).foods }}
                        </p>
                    </div>
                    <div class="mx-5 mt-3">
                        <p class="text-[11px] uppercase tracking-wide text-zinc-400 mb-1">Полезно</p>
                        <div class="p-3 rounded-lg bg-green-50 text-green-800 text-sm opacity-90">{{
                            getNutrientInfo(deficit.deficit).tip }}</div>
                    </div>
                    <div class="mx-5 mt-3 mb-5">
                        <p class="text-[11px] uppercase tracking-wide text-zinc-400 mb-1">Осторожно</p>
                        <div
                            class="bg-red-50 px-3 py-2 border border-red-100 rounded-lg text-xs text-red-700 opacity-90">
                            {{ getNutrientInfo(deficit.deficit).warning }}</div>
                    </div>
                </section>
            </div>
        </div>
    </div>
</template>

<script setup>
import { useFetch, useLocalStorage } from '@vueuse/core'

const question_answer = ref(null)
const { data: questions, error, isLoading } = useFetch('/deficit-questions.json').json()
const number_of_question = ref(0);
const answers = ref([])
const questions_end = ref(false)
const result_for_test = ref(null);
const savedQuizResult = useLocalStorage("deficitQuizResult", null)

function goBack() {
    navigateTo("/");
}

function nextQuestion() {
    if (questions && number_of_question.value === questions.value.length - 1) {
        result_for_test.value = analyzeDeficits(answers.value)

        savedQuizResult.value = {
            date: new Date().toISOString(),
            answers: answers.value,
            result: result_for_test.value
        }

        questions_end.value = true
        return
    }
    answers.value.push({
        type: questions.value[number_of_question.value].type,
        symptom: questions.value[number_of_question.value].symptom,
        value: question_answer.value
    });
    ++number_of_question.value;
    question_answer.value = null;
}

function analyzeDeficits(answers) {
    const grouped = groupAnswersBySymptom(answers);

    const symptomsResults = [];
    for (let symptom = 1; symptom <= 10; symptom++) {
        const result = analyzeSymptom(symptom, grouped[symptom]);
        symptomsResults.push(result);
    }

    const overallDeficits = determineOverallDeficits(symptomsResults);

    return {
        symptoms: symptomsResults,
        overallDeficits: overallDeficits,
        summary: generateSummary(symptomsResults)
    };
}

function groupAnswersBySymptom(answers) {
    const grouped = {
        1: { main: [], control: null },
        2: { main: [], control: null },
        3: { main: [], control: null },
        4: { main: [], control: null },
        5: { main: [], control: null },
        6: { main: [], control: null },
        7: { main: [], control: null },
        8: { main: [], control: null },
        9: { main: [], control: null },
        10: { main: [], control: null }
    };

    for (const answer of answers) {
        if (answer.type === 'main') {
            grouped[answer.symptom].main.push(answer);
        } else if (answer.type === 'control') {
            grouped[answer.symptom].control = answer;
        }
    }

    return grouped;
}

function analyzeSymptom(symptomId, data) {
    const mainAnswers = data.main;
    const controlAnswer = data.control;

    const rawScore = mainAnswers.reduce((sum, a) => sum + a.value, 0);
    const maxScore = mainAnswers.length * 3;

    const hasContradiction = checkContradiction(symptomId, rawScore, controlAnswer);
    const credibility = getCredibility(symptomId, hasContradiction);

    const adjustedScore = rawScore * credibility;

    const riskLevel = getRiskLevel(adjustedScore, maxScore);

    const deficits = mapSymptomToDeficits(symptomId, riskLevel);

    return {
        symptomId,
        symptomName: getSymptomName(symptomId),
        rawScore,
        maxScore,
        adjustedScore: Math.round(adjustedScore * 10) / 10,
        credibility,
        hasContradiction,
        riskLevel,
        deficits,
        recommendation: getRecommendation(riskLevel)
    };
}

function checkContradiction(symptomId, rawScore, controlAnswer) {
    if (!controlAnswer) return false;

    const controlValue = controlAnswer.value;
    const threshold = getThreshold(symptomId);

    if (rawScore < threshold) return false;

    const rules = {
        1: controlValue === 3,   // ломкие ногти, но стрижёт редко
        2: controlValue === 1,   // судороги только в ногах
        3: controlValue === 1,   // усталость, но после нагрузки легче
        4: controlValue === 1,   // раны долго, синяки быстро
        5: controlValue === 1,   // сухие губы, но пятки гладкие
        6: controlValue === 3,   // подёргивания только при движении
        7: controlValue === 1,   // частые простуды, но без врача
        8: controlValue === 3,   // плохое зрение в сумерках + очки для дали
        9: controlValue === 2 || controlValue === 3, // кровоточивость + плохая гигиена
        10: controlValue === 1   // бледность, но никто не замечает
    };

    return rules[symptomId] || false;
}

function getThreshold(symptomId) {
    const questionsCount = getMainQuestionsCount(symptomId);
    return Math.floor((questionsCount * 3) * 2 / 3);
}

function getMainQuestionsCount(symptomId) {
    const counts = {
        1: 3, 2: 3, 3: 3, 4: 3, 5: 3,
        6: 3, 7: 3, 8: 3, 9: 3, 10: 3
    };
    return counts[symptomId];
}

function getCredibility(symptomId, hasContradiction) {
    if (!hasContradiction) return 1.0;

    const softSymptoms = [2, 5, 9];
    if (softSymptoms.includes(symptomId)) return 0.7;

    return 0.4;
}

function getRiskLevel(adjustedScore, maxScore) {
    const percent = (adjustedScore / maxScore) * 100;

    if (percent < 45) return 'low';
    if (percent < 67) return 'moderate';
    return 'high';
}

function getSymptomName(symptomId) {
    const names = {
        1: 'Ломкость ногтей / выпадение волос',
        2: 'Судороги в ногах / покалывание',
        3: 'Постоянная усталость / слабость',
        4: 'Раны заживают долго',
        5: 'Сухая кожа / трещины на губах',
        6: 'Мышечные подёргивания',
        7: 'Частые простуды',
        8: 'Плохое зрение в сумерках',
        9: 'Кровоточивость дёсен',
        10: 'Бледность кожи'
    };
    return names[symptomId];
}

function mapSymptomToDeficits(symptomId, riskLevel) {
    if (riskLevel === 'low') return [];

    const deficitsMap = {
        1: ['Железо', 'Биотин', 'Цинк'],
        2: ['Магний', 'Калий', 'Кальций'],
        3: ['Железо', 'Витамин B12', 'Фолат (B9)'],
        4: ['Цинк', 'Витамин C'],
        5: ['Витамин B2', 'Витамин A', 'Витамин E'],
        6: ['Магний', 'Кальций'],
        7: ['Витамин D', 'Витамин C', 'Цинк'],
        8: ['Витамин A'],
        9: ['Витамин C'],
        10: ['Железо', 'Витамин B12']
    };

    return deficitsMap[symptomId] || [];
}

function getRecommendation(riskLevel) {
    if (riskLevel === 'low') return 'В пределах нормы';
    if (riskLevel === 'moderate') return 'Рекомендуется обратить внимание на питание';
    return 'Рекомендуется сдать анализы крови';
}

function determineOverallDeficits(symptomsResults) {
    const deficitCount = {};

    for (const symptom of symptomsResults) {
        if (symptom.riskLevel === 'high' || symptom.riskLevel === 'moderate') {
            for (const deficit of symptom.deficits) {
                deficitCount[deficit] = (deficitCount[deficit] || 0) + 1;
            }
        }
    }

    const sorted = Object.entries(deficitCount)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 4)
        .map(([deficit, count]) => ({ deficit, symptomsCount: count }));

    return sorted;
}

function generateSummary(symptomsResults) {
    const highRiskSymptoms = symptomsResults.filter(s => s.riskLevel === 'high');
    const moderateRiskSymptoms = symptomsResults.filter(s => s.riskLevel === 'moderate');
    const contradictions = symptomsResults.filter(s => s.hasContradiction);

    let summary = '';

    if (highRiskSymptoms.length > 0) {
        summary += `Высокий риск дефицита по симптомам: ${highRiskSymptoms.map(s => s.symptomName).join(', ')}.\n`;
    }

    if (moderateRiskSymptoms.length > 0) {
        summary += `Средний риск: ${moderateRiskSymptoms.map(s => s.symptomName).join(', ')}.\n`;
    }

    if (contradictions.length > 0) {
        summary += `Обнаружены противоречия в ответах по симптомам: ${contradictions.map(s => s.symptomName).join(', ')}. Возможно, проблема не связана с дефицитом витаминов.\n`;
    }

    if (highRiskSymptoms.length === 0 && moderateRiskSymptoms.length === 0) {
        summary = 'По результатам опроса значительных признаков дефицита витаминов и минералов не обнаружено.';
    }

    return summary;
}

function getNutrientInfo(nutrient) {
    const nutrientsInfo = {
        'Железо': {
            name: 'Железо',
            foods: 'Красное мясо, печень, шпинат, гречка, бобовые',
            tip: 'Сочетайте с витамином C (цитрусовые, болгарский перец) для лучшего усвоения',
            warning: 'Избегайте чая и кофе в течение 1 часа после еды с железом'
        },
        'Биотин': {
            name: 'Биотин (Витамин B7, H)',
            foods: 'Яичный желток, печень, орехи, лосось, авокадо, цветная капуста',
            tip: 'Важен для красоты волос и ногтей',
            warning: 'Сырые яичные белки мешают усвоению биотина'
        },
        'Цинк': {
            name: 'Цинк',
            foods: 'Устрицы, говядина, тыквенные семечки, чечевица',
            tip: 'Важен для иммунитета и заживления ран',
            warning: 'Кофеин и фитаты (зерновые) снижают усвоение'
        },
        'Магний': {
            name: 'Магний',
            foods: 'Орехи (миндаль, кешью), семена тыквы, какао, авокадо, зелень',
            tip: 'Принимайте вечером — помогает от судорог и улучшает сон',
            warning: 'Алкоголь и мочегонные выводят магний'
        },
        'Калий': {
            name: 'Калий',
            foods: 'Бананы, курага, картофель (в мундире), авокадо, шпинат, фасоль',
            tip: 'Важен для сердца и мышц',
            warning: 'Избыток соли выводит калий из организма'
        },
        'Кальций': {
            name: 'Кальций',
            foods: 'Молочные продукты, кунжут, сардины, миндаль, брокколи',
            tip: 'Для усвоения нужен витамин D и магний',
            warning: 'Кофеин и фитиновая кислота мешают усвоению'
        },
        'Витамин B12': {
            name: 'Витамин B12 (Кобаламин)',
            foods: 'Печень, мясо, рыба, яйца, молочные продукты',
            tip: 'Веганам нужно принимать синтетический B12',
            warning: 'Дефицит долго не проявляется — до 5 лет'
        },
        'Фолат (B9)': {
            name: 'Фолат (Витамин B9)',
            foods: 'Листовая зелень, спаржа, брокколи, бобовые, авокадо',
            tip: 'Критически важен для беременных',
            warning: 'Алкоголь сильно снижает уровень фолата'
        },
        'Витамин C': {
            name: 'Витамин C',
            foods: 'Шиповник, облепиха, киви, цитрусовые, болгарский перец',
            tip: 'Улучшает усвоение железа в 6 раз!',
            warning: 'Разрушается при нагревании выше 60°C'
        },
        'Витамин B2': {
            name: 'Витамин B2 (Рибофлавин)',
            foods: 'Печень, грибы, миндаль, творог, яйца',
            tip: 'Не разрушается при варке, но боится света',
            warning: 'Храните молочные продукты в тёмной таре'
        },
        'Витамин A': {
            name: 'Витамин A',
            foods: 'Морковь, тыква, печень трески, яичный желток',
            tip: 'Для усвоения нужны жиры (сметана, масло)',
            warning: 'Передозировка токсична — не превышайте норму'
        },
        'Витамин E': {
            name: 'Витамин E',
            foods: 'Растительные масла, орехи, семена, авокадо, шпинат',
            tip: 'Антиоксидант — защищает клетки от старения',
            warning: 'При дефиците жиров плохо усваивается'
        },
        'Витамин D': {
            name: 'Витамин D',
            foods: 'Жирная рыба (лосось, скумбрия), яичный желток, печень трески',
            tip: 'Вырабатывается на солнце — гуляйте 20 минут в день',
            warning: 'Зимой и в северных широтах нужен приём добавок'
        }
    };
    return nutrientsInfo[nutrient];
}

function getRightDeflectableNutrient(nutrient) {
    const nutrientsInfo = {
        'Железо': 'железа',
        'Биотин': 'биотина',
        'Цинк': 'цинка',
        'Магний': 'магния',
        'Калий': 'калия',
        'Кальций': 'кальция',
        'Витамин B12': 'витамина B12',
        'Фолат (B9)': 'фолата (B9)',
        'Витамин C': 'витамина C',
        'Витамин B2': 'витамина B2',
        'Витамин A': 'витамина A',
        'Витамин E': 'витамина E',
        'Витамин D': 'витамина D'
    };
    return nutrientsInfo[nutrient];
}

</script>