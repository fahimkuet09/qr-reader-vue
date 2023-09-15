<template>
  <div class="app-container">
    <!-- Conditional rendering for login view -->
    <div v-if="!isLoggedIn" class="login-form">
      <label for="username">Username:</label>
      <input type="text" id="username" placeholder="user name" v-model="username" />
      <label for="password">Password:</label>
      <input type="password" id="password" placeholder="password" v-model="password" />
      <button @click="login">Login</button>
    </div>

    <!-- Conditional rendering for main content view -->
    <div v-else>
      <label for="batchSelect">Select Batch:</label>
      <select id="batchSelect" v-model="selectedBatchId">
        <option value="">Select a Batch</option>
        <option v-for="batch in batches" :value="batch.id">{{ batch.title }}</option>
      </select>

      <div class="qrcode-section">
        <QrcodeStream @detect="onDetect" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { QrcodeStream } from 'vue-qrcode-reader'
import axios from 'axios'
import { ref, onMounted } from 'vue'

const isLoggedIn = ref(false) // Track login status
const username = ref('')
const password = ref('')
const selectedBatchId = ref('')
const batches = ref([])
const studentId = ref('')
let lastStudentId = null // Keep track of the last detected student ID

const onDetect = async (detectedCodes) => {
  // Check if the user is logged in before processing QR codes
  if (!isLoggedIn.value) {
    alert('Please log in first')
    return
  }

  try {
    const student_id = detectedCodes[0].rawValue
    console.log(student_id)

    // Check if the detected student ID is different from the last one
    if (student_id !== lastStudentId) {
      lastStudentId = student_id // Update the last detected student ID

      // Store the detected student ID
      studentId.value = student_id

      // Check if both batch and student IDs are available
      if (selectedBatchId.value && studentId.value) {
        // Replace 'YOUR_API_ENDPOINT' with your actual API endpoint URL
        const apiUrl = 'http://127.0.0.1:8000/api/store-attendance'

        // Make an HTTP POST request to the backend API with selectedBatchId and studentId
        const response = await axios.post(apiUrl, {
          batch_id: selectedBatchId.value,
          student_id: studentId.value
        })

        // Handle the response from the API as needed
        console.log('API Response:', response.data)
        alert(response.data.message)
      } else {
        alert('Please select a batch first')
      }
    }
  } catch (error) {
    console.error('Error detecting student ID:', error)
  }
}

const login = async () => {
  if (username.value === 'admin' && password.value === 'admin') {
    isLoggedIn.value = true
  } else {
    alert('Invalid username or password')
  }
}

onMounted(async () => {
  try {
    // Replace 'YOUR_BATCH_API_ENDPOINT' with the actual API endpoint to fetch batches
    const batchApiUrl = 'http://127.0.0.1:8000/api/get-batches'

    // Make an HTTP GET request to fetch batches
    const batchResponse = await axios.get(batchApiUrl)

    // Set the batches data with the response data
    batches.value = batchResponse.data.data
  } catch (error) {
    console.error('Error fetching batches from the backend:', error)
  }
})
</script>

<style scoped>
.app-container {
  background-color: whitesmoke;
  padding: 20px;
}
.qrcode-section {
  margin-top: 20px;
}
.login-form {
  background-color: rgb(102, 103, 161);
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
.login-form label,
.login-form input,
.login-form button {
  display: block; /* Make elements display as block (one below the other) */
  margin-bottom: 10px; /* Add spacing between elements */
}

</style>
