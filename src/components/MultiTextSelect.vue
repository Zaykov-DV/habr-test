<template>
  <div
      class="multiselect"
      ref="multiselect"
  >
    <label
        v-if="label"
        for="input"
        class="multiselect__label"
        :class="{ 'is-disabled': disabled }"
    >
      <span v-if="isRequired">*</span>
      {{ label }}
    </label>
    <div class="multiselect__wrapper">
      <ul
          ref="selectedList"
          role="listbox"
          aria-orientation="horizontal"
          class="multiselect__list"
          :class="{'is-error' : error.length}"
      >
        <li
            v-for="option in selectedOptions"
            :key="option.alias"
            role="option"
            :aria-label="option.name"
            aria-selected="true"
            class="multiselect__tag"
        >
          <Tag @on-delete="removeOption" :option/>
        </li>
        <li
            role="option"
            aria-label="input"
            class="multiselect__input-wrapper"
        >
          <input
              ref="input"
              class="multiselect__input"
              type="text"
              role="combobox"
              aria-autocomplete="list"
              :aria-expanded="isDropdownVisible"
              :placeholder="placeholder"
              @input="onInputChange"
              @click="openDropdown"
          >
        </li>
      </ul>
    </div>
    <span class="multiselect__error" v-if="error.length">{{ error }}</span>
    <Transition name="fade">
    <div
        v-show="isDropdownVisible"
        class="multiselect__dropdown-wrapper"
        :style="{top: `${inputHeight}px`}"
    >
      <ul
          id="select-dropdown"
          class="multiselect__dropdown"
          role="listbox"
      >
        <li
            v-for="option in optionsComputed"
            :key="option.alias"
            tabindex="0"
            class="multiselect__option"
            :class="{
            'is-selected': isSelected(option.alias),
          }"
            role="option"
            @click.prevent="toggleOptionSelect(option)"
        >
          <div class="multiselect__option-item">
            <component :is="itemType(option.type)" :option/>
          </div>
        </li>
      </ul>
    </div>
    </Transition>
  </div>
</template>

<script setup>
import {ref, watch, computed, onMounted, onBeforeUnmount} from 'vue'
import CompanyItem from "@/components/CompanyItem.vue";
import UserItem from "@/components/UserItem.vue";
import Tag from "@/components/Tag.vue";

const props = defineProps({
  options: {
    type: [Array, null],
    default: null,
    required: true,
  },
  placeholder: {
    type: String,
    default: '',
  },
  label: {
    type: String,
    default: null,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  isRequired: {
    type: Boolean,
    default: false,
  },
  error: {
    type: String,
    default: null,
  },
  maxSelectedOptions: {
    type: [Number, null],
    default: null
  }
})

const multiselect = ref(null)
const inputHeight = ref(null)
const selectedOptions = ref([])
const isDropdownVisible = ref(false)

const emit = defineEmits(['update:modelValue', 'onInputChange'])

const onInputChange = (event) => {
 emit('onInputChange', event.target.value)
}

watch(
    () => props.options,
    () => {
      isDropdownVisible.value = !!props.options?.length;
    },
)

const observeHeight = () => {
  const resizeObserver = new ResizeObserver(() => {
    inputHeight.value = multiselect.value?.clientHeight
  })

  resizeObserver.observe(multiselect?.value);
}


const optionsComputed = computed(() => {
  return props.options?.filter(option => !!option)
})

const toggleOptionSelect = (option) => {
  if (!props.maxSelectedOptions || (selectedOptions.value.length + 1) <= props.maxSelectedOptions) {
    if (selectedOptions.value.includes(option)) {
      selectedOptions.value = selectedOptions.value.filter(
          item => item.alias !== option.alias,
      )
    } else {
      selectedOptions.value.push(option)
    }

    emit('update:modelValue', selectedOptions.value)
  }
}

const openDropdown = (event) => {
  if (event.target.value) {
    isDropdownVisible.value = true
  }
}

const removeOption = (alias) => {
  selectedOptions.value = selectedOptions.value.filter(
      item => item.alias !== alias
  )
}

const itemType = (type) => {
  if (type === 'company') return CompanyItem
  return UserItem
}

const isSelected = (id) => {
  return selectedOptions.value.find((opt) => {
    return opt.alias === id
  })
}


const closeDropdown = (element) => {
  if (!multiselect.value?.contains(element.target)) {
    isDropdownVisible.value = false
  }
}


onMounted(() => {
  window.addEventListener('click', closeDropdown)
  observeHeight()
})

onBeforeUnmount(() => {
  window.addEventListener('click', closeDropdown)
})
</script>

<style lang="scss" scoped>
@import "../assets/mixins/utility";

.multiselect {
  position: relative;
  display: flex;
  flex-direction: column;

  &__wrapper {
    display: flex;
    position: relative;
  }

  &__label {
    margin-bottom: 4px;
  }

  &__list {
    margin: 0;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-radius: 4px;
    width: 100%;
    list-style-type: none;
    cursor: text;
    overflow: hidden;
    padding: 4px 8px;
    color: #334155;
    background: #ffffff;
    border: 1px solid #cbd5e1;
    box-shadow: 0 0 0 1px rgb(56, 56, 61);

    &.is-error {
      box-shadow: 0 0 0 1px #ff3b3b;
    }
  }

  &__input-wrapper {
    padding: 0 8px;
  }

  &__input {
    width: 100%;
    padding: 0;
    margin: 0;
    border: none;
    outline: none;
  }

  &__dropdown-wrapper {
    position: absolute;
    width: 100%;
    margin-top: 4px;
    background: #ffffff;
    border: 1px solid #cbd5e1;
    box-shadow: 0px 0px #0000, 0px 0px #0000, 0px 1px 2px 0px rgba(18, 18, 23, 0.05);
    max-height: 224px;
    overflow-y: scroll;
    @include slimScrollbar;
  }

  &__option {
    background-color: #fff;
    transition: background-color .3s ease-in-out;

    &:hover {
      background-color: #d7d7d7;
    }

    &:focus-visible {
      background-color: #d7d7d7;
      outline: none;
    }

    &.is-selected {
      background-color: #a0ff9a;
    }
  }

  &__tag {
    margin-left: -2px;
  }

  &__error {
    color: red;
    margin-top: 2px;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
