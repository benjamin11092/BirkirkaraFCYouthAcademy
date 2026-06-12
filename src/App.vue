<script setup>
import { ref } from 'vue'
import { supabase } from './supabase'

const form = ref({
  kid_name: '',
  kid_surname: '',
  kid_dob: '',
  contact_number: '',
  guardian_name: '',
  guardian_surname: '',
  email: '',
  address: ''
})

const loading = ref(false)
const message = ref('')

async function submitForm() {
  loading.value = true
  message.value = ''

  const { error } = await supabase
    .from('registrations')
    .insert([form.value])

  if (error) {
    message.value = 'Error: ' + error.message
  } else {
  const { error: emailError } = await supabase.functions.invoke(
    'send-registration-email',
    {
      body: {
        email: form.value.email,
        guardian_name: form.value.guardian_name,
        kid_name: form.value.kid_name
      }
    }
  )
   if (emailError) {
    message.value = 'Registration saved, but email failed: ' + emailError.message
  } else {
    message.value = 'Registration submitted successfully. Confirmation email sent.'
  }

    form.value = {
      kid_name: '',
      kid_surname: '',
      kid_dob: '',
      contact_number: '',
      guardian_name: '',
      guardian_surname: '',
      email: '',
      address: ''
    }
  }

  loading.value = false
}
</script>

<template>
  <main class="container">
    <h1>Kid Registration</h1>

    <form @submit.prevent="submitForm">
      <input v-model="form.kid_name" placeholder="Name of Kid" required />
      <input v-model="form.kid_surname" placeholder="Surname of Kid" required />
      <input v-model="form.kid_dob" type="date" required />
      <input v-model="form.contact_number" placeholder="Contact Number" required />
      <input v-model="form.guardian_name" placeholder="Guardian Name" required />
      <input v-model="form.guardian_surname" placeholder="Guardian Surname" required />
      <input v-model="form.email" type="email" placeholder="Email" required />
      <textarea v-model="form.address" placeholder="Address" required></textarea>

      <button :disabled="loading">
        {{ loading ? 'Submitting...' : 'Submit Registration' }}
      </button>
    </form>

    <p>{{ message }}</p>
  </main>
</template>

<style>
.container {
  max-width: 500px;
  margin: 40px auto;
  font-family: Arial, sans-serif;
}

form {
  display: grid;
  gap: 12px;
}

input,
textarea,
button {
  padding: 10px;
  font-size: 16px;
}
</style>