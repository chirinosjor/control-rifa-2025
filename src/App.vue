<template>
  <div>
    <h1>Control Rifa Navidad 2025</h1>

    <!-- Hidden iframe for invisible submission -->
    <iframe name="submitFrame" style="display: none"></iframe>

    <!-- Hidden form targeting the iframe -->
    <form :action="scriptUrl" method="POST" target="submitFrame" ref="formRef">
      <input type="hidden" name="userName" v-model="userName" />
      <input type="hidden" name="userId" v-model="userId" />
      <input type="hidden" name="dateOfPayment" v-model="dateOfPayment" />
      <input type="hidden" name="quantity" v-model="quantity" />
      <input type="hidden" name="amount" v-model="amount" />
    </form>

    <!-- Visible Vue inputs -->
    <div class="form-container">
      <div class="input-group">
        <label for="userName">Nombre:</label>
        <input
          id="userName"
          v-model="userName"
          type="text"
          required
          @blur="userNameTouched = true"
        />
        <span class="error" v-if="userNameError">{{ userNameError }}</span>
      </div>

      <div class="input-group">
        <label for="userId">Número de talonario:</label>
        <input
          id="userId"
          v-model.number="userId"
          type="number"
          required
          @blur="userIdTouched = true"
        />
        <span class="error" v-if="userIdError">{{ userIdError }}</span>
      </div>

      <div class="input-group">
        <label for="dateOfPayment">Fecha del pago (DD/MM):</label>
        <input
          id="dateOfPayment"
          v-model="dateInputFormatted"
          type="text"
          placeholder="DD/MM"
          required
          @blur="dateTouched = true"
        />
        <span class="error" v-if="dateError">{{ dateError }}</span>
      </div>

      <div class="input-group">
        <label for="amount">Monto:</label>
        <input
          id="amount"
          v-model.number="amount"
          type="number"
          min="0"
          step="0.01"
          required
          @blur="amountTouched = true"
        />
        <span class="error" v-if="amountError">{{ amountError }}</span>
      </div>

      <div class="input-group">
        <label for="quantity">Cantidad de números:</label>
        <input
          id="quantity"
          v-model.number="quantity"
          type="number"
          min="1"
          required
          @blur="quantityTouched = true"
        />
        <span class="error" v-if="quantityError">{{ quantityError }}</span>
      </div>

      <button @click="submitForm" :disabled="!isFormValid">Enviar</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const userName = ref('')
const userId = ref(0)
const dateOfPayment = ref('')
const dateInput = ref('')
const quantity = ref(0)
const amount = ref(0)

const userNameTouched = ref(false)
const userIdTouched = ref(false)
const dateTouched = ref(false)
const amountTouched = ref(false)
const quantityTouched = ref(false)

// Your deployed Apps Script URL
const scriptUrl =
  'https://script.google.com/macros/s/AKfycbzyJkLPx4FKgig3mzrwUZOT81AjawwqJ23BQdkXOkTpbyWfj7QkoGOY1pFWb2922LNL0w/exec'
const formRef = ref<HTMLFormElement>()

const dateInputFormatted = computed({
  get: () => dateInput.value,
  set: (value) => {
    let cleaned = value.replace(/\D/g, '').slice(0, 4)
    if (cleaned.length >= 2) {
      cleaned = cleaned.slice(0, 2) + '/' + cleaned.slice(2)
    }
    dateInput.value = cleaned
  },
})

const userNameError = computed(() =>
  !userNameTouched.value ? '' : userName.value.trim() === '' ? 'Nombre requerido' : '',
)
const userIdError = computed(() =>
  !userIdTouched.value
    ? ''
    : userId.value <= 0 || userId.value >= 100
      ? 'Número de talonario debe ser entre 1 y 99'
      : '',
)
const dateError = computed(() => {
  if (!dateTouched.value) return ''
  const dateRegex = /^\d{2}\/\d{2}$/
  if (!dateRegex.test(dateInput.value)) return 'Formato inválido (DD/MM)'
  const parts = dateInput.value.split('/')
  const day = parseInt(parts[0] ?? '', 10)
  const month = parseInt(parts[1] ?? '', 10)
  if (day < 1 || day > 31 || month < 1 || month > 12) return 'Fecha inválida'
  return ''
})
const amountError = computed(() =>
  !amountTouched.value ? '' : amount.value <= 0 ? 'Monto debe ser mayor a 0' : '',
)
const quantityError = computed(() =>
  !quantityTouched.value ? '' : quantity.value <= 0 ? 'Cantidad debe ser mayor a 0' : '',
)

