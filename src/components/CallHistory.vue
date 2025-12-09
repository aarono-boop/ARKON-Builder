<template>
  <div class="min-h-screen bg-gray-900 text-white p-6" role="region" aria-label="Call History">
    <div class="max-w-7xl mx-auto space-y-6">
      
      <!-- Header -->
      <div class="flex items-center justify-between bg-gray-800/60 border border-gray-700 rounded-xl px-4 py-3">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-full bg-blue-600 flex items-center justify-center">
            <i class="pi pi-history text-white text-xl"></i>
          </div>
          <div>
            <h1 class="text-xl font-bold">Call History</h1>
            <p class="text-gray-400 text-xs">View your performance and recent sessions</p>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <Dropdown
            v-model="selectedRange"
            :options="ranges"
            optionLabel="label"
            optionValue="value"
            class="w-40 mic-like-dropdown"
            variant="filled"
            appendTo="body"
            :pt="{
              root: { style: { background: 'var(--p-surface-800)', border: '1px solid rgba(255,255,255,0.12)', borderRadius: '6px', padding: '0.5rem' } },
              panel: { class: 'mic-dropdown-panel', style: { background: 'var(--p-surface-800)', border: '1px solid rgba(255,255,255,0.12)' } },
              overlay: { class: 'mic-dropdown-panel', style: { background: 'var(--p-surface-800)', border: '1px solid rgba(255,255,255,0.12)' } },
              content: { class: 'mic-dropdown-content', style: { background: 'var(--p-surface-800)', padding: '0.25rem' } },
              list: { class: 'mic-dropdown-list', style: { background: 'var(--p-surface-800)' } },
              items: { class: 'mic-dropdown-list', style: { background: 'var(--p-surface-800)' } },
              item: { class: 'mic-dropdown-item hover:bg-white/10', style: { padding: '0.5rem 0.75rem' } }
            }"
          />
        </div>
      </div>

      <!-- Stats Grid -->
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-4">
        <Card v-for="stat in currentStats" :key="stat.label" class="bg-gray-800/40 border border-gray-700">
          <template #content>
            <div class="flex flex-col gap-2">
              <span class="text-gray-400 text-sm">{{ stat.label }}</span>
              <div class="flex items-end justify-between">
                <span class="text-3xl font-bold">{{ stat.value.toLocaleString() }}</span>
                <span :class="['text-sm font-medium', stat.trend >= 0 ? 'text-green-400' : 'text-red-400']">
                  <i :class="['pi', stat.trend >= 0 ? 'pi-arrow-up' : 'pi-arrow-down', 'text-xs']"></i>
                  {{ Math.abs(stat.trend) }}%
                </span>
              </div>
            </div>
          </template>
        </Card>
      </div>

      <!-- Recent Dial Sessions -->
      <div class="bg-gray-800/40 border border-gray-700 rounded-xl p-0 overflow-hidden">
        <div class="p-4 border-b border-gray-700">
          <h2 class="text-lg font-semibold">Recent Dial Sessions</h2>
        </div>
        <DataTable 
          v-model:expandedRows="expandedRows"
          :value="recentSessions" 
          dataKey="id"
          scrollable 
          scrollHeight="500px" 
          :tableStyle="{ tableLayout: 'fixed' }" 
          size="large"
        >
          <Column field="date" header="Date" headerClass="py-4 px-4" bodyClass="py-4 px-4">
            <template #body="{ data }">
              {{ formatDate(data.date) }}
            </template>
          </Column>
          <Column field="duration" header="Duration" headerClass="py-4 px-4" bodyClass="py-4 px-4" />
          <Column field="contacts" header="Contacts" headerClass="py-4 px-4" bodyClass="py-4 px-4" />
          <Column field="calls" header="Calls" headerClass="py-4 px-4" bodyClass="py-4 px-4" />
          <Column field="liveAnswers" header="Live Answers" headerClass="py-4 px-4" bodyClass="py-4 px-4" />
          <Column field="voicemails" header="Voicemails" headerClass="py-4 px-4" bodyClass="py-4 px-4" />
          <Column field="status" header="Status" headerClass="py-4 px-4" bodyClass="py-4 px-4">
            <template #body="{ data }">
              <Badge :value="data.status" :severity="getStatusSeverity(data.status)" />
            </template>
          </Column>
          <Column header="Details" headerClass="py-4 px-4" bodyClass="py-4 px-4" style="width: 100px">
            <template #body="{ data }">
              <Button
                :icon="isRowExpanded(data) ? 'pi pi-chevron-down' : 'pi pi-chevron-right'"
                text
                rounded
                :aria-label="isRowExpanded(data) ? 'Collapse Session' : 'Expand Session'"
                @click="toggleRow(data)"
              />
            </template>
          </Column>
          <template #expansion="{ data }">
            <div class="p-4 bg-gray-900/30 border-t border-gray-700">
              <SessionDetails :session="data" :embedded="true" />
            </div>
          </template>
        </DataTable>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import Dropdown from 'primevue/dropdown'
