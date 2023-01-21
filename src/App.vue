<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightChartEl = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref(0);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => weight.weight)
        .slice(-7);

      weightChart.value.update();
      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((weight) => new Date(weight.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: ws
                .sort((a, b) => a.date - b.date)
                .map((weight) => weight.weight),
              backgroundColor: "rgba(99, 102, 241, 0.2)",
              borderColor: "rgba(99, 102, 241, 1)",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main class="max-w-3xl mx-auto p-6">
    <h1 class="text-4xl font-bold text-center mb-8">Basic Weight Tracker</h1>

    <div
      class="rounded-full border-8 border-indigo-500 mx-auto mb-6 text-xl flex flex-col justify-center items-center w-48 h-48 text-center bg-white shadow-md"
    >
      <span class="block font-bold mb-2">{{ currentWeight.weight }}</span>
      <small class="italic text-zinc-500">Current Weight (kg)</small>
    </div>

    <form
      class="flex mb-8 rounded-lg overflow-hidden border-2 border-zinc-500 ease-in duration-200 hover:border-indigo-500 focus-within:border-indigo-500"
      @submit.prevent="addWeight"
    >
      <input
        type="number"
        step="0.1"
        v-model="weightInput"
        class="appearance-none bg-white text-xl px-8 py-4 grow"
      />

      <input
        type="submit"
        value="Add weight"
        class="appearance-none cursor-pointer text-white text-xl font-bold hover:bg-white ease-in duration-200 hover:text-indigo-500 hover:border-l-indigo-500 px-8 py-4 bg-indigo-500 hover:border-l-2"
      />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2 class="mb-4 font-normal text-zinc-500">Last 7 days</h2>

      <div class="w-full bg-white p-4 rounded-lg mb-8 max-w-3xl shadow-md">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history p-0 m-0">
        <h2 class="mb-4 font-normal text-zinc-500">Weight History</h2>
        <ul class="[&>*:nth-child(even)]:bg-neutral-200">
          <li
            class="flex justify-between items-center p-2 cursor-pointer hover:bg-white"
            v-for="weight in weights"
          >
            <span class="block text-xl font-bold mr-4"
              >{{ weight.weight }}kg</span
            >
            <small class="italic text-zinc-500">
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>
