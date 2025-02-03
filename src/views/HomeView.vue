<script setup>
import { ref, computed } from 'vue'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Table, TableHeader, TableRow, TableHead, TableBody, TableCell } from '@/components/ui/table'
import DarkMode from '@/components/DarkMode.vue'


const inputTokens = ref(0)
const outputTokens = ref(0)
const searchQuery = ref('')
const selectedModel = ref('')
const currentPage = ref(1)
const itemsPerPage = 5


const models = ref([
  { id: '1', provider: 'openAI', model: 'gpt-4o', input_per_million_token: 2.50, output_per_million_token: 10.00 },
  { id: '2', provider: 'openAI', model: 'gpt-4o-2024-11-20', input_per_million_token: 2.50, output_per_million_token: 10.00 },
  { id: '3', provider: 'anthropic', model: 'claude 3.5 Sonnet', input_per_million_token: 3.00, output_per_million_token: 15.00 },
  { id: '4', provider: 'anthropic', model: 'claude 3.5 Haiku', input_per_million_token: 1.50, output_per_million_token: 7.00 },
  { id: "5",provider: "Gemini",model: "Gemini 1.5 Pro",input_per_million_token: 2.80,output_per_million_token: 11.00},
{id: "6",provider: "Gemini",model: "Gemini 1.5 Flash",input_per_million_token: 1.20,output_per_million_token: 5.50}

])


const filteredModels = computed(() => {
  return models.value.filter(model =>
    model.provider.toLowerCase().includes(searchQuery.value.toLowerCase()) &&
    (selectedModel.value === '' || model.provider === selectedModel.value)
  )
})

const paginatedModels = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredModels.value.slice(start, start + itemsPerPage)
})

const totalPages = computed(() => Math.ceil(filteredModels.value.length / itemsPerPage))


const calculatedModels = computed(() => {
  return paginatedModels.value.map(model => {
    const inputCost = inputTokens.value * (model.input_per_million_token / 1_000_000)
    const outputCost = outputTokens.value * (model.output_per_million_token / 1_000_000)
    return { ...model, totalCost: inputCost + outputCost }
  })
})


const changePage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
  }
}
</script>

<template>
  <div class="relative min-h-screen bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-gray-100 p-6">

    <div class="absolute top-4 right-4">
      <DarkMode />
    </div>


    <h1 class="text-3xl font-bold text-center mb-6 text-blue-500">AI Pricing</h1>

    <div class="flex flex-col md:flex-row justify-center items-center space-y-4 md:space-y-0 md:space-x-6">
      <span>Input Tokens</span>
      <Input v-model="inputTokens" type="number" placeholder="Input Tokens" class="w-40" />
      <span>Output Tokens</span>
      <Input v-model="outputTokens" type="number" placeholder="Output Tokens" class="w-40" />
      <Input v-model="searchQuery" type="text" placeholder="Search Model" class="w-60" />

    </div>


    <div class="mt-6 overflow-x-auto px-5">
      <Table class="min-w-full border rounded-lg">
        <TableHeader>
          <TableRow>
            <TableHead class="px-4 py-2 text-blue-500">ID</TableHead>
            <TableHead class="px-4 py-2 text-blue-500">Provider</TableHead>
            <TableHead class="px-4 py-2 text-blue-500">Model</TableHead>
            <TableHead class="px-4 py-2 text-blue-500">Input Cost ($/M Tokens)</TableHead>
            <TableHead class="px-4 py-2 text-blue-500">Output Cost ($/M Tokens)</TableHead>
            <TableHead class="px-4 py-2 text-blue-500">Total Cost ($)</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="model in calculatedModels" :key="model.id">
            <TableCell class="px-4 py-2">{{ model.id }}</TableCell>
            <TableCell class="px-4 py-2">{{ model.provider }}</TableCell>
            <TableCell class="px-4 py-2">{{ model.model }}</TableCell>
            <TableCell class="px-4 py-2">${{ model.input_per_million_token.toFixed(2) }}</TableCell>
            <TableCell class="px-4 py-2">${{ model.output_per_million_token.toFixed(2) }}</TableCell>
            <TableCell class="px-4 py-2 font-bold">${{ model.totalCost.toFixed(2) }}</TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>


    <div class="flex justify-center mt-6 space-x-2 text-blue-500">
      <Button variant="outline" @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Previous</Button>
      <span class="px-4 py-2 font-bold">{{ currentPage }} / {{ totalPages }}</span>
      <Button variant="outline" @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">Next</Button>
    </div>
  </div>
</template>
