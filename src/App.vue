<template>
  <div class="min-h-screen bg-gray-400 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="absolute inset-0 bg-gradient-to-r from-green-200 to-light-blue-300 shadow-xl transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white bg-opacity-80 shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-md mx-auto">
          <div>
            <h1 class="text-2xl font-normal underline  text-center"><i class="bi bi-file-earmark-text"></i> AI Document <span class="text-blue-500">Q&A</span></h1>
          </div>
          <div class="divide-y divide-gray-200">
            <div class="py-8 text-base leading-6 space-y-4 text-gray-700 sm:text-lg sm:leading-7">
              <div class="flex items-center space-x-4">
                <label class="block text-sm font-medium text-gray-700">
                  Upload File (PDF or TXT):
                </label>
                <input type="file" @change="handleFileUpload" accept=".pdf,.txt" class="mt-1 block w-full text-sm text-gray-500
                  file:mr-4 file:py-2 file:px-4
                  file:rounded-full file:border-0
                  file:text-sm file:font-semibold
                  file:bg-cyan-50 file:text-cyan-700
                  hover:file:bg-cyan-100
                "/>
              </div>
              <div v-if="uploadedFile" class="text-sm text-gray-500">
                Uploaded: {{ uploadedFile }}
              </div>
              <div class="mt-6">
                <label for="question" class="block text-sm font-medium text-gray-700">Your Question:</label>
                <input type="text" id="question" v-model="question" @keyup.enter="askQuestion" class="mt-1 block w-full p-2 rounded-md border-gray-300 shadow-lg focus:border-cyan-300 focus:ring focus:ring-cyan-200 focus:ring-opacity-50" placeholder="Ask a question about the document...">
              </div>
              <div class="mt-6">
                <button @click="askQuestion" 
                class="inline-flex items-center 
                px-4 py-2 border border-green-500 text-sm font-medium rounded-md shadow-sm text-white bg-green-600 hover:bg-cyan-700 
                focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-cyan-500"
                :disabled="isLoading">
                <svg 
                  v-if="isLoading" 
                  class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" 
                  xmlns="http://www.w3.org/2000/svg" 
                  fill="none" 
                  viewBox="0 0 24 24"
                >
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                {{ isLoading ? 'Processing...' : 'Ask Question' }}
                </button>
              </div>
            </div>
            <div class="pt-6 text-base leading-6 font-bold sm:text-lg sm:leading-7">
              <p v-if="displayedAnswer" class="text-gray-800 font-light"><i class="bi bi-robot text-gray-600 text-xl mt-1"></i> Answer: {{ displayedAnswer }}</p>
              <p v-if="error" class="text-red-500">{{ error }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

import { onUnmounted } from 'vue';
import axios from 'axios'

export default {
  setup() {
    const uploadedFile = ref(null)
    const question = ref('')
    const answer = ref('')
    const displayedAnswer=ref('')
    const isTyping = ref(false)
   const isLoading=ref(false)
    const error = ref('')

    const handleFileUpload = async (event) => {
      const file = event.target.files[0]
      if (!file) return

      const formData = new FormData()
      formData.append('file', file)

      try {
        const response = await axios.post('http://localhost:5000/upload', formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        })
        uploadedFile.value = response.data.filename
        error.value = ''
      } catch (err) {
        error.value = 'Error uploading file. Please try again.'
        console.error(err)
      }
    }

    const askQuestion = async () => {
      if (!uploadedFile.value || !question.value) {
        error.value = 'Please upload a file and enter a question.'
        return
      }
      isLoading.value = true
     error.value = ''
      try {
        const response = await axios.post('http://localhost:5000/ask', {
          filename: uploadedFile.value,
          question: question.value
        })
        answer.value = response.data.answer
        error.value = ''
        startTypingAnimation()
      } catch (err) {
        error.value = 'Error getting answer. Please try again.'
        console.error(err)
      }finally{
        isLoading.value = false
      }
    }
    const startTypingAnimation = () => {
      isTyping.value = true
      displayedAnswer.value = ''
      let i = 0
      const typingInterval = setInterval(() => {
        if (i < answer.value.length) {
          displayedAnswer.value += answer.value.charAt(i)
          i++
        } else {
          clearInterval(typingInterval)
          isTyping.value = false
        }
      }, 30) // Adjust the interval (in milliseconds) to control typing speed
    }

    // Stop the animation if the component is unmounted
    onUnmounted(() => {
      isTyping.value = false
    })

    return {
      uploadedFile,
      question,
      displayedAnswer,
      error,
      handleFileUpload,
      askQuestion,
      isTyping,
      isLoading
    }
  }
}
</script>
<style scoped>
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css");




</style>