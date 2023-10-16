<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from "vue";

const API_ENDPOINT = "https://api.frontendeval.com/fake/crypto/";
type JSONResponse = { value: number };
const REFRESH_TIME = 10000; // 10 seconds

const currencies = ["USD", "EUR", "GBP", "CNY", "JPY"] as const;
type Currency = (typeof currencies)[number];

const currentValue = ref<number>();
const amount = ref<number>();
const currency = ref<Currency>(currencies[0]);
let lastValue: number | undefined = undefined;
let intervalId: number | null = null;

const getRoundedValue = (value: number) => Math.round(value * 100) / 100;

const fetchValue = async (amount: number, currency: Currency) => {
  if (!amount || !currency) return;

  const response = await fetch(`${API_ENDPOINT}${currency}`);
  if (response.ok) {
    const data = (await response.json()) as JSONResponse;

    lastValue = currentValue.value;
    currentValue.value = getRoundedValue(data.value * amount);
    return;
  }

  // TODO: error handling
};

const changedValue = computed(() => {
  if (currentValue.value && lastValue) {
    return getRoundedValue(currentValue.value - lastValue);
  }

  return 0;
});

const displayChangedValue = computed(() => {
  if (changedValue.value) {
    return changedValue.value > 0
      ? `↑ ${changedValue.value}`
      : `↓ ${changedValue.value}`;
  }

  return "";
});

watch([amount, currency, amount], async ([newAmount, newCurrency]) => {
  lastValue = undefined;
  currentValue.value = undefined;

  if (intervalId) {
    clearInterval(intervalId);
    intervalId = null;
  }

  if (newAmount && newAmount > 0) {
    await fetchValue(newAmount, newCurrency);
    intervalId = setInterval(() => {
      amount.value && fetchValue(amount.value, currency.value);
    }, REFRESH_TIME);
  }
});

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }
});
</script>

<template>
  <form class="inputForm">
    <input v-model="amount" />

    <select v-model="currency">
      <option v-for="currency in currencies" :value="currency">
        {{ currency }}
      </option>
    </select>
  </form>

  <div class="result" v-if="currentValue">
    <p>{{ currentValue }} WUC</p>
    <p
      v-if="changedValue"
      :class="{
        valueUp: changedValue > 0,
        valueDown: changedValue < 0,
      }"
    >
      {{ displayChangedValue }}
    </p>
  </div>
</template>

<style scoped>
.inputForm {
  display: flex;
  justify-content: center;
  gap: 4px;
}

.result {
  display: flex;
  justify-content: center;
  gap: 16px;
}

.valueUp {
  color: green;
}

.valueDown {
  color: red;
}
</style>
