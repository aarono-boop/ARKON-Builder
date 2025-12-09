<template>
  <div class="space-y-6">
    <!-- Header (Only shown if not embedded) -->
    <div v-if="!embedded" class="flex items-center gap-4">
      <Button icon="pi pi-arrow-left" label="Back" text @click="$emit('back')" />
      <div>
        <h2 class="text-xl font-bold">Session Details</h2>
        <p class="text-gray-400 text-sm">{{ formatDate(session.date) }} • {{ session.duration }}</p>
      </div>
    </div>

    <!-- Session Stats Summary (Only shown if not embedded, or maybe we want it? User said "details open up... below the main row". The main row already has stats. So maybe we don't need stats again?) -->
    <!-- The user prompt says: "show the following... Each Call - Details - Recording Playback...". It doesn't explicitly say to hide the stats, but they are redundant if right below the row. -->
    <!-- However, the row only shows summary numbers. The cards show the same numbers. I'll hide them if embedded to save space and avoid redundancy. -->
    <div v-if="!embedded" class="grid grid-cols-2 md:grid-cols-5 gap-4">
      <Card class="bg-gray-800/40 border border-gray-700">
        <template #content>
          <div class="text-center">
            <div class="text-2xl font-bold">{{ session.calls }}</div>
            <div class="text-xs text-gray-400">Calls</div>
          </div>
        </template>
      </Card>
      <Card class="bg-gray-800/40 border border-gray-700">
        <template #content>
          <div class="text-center">
            <div class="text-2xl font-bold">{{ session.contacts }}</div>
            <div class="text-xs text-gray-400">Contacts</div>
          </div>
        </template>
      </Card>
      <Card class="bg-gray-800/40 border border-gray-700">
        <template #content>
          <div class="text-center">
            <div class="text-2xl font-bold">{{ session.liveAnswers }}</div>
            <div class="text-xs text-gray-400">Live Answers</div>
          </div>
        </template>
      </Card>
      <Card class="bg-gray-800/40 border border-gray-700">
        <template #content>
          <div class="text-center">
            <div class="text-2xl font-bold">{{ session.voicemails }}</div>
            <div class="text-xs text-gray-400">Voicemails</div>
          </div>
        </template>
      </Card>
      <Card class="bg-gray-800/40 border border-gray-700">
        <template #content>
          <div class="text-center">
            <div class="text-2xl font-bold text-green-400">{{ session.status }}</div>
            <div class="text-xs text-gray-400">Status</div>
          </div>
        </template>
      </Card>
    </div>

    <!-- Calls List -->
    <div class="space-y-4">
      <h3 v-if="!embedded" class="text-lg font-semibold">Calls in this Session</h3>
      
      <Accordion v-model:value="activeCallIds" multiple>
        <AccordionPanel v-for="call in calls" :key="call.id" :value="call.id">
          <AccordionHeader>
            <template #toggleicon>
                <i :class="['pi', activeCallIds.includes(call.id) ? 'pi-chevron-down' : 'pi-chevron-right']"></i>
            </template>
            <div class="flex items-center justify-between w-full pr-4">
              <div class="flex items-center gap-4">
                <div class="w-10 h-10 rounded-full bg-gray-700 flex items-center justify-center">
                  <i class="pi pi-user text-white"></i>
                </div>
                <div>
                  <div class="font-bold">{{ call.contactName }}</div>
                  <div class="text-sm text-gray-400">{{ call.phoneNumber }}</div>
                </div>
              </div>
              <div class="flex items-center gap-6">
                <div class="text-right">
                  <div class="text-sm font-medium">{{ call.duration }}</div>
                  <div class="text-xs text-gray-400">{{ formatTime(call.dateTime) }}</div>
                </div>
                <Badge :value="call.status" :severity="getCallStatusSeverity(call.status)" />
              </div>
            </div>
          </AccordionHeader>
          <AccordionContent>
            <Tabs value="details">
                <TabList>
                    <Tab value="details">Details</Tab>
                    <Tab value="recording">Recording</Tab>
                    <Tab value="transcript">Transcript</Tab>
                    <Tab value="notes">User Notes</Tab>
                    <Tab value="ai-notes">AI Notes</Tab>
                    <Tab value="coach">Coach's Recap</Tab>
                </TabList>
                <TabPanels>
                    <TabPanel value="details">
                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <span class="text-gray-400 block text-sm">Contact Name</span>
                                <span class="font-medium">{{ call.contactName }}</span>
                            </div>
                            <div>
                                <span class="text-gray-400 block text-sm">Phone Number</span>
                                <span class="font-medium">{{ call.phoneNumber }}</span>
                            </div>
                            <div>
                                <span class="text-gray-400 block text-sm">Date & Time</span>
                                <span class="font-medium">{{ formatDateTime(call.dateTime) }}</span>
                            </div>
                            <div>
                                <span class="text-gray-400 block text-sm">Duration</span>
                                <span class="font-medium">{{ call.duration }}</span>
                            </div>
                            <div>
                                <span class="text-gray-400 block text-sm">Outcome</span>
                                <span class="font-medium">{{ call.status }}</span>
                            </div>
                        </div>
                    </TabPanel>
                    <TabPanel value="recording">
                        <div class="flex flex-col items-center justify-center p-4 bg-gray-900/50 rounded-lg">
                            <div v-if="call.recordingUrl" class="w-full flex flex-col items-center">
                                <audio controls class="w-full max-w-md">
                                    <source :src="call.recordingUrl" type="audio/mpeg">
                                    Your browser does not support the audio element.
                                </audio>
                                <p class="mt-2 text-sm text-gray-400">Recording duration: {{ call.duration }}</p>
                            </div>
                            <div v-else class="text-gray-400 italic">
                                No recording available for this call.
                            </div>
                        </div>
                    </TabPanel>
                    <TabPanel value="transcript">
                        <div class="bg-gray-900/50 p-4 rounded-lg max-h-60 overflow-y-auto whitespace-pre-wrap text-sm leading-relaxed">
                            {{ call.transcript || 'No transcript available.' }}
                        </div>
                    </TabPanel>
                    <TabPanel value="notes">
                        <div class="bg-gray-900/50 p-4 rounded-lg min-h-[100px]">
                            <p class="text-sm">{{ call.userNotes || 'No user notes for this call.' }}</p>
                        </div>
                    </TabPanel>
                    <TabPanel value="ai-notes">
                        <div class="bg-gray-900/50 p-4 rounded-lg">
                            <div class="prose prose-invert prose-sm max-w-none" v-html="call.aiNotes || 'No AI notes generated.'"></div>
                        </div>
                    </TabPanel>
                    <TabPanel value="coach">
                        <div class="bg-blue-900/20 border border-blue-800 p-4 rounded-lg">
                            <div class="flex items-start gap-3">
                                <i class="pi pi-star-fill text-yellow-500 mt-1"></i>
                                <div>
                                    <h4 class="font-bold text-blue-300 mb-2">Coach's Recap</h4>
                                    <p class="text-sm text-gray-300 leading-relaxed">{{ call.coachRecap || 'No coach recap available.' }}</p>
                                </div>
                            </div>
                        </div>
                    </TabPanel>
                </TabPanels>
            </Tabs>
          </AccordionContent>
        </AccordionPanel>
      </Accordion>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import Button from 'primevue/button'
import Card from 'primevue/card'
import Accordion from 'primevue/accordion'
import AccordionPanel from 'primevue/accordionpanel'
import AccordionHeader from 'primevue/accordionheader'
import AccordionContent from 'primevue/accordioncontent'
import Badge from 'primevue/badge'
import Tabs from 'primevue/tabs'
import TabList from 'primevue/tablist'
import Tab from 'primevue/tab'
import TabPanels from 'primevue/tabpanels'
import TabPanel from 'primevue/tabpanel'

const props = withDefaults(defineProps<{
  session: any
  embedded?: boolean
}>(), {
  embedded: false
})

defineEmits(['back'])

const formatDate = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  }).format(new Date(date))
}

