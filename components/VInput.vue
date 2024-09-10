<template>
  <div>
    <span class=" " dir="rtl">{{ label }} <span v-if="required" class="text-red-500">*</span></span>
    <div :class="{'mt-2':label}" class="relative rounded-lg   ">
      <div :class="{'!border-red-500':error}" class="rounded-lg flex items-center border border-transparent  ">
        <input
            v-model="modelValue"
            :class="[customCss]"
            :disabled="disabled"
            :inputmode="inputMode"
            :maxlength="maxLength"
            :name="vname"
            :placeholder="placeHolder"
            :type="dataType"
            class="w-full rounded-lg border placeholder:text-right  leading-10 ring-0 text-[14px] pr-2 placeholder:text-[14px] outline-0 px-2"
            dir="auto"
            v-on="handlers"
        />
        <DynamicIcon v-if="iconName" :icon="iconName" class="ml-2"></DynamicIcon>
      </div>
    </div>
    <ErrorMessage v-slot="{ message }" :name="vname" as="span" class="text-sm !text-red-500">
      {{ message }}
    </ErrorMessage>
  </div>
</template>


<script lang="ts" setup>
import DynamicIcon from "../components/utilities/DynamicIcon.vue";
import {ErrorMessage, useField} from "vee-validate";
import {toRef} from 'vue';
import {modes} from "../../models/interactionModes";

let props = defineProps({
  dataType: {
    type: String,
    default: 'text'
  },
  required: {
    type: Boolean,
    default: true,

  },
  maxLength: {
    type: String
  },
  error: {
    type: Boolean,
    default: false
  },
  vname: {
    type: String,
    required: true
  },
  errorMessage: {
    type: String,

  },
  placeHolder: {
    type: String,
  },
  iconName: {
    type: String,
  },
  label: {
    type: String
  },
  customCss: {
    type: String
  },
  validationMessage: {
    type: String
  },
  mode: {
    type: String,
    default: 'lazy',
  },
  disabled: {
    type: Boolean,
    default: false
  },
  inputMode: {
    type: String,
    default: 'text'
  },
  hasDefault: {
    type: Boolean,
    default: false
  }
})

const modelValue = defineModel()
const useFieldOptions = (toRef(props, 'vname'))
const useFieldOptions2 = (toRef(props, 'vname'), modelValue.value)

const {value, meta, handleChange, handleBlur} = useField(props.hasDefault ? useFieldOptions2 : useFieldOptions);

watch(() => modelValue.value, (newVal) => {
  if (props.dataType === 'tel' || props.inputMode === 'numeric') {
    value.value = useUtils().convertNumbers2English(newVal);
  } else {
    value.value = newVal;
  }
}, {immediate: true});

const handlers = computed(() => {
  const on = {
    blur: handleBlur,
    // default input event to sync the value
    // the `false` here prevents validation
    input: [(e) => handleChange(e, false)],
  };
  // Get list of validation events based on the current mode
  const triggers = modes[props.mode]({

    meta,
  });
  // add them to the "on" handlers object
  triggers.forEach((t) => {
    if (Array.isArray(on[t])) {
      on[t].push(handleChange);
    } else {
      on[t] = handleChange;
    }
  });

  return on;
});

</script>

<style>
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}
</style>
