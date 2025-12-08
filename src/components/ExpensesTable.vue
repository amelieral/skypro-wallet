<template>
  <div class="table-container">

    <h2 class="table-title">Таблица расходов</h2>

    <table>
      <thead>
        <tr>
          <th>Описание</th>
          <th>Категория</th>
          <th>Дата</th>
          <th>Сумма</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="expense in processedExpenses" :key="expense.id">
          <td>{{ expense.description }}</td>
          <td>{{ expense.translatedCategory }}</td>
          <td>{{ expense.formattedDate }}</td>
          <td>{{ expense.amount.toLocaleString() }} ₽</td>
          <td>
            <img
              src="@/assets/icon/food-del.svg"
              alt="Удалить"
              class="delete-icon"
              @click="handleDeleteExpense(expense.id)"
            />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { expensesStore } from '@/store/expenses'

const categoryTranslations = {
  food: 'Еда',
  transport: 'Транспорт',
  housing: 'Жилье',
  entertainment: 'Развлечения',
  education: 'Образование',
  other: 'Другое',
}

const formatDate = (isoDate) => {
  try {
    const options = {
      day: '2-digit',
      month: '2-digit',
      year: 'numeric',
      timeZone: 'UTC',
    }
    return new Date(isoDate).toLocaleDateString('ru-RU', options)
  } catch {
    return isoDate
  }
}

const processedExpenses = computed(() => {
  return [...expensesStore.state.value]
    .sort((a, b) => new Date(b.date) - new Date(a.date))
    .map((expense) => ({
      ...expense,
      translatedCategory: categoryTranslations[expense.category] || expense.category,
      formattedDate: formatDate(expense.date),
    }))
})

onMounted(() => {
  expensesStore.getExpenses()
})

const handleDeleteExpense = async (id) => {
  if (confirm('Вы уверены, что хотите удалить эту запись?')) {
    try {
      await expensesStore.deleteExpense(id)
    } catch (err) {
      if (err.message === 'Транзакция уже удалена') {
        alert('Эта транзакция уже была удалена')
      } else {
        alert('Произошла ошибка при удалении')
      }
    }
  }
}
</script>

<style scoped>
.table-container {
  flex: 1;
  background: white;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 24px;
  overflow-x: auto;
}

.table-container table {
  width: 100%;
  min-width: 379px;
  border-collapse: collapse;
}

.table-title {
  font-family: 'Montserrat', sans-serif;
  font-weight: 700;
  font-size: 24px;
  line-height: 1;
  letter-spacing: 0;
  margin-left: 15px;
  margin-bottom: 32px;
  color: #000000;
}

.table-container thead {
  border-bottom: 1px solid #e2e8f0;
}

.table-container th {
  padding:12px 16px;
  vertical-align: middle;
  text-align: left;
  white-space: nowrap;
  color: #999999;
  font-weight: 400;
}

.table-container td {
  padding: 12px 16px;
  vertical-align: middle;
  text-align: left;
  white-space: nowrap;
}

.table-container tr:last-child td {
  border-bottom: none;
}

.delete-icon {
  width: 20px;
  height: 20px;
  cursor: pointer;
  transition: opacity 0.3s;
  min-width: 20px;
}

.delete-icon:hover {
  opacity: 0.7;
}

.table-container th:last-child,
.table-container td:last-child {
  width: 40px;
  text-align: center;
  padding-right: 8px;
}
</style>
