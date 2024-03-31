<template>
  <div class="custom-file" :class="{'disabled': disabled}">
    <div class="custom-file__inner">
      <input
        type="file" 
        class="custom-file__input"
        ref="fileInputRef"
        :id="inputID"
        :accept="accept"
        :disabled="disabled"
        @click="handleFileChange"
        @change="handleFileChange"
      >
      <label class="custom-file__button" :for="inputID">
        {{ text }}
      </label>
      <div class="custom-file__info">
        <span v-if="uploading" class="custom-file__loading">
          <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M16 8C16 9.58225 15.5308 11.129 14.6518 12.4446C13.7727 13.7602 12.5233 14.7855 11.0615 15.391C9.59966 15.9965 7.99113 16.155 6.43928 15.8463C4.88743 15.5376 3.46197 14.7757 2.34315 13.6569C1.22433 12.538 0.4624 11.1126 0.153718 9.56072C-0.154964 8.00887 0.00346269 6.40034 0.608964 4.93853C1.21446 3.47672 2.23985 2.22729 3.55544 1.34824C4.87103 0.469192 6.41775 -1.88681e-08 8 0V1.44C6.70255 1.44 5.43425 1.82474 4.35546 2.54556C3.27667 3.26638 2.43586 4.29091 1.93935 5.4896C1.44284 6.68828 1.31293 8.00728 1.56605 9.27979C1.81917 10.5523 2.44395 11.7212 3.36138 12.6386C4.27881 13.5561 5.44769 14.1808 6.72021 14.434C7.99272 14.6871 9.31172 14.5572 10.5104 14.0606C11.7091 13.5641 12.7336 12.7233 13.4544 11.6445C14.1753 10.5658 14.56 9.29745 14.56 8H16Z" fill="#00A8A5"/>
          </svg>
        </span>
        <span>{{ file ? file.name : 'Файл не выбран' }}</span>
      </div>
    </div>
    <span class="custom-file__error" v-if="computedErrMessage.length">{{ computedErrMessage }}</span>
  </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits, ref, Ref, computed } from 'vue';
const emit = defineEmits(['file-selected', 'file-cancel-download', 'file-removed']);

const props = defineProps({
  inputID: {
    type: String,
    required: true,
  },
  text: {
    type: String,
    default: 'Выбрать файл',
  },
  accept: {
    type: String, 
    default: null,
  },
  maxSize: {
    type: Number,
    default: 0,
  },
  hasError: {
    type: String,
    default: ''
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  uploading: {
    type: Boolean,
    default: false,
  },
  uploaded: {
    type: Boolean,
    default: false,
  }
})

const fileInputRef = ref<HTMLInputElement | null>(null);
const file: Ref<File | null> = ref(null);
const errorText: Ref<string> = ref('');

const computedErrMessage = computed(() => {
  if (props.hasError) {
    clearFileInput();
  }
  return props.hasError || errorText.value || '';
})

const handleFileChange = (event: Event) => {
  // Если кнопка недоступна
  if (props.disabled) {
    event.preventDefault();
    return;
  }

  if (fileInputRef.value) {
    // Если файл в статусе загрузки
    if (props.uploading) {
      event.preventDefault();
      clearFileInput();
      return emit('file-cancel-download');
    }

    // Если файл загружен
    if (props.uploaded) {
      event.preventDefault();
      clearFileInput();
      return emit('file-removed');
    }

    const selectedFile = (fileInputRef.value.files && fileInputRef.value.files[0]) || null;

    if (selectedFile) {
      const maxSizeInBytes = props.maxSize * 1024 * 1024;

      if (maxSizeInBytes && selectedFile.size > maxSizeInBytes) {
        errorText.value = `Размер файла превышает ${props.maxSize} MB.`;
      } else if (props.accept && !selectedFile.type.match(props.accept)) {
        errorText.value = 'Неверный формат файла.';
      } else {
        errorText.value = '';
        file.value = selectedFile;
        emit('file-selected', selectedFile);
      }
    }
  }
};

const clearFileInput = () => {
  if (fileInputRef.value) {
    file.value = null;
    fileInputRef.value.value = '';
  }
};

</script>

<style scoped>
@keyframes rotation {
  0% {
      transform: rotate(0deg);
  }
  100% {
      transform: rotate(360deg);
  }
}

.custom-file {
  position: relative;
  display: inline-block;
  flex-direction: column;
}

.custom-file.disabled .custom-file__button {
  color: var(--neutral-400-base);
  cursor: not-allowed;
}

.custom-file__inner {
  display: flex;
  font-size: 1.4rem;
  line-height: 1.142;
}

.custom-file__button {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px;
  border-radius: 12px;
  background-color: var(--base-white);
  border: 1px solid var(--neutral-300-base);
  transition: background-color 0.3s;
  cursor: pointer;
}

.custom-file:not(.disabled) .custom-file__button:hover {
  background-color: var(--neutral-100-base);
}

.custom-file input[type=file] {
  position: absolute;
  width: 0;
  height: 0;
  opacity: 0;
  z-index: -100;
}

.custom-file:not(.disabled) input[type=file]:focus + .custom-file__button {
  box-shadow: var(--box-shadow);
}

.custom-file__info {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 16px;
  color: var(--neutral-400-base);
}

.custom-file__error {
  display: inline-block;
  font-size: 1.3rem;
  line-height: 1.2;
  margin-top: 8px;
  color: var(--error-500-base);
}

.custom-file__loading {
  width: 16px;
  height: 16px;
  margin-right: 8px;
}

.custom-file__loading svg {
  animation: rotation 2s linear infinite;
}

.custom-file__loading svg path {
  fill: var(--primary-500-base);
}

</style>