const formatTime = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    hour: 'numeric',
    minute: 'numeric'
  }).format(new Date(date))
}

const formatDateTime = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric',
    hour: 'numeric',
    minute: 'numeric'
  }).format(new Date(date))
}

const getCallStatusSeverity = (status: string) => {
  switch (status) {
    case 'Completed': return 'success'
    case 'No Answer': return 'warn'
    case 'Voicemail': return 'info'
    default: return 'secondary'
  }
}

const activeCallIds = ref(['1'])

// Mock Calls Data
const calls = computed(() => {
    // Generate some mock calls based on the session date
    const sessionDate = new Date(props.session.date)
    return [
        {
            id: '1',
            contactName: 'Sam Sample',
            phoneNumber: '(312) 586-9748',
            dateTime: new Date(sessionDate.getTime() + 1000 * 60 * 5),
            duration: '5m 23s',
            status: 'Completed',
            recordingUrl: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3',
            transcript: "Agent: Hello, this is [Name] from [Company].\nSam: Hi, what is this about?\nAgent: I'm calling to follow up on your interest in our new product.\nSam: Oh, right. I did download the brochure.\nAgent: Great! Did you have any specific questions?\nSam: Actually, yes. How does the pricing work for small teams?\nAgent: We have a special tier for teams under 10 users. It starts at $29/user.\nSam: That sounds reasonable. Can you send me more info?\nAgent: Absolutely. I'll email you the details right now.\nSam: Thanks!",
            userNotes: 'Interested in the small team plan. Sent pricing info.',
            aiNotes: '<ul><li><strong>Sentiment:</strong> Positive</li><li><strong>Key Topics:</strong> Pricing, Small Team Plan</li><li><strong>Action Items:</strong> Send pricing email</li></ul>',
            coachRecap: 'Great job identifying the customer\'s need quickly. You pivoted to the small team plan immediately when he mentioned pricing. Next time, try to lock in a follow-up meeting time before hanging up.'
        },
        {
            id: '2',
            contactName: 'George Sample',
            phoneNumber: '(202) 744-9556',
            dateTime: new Date(sessionDate.getTime() + 1000 * 60 * 15),
            duration: '1m 12s',
            status: 'Voicemail',
            recordingUrl: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3',
            transcript: "[Voicemail Beep]\nAgent: Hi George, this is [Name] with [Company]. I was hoping to catch you to discuss how we can help streamline your workflow. I'll try you again later, or feel free to call me back at [Number]. Thanks!",
            userNotes: 'Left voicemail.',
            aiNotes: '<ul><li><strong>Outcome:</strong> Voicemail</li><li><strong>Next Step:</strong> Follow up in 2 days</li></ul>',
            coachRecap: 'Good voicemail script. You were clear and concise. Make sure to state your value proposition a bit earlier in the message next time.'
        },
        {
            id: '3',
            contactName: 'Jennifer Martinez',
            phoneNumber: '(555) 456-7890',
            dateTime: new Date(sessionDate.getTime() + 1000 * 60 * 25),
            duration: '0m 45s',
            status: 'No Answer',
            recordingUrl: '',
            transcript: "[No Answer]",
            userNotes: '',
            aiNotes: '<ul><li><strong>Outcome:</strong> No Answer</li></ul>',
            coachRecap: 'Keep dialing! Persistence is key.'
        }
    ]
})
</script>

