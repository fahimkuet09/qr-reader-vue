<template>
  <div>
    <QrcodeStream @detect="onDetect" />
    <label for="batchSelect">Select Batch:</label>
    <select id="batchSelect" v-model="selectedBatchId">
      <option value="">Select a Batch</option>
      <option v-for="batch in batches" :value="batch.id">{{ batch.title }}</option>
    </select>
  </div>
</template>

<script setup>
import { QrcodeStream } from 'vue-qrcode-reader'
import axios from 'axios'
import { ref, onMounted } from 'vue'

const selectedBatchId = ref('')
const batches = ref([])
const studentId = ref('')
let lastStudentId = null // Keep track of the last detected student ID

const onDetect = async (detectedCodes) => {
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
        

        // You can also display a success message to the user if needed
        //alert('Student ID and Batch ID sent to the backend successfully!')
      }
    }
  } catch (error) {
    console.error('Error detecting student ID:', error)
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
/* Your scoped CSS styles go here */
</style>
