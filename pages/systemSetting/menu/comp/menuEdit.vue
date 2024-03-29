<template>
  <v-row justify="center">
    <v-col cols="12">
      <v-text-field
        v-model="menuNm"
        label="Menu Name *"
        :error-messages="errors.menuNm"
      />
    </v-col>
    <v-col cols="6">
      <input-select-box-comp
        :items="menuLvItems"
        :selected-value="menuLv"
        :error-messages="errors.menuLv"
        @call-back-selected-value="callBackMenuLvItems"
      />
    </v-col>
    <v-col cols="6">
      <input-select-box-comp
        :items="parentItems"
        :selected-value="prnMenuId"
        :error-messages="errors.prnMenuId"
        @call-back-selected-value="callBackParentItems"
      />
    </v-col>
    <v-col cols="12">
      <v-text-field v-model="url" label="Url" />
    </v-col>
    <v-col cols="1" class="d-flex justify-center pt-5">
      <v-icon :icon="activeIcon" size="40px" />
    </v-col>
    <v-col cols="9">
      <v-text-field v-model="icon" label="Icon" disabled />
    </v-col>
    <v-col cols="2" class="d-flex justify-left mt-2">
      <v-btn
        class="ma-1 v-icon--size-x-small"
        color="#388E3C"
        @click="handleMenuIcon()"
      >
        <v-icon icon="mdi-playlist-edit" />
      </v-btn>
    </v-col>
    <v-col cols="6">
      <v-text-field v-model="ord" label="Ord" />
    </v-col>
    <v-col cols="6">
      <input-select-box-comp :items="useYnItems" :selected-value="useYn" />
    </v-col>
    <menu-icon-pop
      :menu-icon-popup="menuIconPopup"
      :icon="icon"
      @call-back-menu-icon-popup="handleMenuIconPop"
      @call-back-menu-icon-save="handleMenuIconSave"
    />
  </v-row>
</template>
<script lang="ts" setup>
import { onMounted, ref, watch } from 'vue'
import InputSelectBoxComp from '~/components/InputSelectBoxComp.vue'
import { mainStore } from '@/stores/main'
import MenuIconPop from '~/pages/systemSetting/menu/comp/menuIconPop.vue'

interface MenuItem {
  id: string
  lv: string
  label: string
  useYn: string
  upperId?: string
  icon?: string
  url?: string
  ord?: string
}

const props = defineProps<{
  selectedMenuItems: {
    menuItems: MenuItem[]
    parentMenuItems: MenuItem[]
    mode: string | null
  }
}>()

const emits = defineEmits(['callBackSave'])

const main = mainStore()

const prnMenuId = ref<string>('')
const activeIcon = ref<string>('mdi-folder-open')
const errors = ref({})
const menuIconPopup = ref<boolean>(false)

const menuNm = ref<string>('')
const menuLv = ref<string>('')
const icon = ref<string>('')
const useYn = ref<string>('')
const url = ref<string>('')
const ord = ref<string>('')

const menuLvData = ref([
  { label: '상위 메뉴', value: '1' },
  { label: '하위 메뉴', value: '2' }
])
const useYnData = ref([
  { label: '사용', value: 'Y' },
  { label: '미사용', value: 'N' }
])

const menuLvItems = ref({
  data: menuLvData,
  option: {
    label: 'Menu Lv *',
    type: 'select'
  }
})

const parentItems = ref({
  data: [],
  option: {
    label: 'Parent Menu',
    disabled: true
  }
})

const useYnItems = ref({
  data: useYnData,
  option: {
    label: 'Use *'
  }
})

const setData = (value: any): void => {
  const { menuItems, parentMenuItems, mode } = value

  prnMenuId.value = menuItems.upperId
  menuNm.value = menuItems.label
  menuLv.value = menuItems.lv
  icon.value = menuItems.icon
  useYn.value = menuItems.useYn
  url.value = menuItems.url
  ord.value = menuItems.ord

  if (menuItems.icon === '' && menuItems.lv === '1') {
    activeIcon.value = 'mdi-folder-outline'
  } else if (menuItems.icon === '' && menuItems.lv === '2') {
    activeIcon.value = 'mdi-folder-open'
  } else if (menuItems.icon === '') {
    activeIcon.value = 'mdi-folder-open'
  } else {
    activeIcon.value = menuItems.icon
  }

  if (mode === 'add') {
    menuLvItems.value = {
      data: menuLvData,
      option: {
        label: 'Menu Lv',
        type: 'select'
      }
    }
  }

  // prettier-ignore
  const data = parentMenuItems?.map(item => ({ label: item.label, value: item.id })) ?? []

  parentItems.value = {
    data,
    option: {
      label: 'Parent Menu',
      disabled: true
    }
  }
}

const callBackMenuLvItems = (value: string) => {
  const disabled: boolean = value !== '2'

  parentItems.value = {
    ...parentItems.value,
    option: {
      label: 'Parent Menu',
      disabled
    }
  }

  menuLv.value = value
}

const callBackParentItems = (value: string) => {
  prnMenuId.value = value
}

const saveConfirm = () => {
  main.confirmAlertOption = {
    text: '저장 하시겠습니까?',
    fnc: save
  }
  main.isConfirmAlert = true
}

const save = () => {
  if (validation()) {
    const data = {
      menuNm: menuNm.value,
      menuLv: menuLv.value,
      prnMenuId: prnMenuId.value,
      icon: icon.value,
      useYn: useYn.value,
      url: url.value,
      ord: ord.value
    }

    emits('callBackSave', data)
  } else {
    main.alertOption = {
      title: 'Warning',
      text: '입력하신 내용을 확인해주세요.'
    }
    main.isAlert = true
  }
}

// 유효성 처리 값 입력 폼에 오류 내용을 전달한다.
const validation = (): boolean => {
  const error = {}
  let chk = true

  if (menuNm.value === '') {
    error.menuNm = 'Please enter the menu name'
    chk = false
  }

  if (menuLv.value === '') {
    error.menuLv = 'Please select the menu lv'
    chk = false
  }

  if (menuLv.value === '2' && prnMenuId.value === '') {
    error.prnMenuId = 'Please select the parent menu '
    chk = false
  }

  errors.value = error

  return chk
}

const handleMenuIcon = () => {
  menuIconPopup.value = true
}

const handleMenuIconSave = (value: string) => {
  icon.value = value
  activeIcon.value = value
}

const handleMenuIconPop = (value: boolean) => {
  menuIconPopup.value = value
}

watch(
  () => props.selectedMenuItems,
  (newValue) => {
    setData(newValue)
  }
)

watch(
  () => menuLv.value,
  (newValue) => {
    if (newValue !== '2') {
      prnMenuId.value = ''
    }
  }
)

onMounted(() => {
  setData(props.selectedMenuItems)
})

defineExpose({
  saveConfirm
})
</script>

<style scoped></style>