<style scoped>
:deep(.p-accordionpanel) {
    border: none !important;
    box-shadow: none !important;
}
:deep(.p-accordionheader) {
    border: none !important;
    background: transparent !important;
    box-shadow: none !important;
    padding: 1rem !important;
}
:deep(.p-accordionheader:hover) {
    background: rgba(255, 255, 255, 0.05) !important;
}
:deep(.p-accordion-header-link) {
    background: transparent !important;
    border: none !important;
    padding: 0 !important;
}
:deep(.p-accordion-content) {
    background: transparent !important;
    border: none !important;
    padding: 0 1rem 1rem 1rem !important;
}
:deep(.p-tabs) {
    background: transparent !important;
    border: none !important;
}
:deep(.p-tablist) {
    background: transparent !important;
    border: none !important;
}
:deep(.p-tablist-tab-list) {
    background: transparent !important;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1) !important;
    border-top: none !important;
    border-left: none !important;
    border-right: none !important;
}
:deep(.p-tab) {
    background: transparent !important;
    border: none !important;
    border-bottom: 2px solid transparent !important;
    margin-bottom: -1px !important;
}
:deep(.p-tab-active) {
    background: transparent !important;
    border-color: var(--p-primary-color) !important;
}
:deep(.p-tabpanels) {
    background: transparent !important;
    border: none !important;
    padding: 1.5rem 0 0 0 !important;
}
:deep(.p-tabpanel) {
    background: transparent !important;
    border: none !important;
}
</style>
