<template>
  <MultiTextSelect
      v-model="model"
      :options="optionsList"
      :error="errorMessage"
      class="app__control"
      label="Пользователь или компания"
      placeholder="Не выбрано"
      is-required
      @on-input-change="fetchPricingFactors"
  />
</template>

<script setup>
import {ref} from 'vue'
import MultiTextSelect from "@/components/MultiTextSelect.vue";
import {debounce} from "@/composables/debounce.js";

const model = defineModel({
  type: [String, Object, null],
  default: null,
})

const optionsList = ref([])
const errorMessage = ref('')

const fetchPricingFactors = debounce(async (event) => {
  if (!event) {
    optionsList.value = []
    errorMessage.value = ''
  }
  if (event.length > 3) {
    await fetch(`https://habr.com/kek/v2/publication/suggest-mention?q=${event}`)
        .then(res => {
          if (res.ok) {
            errorMessage.value = ''
            res.json().then(res => (optionsList.value = res.data))
          } else {
            errorMessage.value = `Unsupported query`
          }
        })
        .catch(e => {
          errorMessage.value = e.message
        })
  }
}, 500)

</script>

<style lang="scss" scoped>
.app {
  &__control {
    width: 400px;
  }
}
</style>
