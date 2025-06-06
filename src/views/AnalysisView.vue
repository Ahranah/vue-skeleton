<script setup>
import { useTransactionStore } from '@/stores/transactionStore'
import { useTransactionCategoryStore } from '@/stores/transactionCategoryStore'
import { TRANSACTION_TYPE } from '@/types'
import { computed, reactive } from 'vue'
import PieChart from '@/components/PieChart.vue'
import AnalysisList from '@/components/AnalysisList.vue'

const transactionStore = useTransactionStore()
const transactionCategoryStore = useTransactionCategoryStore()
const transactions = computed(() => transactionStore.states.transactions)
const transactionCategories = computed(() => transactionCategoryStore.states.transactionCategories)

const states = reactive({
  period: 1,
  transactionType: 'income',
})

const expenses = computed(() => {
  const now = new Date()
  const past = new Date(now.getTime() - states.period * 30 * 24 * 60 * 60 * 1000) // N개월 전 (30일 기준)

  return transactions.value
    .filter(
      (t) => t.typeId === TRANSACTION_TYPE.expense && new Date(t.date).getTime() >= past.getTime(),
    )
    .sort((a, b) => {
      return new Date(b.date).getTime() - new Date(a.date).getTime()
    })
})

const incomes = computed(() => {
  const now = new Date()
  const past = new Date(now.getTime() - states.period * 30 * 24 * 60 * 60 * 1000) // N개월 전 (30일 기준)

  return transactions.value
    .filter(
      (t) => t.typeId === TRANSACTION_TYPE.income && new Date(t.date).getTime() >= past.getTime(),
    )
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime())
})

const categorialExpense = computed(() =>
  expenses.value.reduce((res, e) => {
    if (!res[e.categoryId]) res[e.categoryId] = 0
    res[e.categoryId] += e.amount
    return res
  }, {}),
)

const categorialIncome = computed(() =>
  incomes.value.reduce((res, e) => {
    if (!res[e.categoryId]) res[e.categoryId] = 0
    res[e.categoryId] += e.amount
    return res
  }, {}),
)

const total = computed(() => {
  return (
    incomes.value.reduce((sum, i) => sum + i.amount, 0) -
    expenses.value.reduce((sum, i) => sum + i.amount, 0)
  ).toLocaleString()
})

const totalIncome = computed(() => {
  return incomes.value.reduce((sum, i) => sum + i.amount, 0).toLocaleString()
})

const totalExpense = computed(() => {
  return expenses.value.reduce((sum, i) => sum + i.amount, 0).toLocaleString()
})

const getCategoryName = (categoryId) => {
  if (transactionCategories.value.length === 0) {
    return 'loading'
  }

  return transactionCategories.value.find(
    (transactionCategory) => transactionCategory.id === categoryId,
  ).name
}
</script>

<template>
  <div class="card py-3 px-lg-5 shadow-sm">
    <div class="card-body">
      <div class="mb-4">
        <span class="fs-4 me-2">총 수익</span>
        <span class="fs-2">{{ total }} 원</span>
      </div>

      <div class="mb-4 d-flex justify-content-between">
        <div>
          <select class="form-select" v-model="states.period">
            <option value="1">1개월</option>
            <option value="3">3개월</option>
            <option value="6">6개월</option>
          </select>
        </div>
        <div class="d-flex">
          <button class="btn btn-warning me-1" @click="states.transactionType = 'income'">
            수익
          </button>
          <button class="btn btn-warning" @click="states.transactionType = 'expense'">지출</button>
        </div>
      </div>

      <hr class="" />

      <div class="mb-4 px-md-4">
        <div class="mb-4">
          <span class="fs-5 me-2">
            {{ states.transactionType === 'income' ? '순 수익' : '순 지출' }}
          </span>
          <span class="fs-4">
            {{ states.transactionType === 'income' ? totalIncome : totalExpense }}
            원
          </span>
        </div>

        <div class="row flex-column flex-md-row">
          <div class="col-lg-6 mb-3 mb-lg-0">
            <template v-if="states.transactionType === 'expense'">
              <PieChart
                :series="Object.values(categorialExpense)"
                :labels="Object.keys(categorialExpense).map(getCategoryName)"
              />
            </template>
            <template v-if="states.transactionType === 'income'">
              <PieChart
                :series="Object.values(categorialIncome)"
                :labels="Object.keys(categorialIncome).map(getCategoryName)"
              />
            </template>
          </div>

          <div class="col-lg-6 px-2">
            <template v-if="states.transactionType === 'income'">
              <AnalysisList :categorialTransaction="categorialIncome" />
            </template>
            <template v-if="states.transactionType === 'expense'">
              <AnalysisList :categorialTransaction="categorialExpense" />
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
