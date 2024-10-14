<template>
  <div class="tag">
            <span class="tag__text">
              @{{ option.alias }}
            </span>
    <button
        class="tag__btn-remove"
        @click="onBtnDelete(option.alias)"
    >
      x
    </button>
  </div>
</template>

<script setup>
defineProps({
  option: {
    type: Object,
    default: () => {}
  }
})

const emit = defineEmits(['on-delete'])

const onBtnDelete = (alias) => {
  emit('on-delete', alias)
}
</script>

<style scoped lang="scss">
.tag {
  padding: 4px;
  background-color: grey;
  color: white;
  margin: 2px;
  border-radius: 4px;

  &__btn-remove {
    background-color: transparent;
    outline: none;
    border: none;
    color: white;
    cursor: pointer;
    font-size: 12px;
  }
}
</style>
