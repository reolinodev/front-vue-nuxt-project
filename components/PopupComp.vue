<template>
  <v-dialog v-model="popupOpen" persistent :width="popupWidth">
    <v-card>
      <v-toolbar flat color="indigo" dark>
        <v-btn icon @click="closePopup">
          <v-icon>mdi-close</v-icon>
        </v-btn>
        <v-toolbar-title>{{ popupTitle }}</v-toolbar-title>
      </v-toolbar>

      <v-card-text>
        <slot />
      </v-card-text>

      <v-divider />

      <v-card-actions class="justify-end ga-1 mt-2 mr-3">
        <v-btn
          color="error"
          variant="outlined"
          prepend-icon="mdi-close-thick"
          width="120"
          @click="closePopup"
        >
          close
        </v-btn>
        <v-btn
          v-if="popupType === 'edit'"
          color="primary"
          variant="outlined"
          prepend-icon="mdi-check"
          width="120"
          @click="saveEvent"
        >
          save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps<{
  popupOpen: boolean
  popupTitle: string
  popupWidth: string
  popupType: string
}>()

const emits = defineEmits(['callBackPopup', 'callBackSave'])

const popupOpen = ref<boolean>(props.popupOpen)
const popupTitle = ref<string>(props.popupTitle)
const popupWidth = ref<string>(props.popupWidth)
const popupType = ref<string>(props.popupType)

const closePopup = () => {
  popupOpen.value = false
  emits('callBackPopup', false)
}

const saveEvent = () => {
  emits('callBackSave')
}

watch(
  () => props.popupOpen,
  (newValue) => {
    popupOpen.value = newValue
  }
)

watch(
  () => props.popupTitle,
  (newValue) => {
    popupTitle.value = newValue
  }
)

watch(
  () => props.popupWidth,
  (newValue) => {
    popupWidth.value = newValue
  }
)

watch(
  () => props.popupType,
  (newValue) => {
    popupType.value = newValue
  }
)
</script>

<style scoped></style>
