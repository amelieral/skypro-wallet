<template>
  <div class="new-expense-modal">
    <h2>Новый расход</h2>
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label>Описание <span v-if="errors.description" class="error-star">*</span></label>
        <input
          v-model="formData.description"
          type="text"
          placeholder="Введите описание"
          required
          :class="{
            invalid: errors.description,
            valid: formData.description.trim().length > 0 && !errors.description,
          }"
          @blur="validateField('description')"
        />
      </div>

      <div class="form-group">
        <label>Категория<span v-if="errors.description" class="error-star">*</span></label>
        <div class="category-grid">
          <div
            v-for="category in categories"
            :key="category.name"
            class="category-card"
            :class="{ active: formData.category === category.name }"
            @click="formData.category = category.name"
          >
            <img :src="category.icon" alt="" class="category-icon" />
            <span class="category-name">{{ category.name }}</span>
          </div>
        </div>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label>Дата<span v-if="errors.description" class="error-star">*</span></label>
          <input
            v-model="formData.date"
            type="date"
            required
            placeholder="Введите дату"
            class="date-input"
            :class="{
              invalid: errors.amount,
              valid: formData.amount && !errors.amount,
            }"
            @blur="validateField('date')"
          />
        </div>

        <div class="form-group">
          <label>Сумма<span v-if="errors.description" class="error-star">*</span></label>
          <input
            v-model.number="formData.amount"
            type="number"
            min="1"
            placeholder="Введите сумму"
            required
            :class="{
              invalid: errors.amount,
              valid: formData.amount && !errors.amount,
            }"
            @blur="validateField('amount')"
          />
        </div>
      </div>

      <p v-if="error" class="error">{{ error }}</p>
      <button type="submit" class="submit-button" :disabled="isDisabled">Добавить новый расход</button>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { expensesStore } from '@/store/expenses'
import foodIcon from '@/assets/icon/food.svg'
import carIcon from '@/assets/icon/car.svg'
import houseIcon from '@/assets/icon/house.svg'
import gameIcon from '@/assets/icon/game.svg'
import studyIcon from '@/assets/icon/study.svg'
import messageIcon from '@/assets/icon/message.svg'

const formData = ref({
  description: '',
  category: 'Еда',
  date: '',
  amount: null,
})

const error = ref('')

const errors = ref({
  description: false,
  date: false,
  amount: false,
})

const categories = [
  { name: 'Еда', icon: foodIcon },
  { name: 'Транспорт', icon: carIcon },
  { name: 'Жилье', icon: houseIcon },
  { name: 'Развлечения', icon: gameIcon },
  { name: 'Образование', icon: studyIcon },
  { name: 'Другое', icon: messageIcon },
]

const categoryMapping = {
  Еда: 'food',
  Транспорт: 'transport',
  Жилье: 'housing',
  Развлечения: 'joy',
  Образование: 'education',
  Другое: 'others',
}

const isDisabled = computed(() => {
  return (
    Object.values(errors.value).some((err) => err) ||
    !formData.value.description.trim() ||
    !formData.value.date ||
    !formData.value.amount
  )
})

const validateField = (field) => {
  switch (field) {
    case 'description': {
      const descValue = formData.value.description.trim()
      errors.value.description = !descValue || descValue.length < 4
      break
    }
    case 'amount': {
      const amountValue = formData.value.amount
      errors.value.amount = !amountValue || amountValue <= 0 || isNaN(amountValue)
      break
    }
    case 'date':
      errors.value.date = !formData.value.date
      break
  }
}

const validateForm = () => {
  validateField('description')
  validateField('amount')
  validateField('date')
  return Object.values(errors.value).every((err) => !err)
}

const handleSubmit = async () => {
  if (!validateForm()) {
    error.value = 'Пожалуйста, заполните все обязательные поля корректно'
    return
  }

  try {
    await expensesStore.addExpense({
      description: formData.value.description.trim(),
      category: categoryMapping[formData.value.category],
      date: formData.value.date,
      sum: formData.value.amount,
    })

    resetForm()
    error.value = ''
    alert('Расход успешно добавлен!')
  } catch (err) {
    if (err.response && err.response.data) {
      error.value = err.response.data.message || 'Произошла ошибка'
    } else {
      error.value = err.message || 'Произошла ошибка при сохранении расхода'
    }
  }
}

const resetForm = () => {
  formData.value = {
    description: '',
    category: 'Еда',
    date: '',
    amount: null,
  }
  errors.value = {
    description: false,
    date: false,
    amount: false,
  }
}
</script>

<style scoped>
.new-expense-modal {
  background: #ffffff;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
  max-width: 500px;
  width: 379px;
}

.new-expense-modal h2 {
  font-size: 24px;
  font-weight: 700;

  margin-bottom: 24px;
  color: #000000;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  font-size: 16px;
  color: #000000;
}

.error-star {
  color: #dc2626; /* красный */
  margin-left: 3px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 12px 16px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  font-size: 12px;
  transition: all 0.2s;
  box-sizing: border-box;
  height: 48px;
    font-family: 'Montserrat', sans-serif;
}

.submit-button:disabled {
  background: #999999;
  cursor: not-allowed;
  opacity: 0.8;
}

.form-group input:focus {
  border-color: #6366f1;
  outline: none;
  box-shadow: 0 0 0 2px rgba(99, 102, 241, 0.2);
}

.form-group input.valid {
  background: #f3ebff;
  box-shadow: 0 0 0 1px #7334ea;
  border: 1px solid transparent;
}

.form-group input.valid:focus {
  border-color: #7334ea;
  box-shadow: 0 0 0 2px rgba(115, 52, 234, 0.2);
}

.form-group input.invalid {
  border-color: #dc2626;
  background: #fef2f2;
  animation: shake 0.4s ease-in-out;
}

.form-group input.invalid:focus {
  box-shadow: 0 0 0 2px rgba(220, 38, 38, 0.2);
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-5px);
  }
  50% {
    transform: translateX(5px);
  }
  75% {
    transform: translateX(-3px);
  }
}

.category-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  justify-content: flex-start;
}

.category-card {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 12px;
  height: 31px;
  min-width: 60px;
  padding: 8px 20px;
  background: #f4f5f6;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.2s;
  box-sizing: border-box;
  border: 1px solid transparent;
}

.category-card:hover .category-icon {
  color: #7334ea;
}

.category-card.active {
  background: rgba(115, 52, 234, 0.1);
  border-color: #7334ea;
}

.category-card.active .category-icon,
.category-card.active .category-name {
  color: #7334ea;
}

.category-icon {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  color: #333;
  transition: color 0.2s;
}

.category-name {
  font-size: 14px;
  color: #333;
  white-space: nowrap;
  transition: color 0.2s;
}

.form-row {
  display: block;
}

.submit-button {
  width: 100%;
  padding: 10px 20px;
  background: #7334ea;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  height: 48px;
  box-sizing: border-box;
  font-family: 'Montserrat', sans-serif;
}
</style>
