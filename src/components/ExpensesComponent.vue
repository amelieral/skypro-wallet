<template>
  <div class="container">
    <h1>Мои расходы</h1>
    <div class="content-wrapper">
      <ExpensesTable />

      <NewExpenseModal />
    </div>
  </div>
</template>

<script setup>
import ExpensesTable from './ExpensesTable.vue'
import NewExpenseModal from './NewExpenseModal.vue'
</script>

<style scoped>
.container {
  max-width: 1600px;
  margin: 0 auto;
  padding: 30px;
  min-height: 100vh;
}

h1 {
  font-size: 32px;
  color: #000000;
  margin-bottom: 30px;
  font-weight: 700;
}

.content-wrapper {
  display: flex;
  gap: 30px;
  align-items: flex-start;
}
</style>