import Card from 'primevue/card'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import Badge from 'primevue/badge'
import Button from 'primevue/button'
import SessionDetails from './SessionDetails.vue'

const ranges = [
  { label: 'Past Week', value: 'week' },
  { label: 'Past Month', value: 'month' }
]
const selectedRange = ref('week')
const expandedRows = ref({})

const toggleRow = (data: any) => {
  const newExpandedRows = { ...expandedRows.value }
  if (newExpandedRows[data.id]) {
    delete newExpandedRows[data.id]
  } else {
    // Optional: Close other rows if you only want one open at a time
    // newExpandedRows = {} 
    newExpandedRows[data.id] = true
  }
  expandedRows.value = newExpandedRows
}

const isRowExpanded = (data: any) => {
  return !!expandedRows.value[data.id]
}

// Mock Data
const statsData = {
  week: [
    { label: 'Dial Sessions', value: 12, trend: 5 },
    { label: 'Contacts', value: 450, trend: 12 },
    { label: 'Calls', value: 380, trend: 8 },
    { label: 'Live Answers', value: 45, trend: -2 },
    { label: 'Emails', value: 120, trend: 15 },
    { label: 'Voicemails', value: 210, trend: 4 }
  ],
  month: [
    { label: 'Dial Sessions', value: 48, trend: 10 },
    { label: 'Contacts', value: 1800, trend: 20 },
    { label: 'Calls', value: 1520, trend: 15 },
    { label: 'Live Answers', value: 190, trend: 5 },
    { label: 'Emails', value: 540, trend: 25 },
    { label: 'Voicemails', value: 850, trend: 8 }
  ]
}

const currentStats = computed(() => {
  return selectedRange.value === 'week' ? statsData.week : statsData.month
})

const recentSessions = ref([
  { id: '1', date: new Date(Date.now() - 1000 * 60 * 60 * 2), duration: '45m', contacts: 50, calls: 48, liveAnswers: 5, voicemails: 30, status: 'Completed' },
  { id: '2', date: new Date(Date.now() - 1000 * 60 * 60 * 26), duration: '1h 15m', contacts: 80, calls: 75, liveAnswers: 8, voicemails: 45, status: 'Completed' },
  { id: '3', date: new Date(Date.now() - 1000 * 60 * 60 * 50), duration: '30m', contacts: 25, calls: 22, liveAnswers: 2, voicemails: 15, status: 'Completed' },
  { id: '4', date: new Date(Date.now() - 1000 * 60 * 60 * 74), duration: '55m', contacts: 60, calls: 58, liveAnswers: 7, voicemails: 35, status: 'Completed' },
  { id: '5', date: new Date(Date.now() - 1000 * 60 * 60 * 98), duration: '1h 05m', contacts: 70, calls: 68, liveAnswers: 9, voicemails: 40, status: 'Completed' },
  { id: '6', date: new Date(Date.now() - 1000 * 60 * 60 * 122), duration: '40m', contacts: 35, calls: 32, liveAnswers: 4, voicemails: 20, status: 'Completed' },
  { id: '7', date: new Date(Date.now() - 1000 * 60 * 60 * 146), duration: '1h 30m', contacts: 100, calls: 95, liveAnswers: 12, voicemails: 60, status: 'Completed' }
])

const formatDate = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    month: 'short',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric'
  }).format(date)
}

const getStatusSeverity = (status: string) => {
  switch (status) {
    case 'Completed': return 'success'
    case 'In Progress': return 'info'
    case 'Cancelled': return 'danger'
    default: return 'secondary'
  }
}
</script>

<style scoped>
:deep(.p-card-content) {
  padding: 1.25rem;
}
:deep(.p-datatable-row-expansion > td) {
    padding: 0 !important;
    border: none !important;
    background: transparent !important;
}
</style>