const isFormValid = computed(() => {
  if (
    userName.value.trim() === '' ||
    userId.value <= 0 ||
    userId.value >= 100 ||
    amount.value <= 0 ||
    quantity.value <= 0
  )
    return false

  const dateRegex = /^\d{2}\/\d{2}$/
  if (!dateRegex.test(dateInput.value)) return false
  const [dayStr, monthStr] = dateInput.value.split('/')
  const day = parseInt(dayStr ?? '', 10)
  const month = parseInt(monthStr ?? '', 10)
  if (isNaN(day) || isNaN(month) || day < 1 || day > 31 || month < 1 || month > 12) return false

  return true
})

const submitForm = () => {
  // Set full date with current year before submission
  const parts = dateInput.value.split('/')
  if (parts.length === 2) {
    const [dayRaw, monthRaw] = parts
    const day = dayRaw?.padStart(2, '0')
    const month = monthRaw?.padStart(2, '0')
    dateOfPayment.value = `2025-${month}-${day}`

    // Update the hidden input manually
    const dateInputEl = formRef.value?.querySelector<HTMLInputElement>(
      "input[name='dateOfPayment']",
    )
    if (dateInputEl) dateInputEl.value = dateOfPayment.value
  }

  console.log('Submitting form with date:', dateOfPayment.value)
  formRef.value?.submit()
  alert('Datos enviados correctamente!')

  // Reset form
  userName.value = ''
  userId.value = 0
  dateInput.value = ''
  dateOfPayment.value = ''
  amount.value = 0
  quantity.value = 0

  // Reset touched states
  userNameTouched.value = false
  userIdTouched.value = false
  dateTouched.value = false
  amountTouched.value = false
  quantityTouched.value = false
}
</script>

<style scoped>
/* Enhanced styling for mobile web app */
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  margin: 0;
  padding: 0;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

div {
  margin: 0 auto;
  padding: 0.5rem;
  max-width: 320px;
  width: auto;
}

h1 {
  text-align: center;
  margin-bottom: 2rem;
  font-size: 1.8rem;
  color: #333;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.form-container {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.18);
}

form {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  width: 100%;
  margin-bottom: 0; /* Override previous */
}

label {
  margin-bottom: 0.5rem;
  font-weight: 600;
  font-size: 0.9rem;
  color: #555;
}

input {
  padding: 1rem 0.75rem;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
  width: 100%;
  transition: all 0.3s ease;
  background: #f8f9fa;
}

input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  background: white;
}

.error {
  color: #dc3545;
  font-size: 0.8rem;
  margin-top: 0.25rem;
  display: block;
  font-weight: 500;
}

button {
  padding: 0.875rem 2rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  margin-top: 1rem;
  align-self: center;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

button:active:not(:disabled) {
  transform: translateY(0);
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

@media (min-width: 768px) {
  form {
    gap: 2rem;
  }
  h1 {
    font-size: 2.2rem;
  }
  .form-container {
    padding: 2.5rem;
  }
  div {
    max-width: 400px;
  }
}

@media (max-width: 480px) {
  div {
    padding: 0.5rem;
  }
  .form-container {
    padding: 1.5rem;
    border-radius: 8px;
  }
  input {
    padding: 0.875rem 0.625rem;
    font-size: 0.95rem;
  }
  button {
    width: 100%;
    padding: 0.875rem;
  }
  h1 {
    font-size: 1.5rem;
    margin-bottom: 1.5rem;
  }
}

@media (max-width: 375px) {
  div {
    padding: 0.5rem;
    max-width: 280px;
  }
  .form-container {
    padding: 1rem;
    border-radius: 6px;
  }
  input {
    padding: 0.75rem 0.5rem;
    font-size: 0.9rem;
  }
  label {
    font-size: 0.8rem;
  }
  button {
    padding: 0.75rem;
    font-size: 0.95rem;
  }
  h1 {
    font-size: 1.4rem;
    margin-bottom: 1rem;
  }
  form {
    gap: 1rem;
  }
}

@media (max-width: 320px) {
  .form-container {
    padding: 0.75rem;
    margin: 0.5rem;
  }
  input {
    padding: 0.75rem 0.5rem;
    font-size: 0.85rem;
    border-radius: 6px;
  }
  label {
    font-size: 0.8rem;
    margin-bottom: 0.25rem;
  }
  button {
    padding: 0.625rem;
    font-size: 0.9rem;
    border-radius: 6px;
  }
  h1 {
    font-size: 1.2rem;
    margin-bottom: 0.75rem;
  }
  .error {
    font-size: 0.75rem;
  }
  form {
    gap: 1rem;
  }
  div {
    padding: 0.25rem;
  }
}
</style>
