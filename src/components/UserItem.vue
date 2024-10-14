<template>
  <div class="option-item">
    <img v-if="option.avatar" class="option-item__avatar" :src="option.avatar" />
    <IconUserDefault v-else class="option-item__avatar" />
    <div>
      <h4 class="option-item__name">{{ option.name ?? option.alias }}</h4>
      <span class="option-item__alias"> @{{ option.alias }}</span>
    </div>
  </div>
</template>

<script setup>
import IconUserDefault from "@/components/icons/IconUserDefault.vue";

defineProps({
  option: {
    type: Object,
    default: () => {}
  }
})
</script>