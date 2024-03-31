<template>
  <!-- 
    inputID - id инпута                                   : string
    text - Текст кнопки (смотреть computed - textButton)  : string
    accept - Принимаемые типы файлов                      : string
    has-error - Ошибка от родительского компонента        : string
    max-size - Максимальный размер файла (в мегабайтах)   : number
    uploading - Статус "загрузки в текущий момент"        : boolean
    uploaded - Статус "загружен ли файл"                  : boolean
    disabled - disabled для инпута                        : boolean
   -->
  <CustomFileInput 
    inputID="someId"
    :text="textButton"
    :accept="accept"
    :has-error="hasError"
    :max-size="maxSize"
    :uploading="uploading"
    :uploaded="fileIsUploaded"
    :disabled="disabled"
    @file-selected="uploadFileHandler"
    @file-cancel-download="stopUploadingHandler"
    @file-removed="removeFileHandler"
  />
</template>

<script setup lang="ts">
import { ref, Ref, computed } from 'vue';
import CustomFileInput from './components/UI/CustomFileInput.vue';

const accept = '';
const hasError: Ref<string> = ref('');
const maxSize = 15;
const uploading: Ref<boolean> = ref(false);
const fileIsUploaded: Ref<boolean> = ref(false);
const disabled = false;

const textButton = computed(() => {
  if (uploading.value) return 'Отменить';
  if (!uploading.value && fileIsUploaded.value) return 'Удалить';
  return 'Выбрать файл'
});

const uploadFileHandler = (file: File) => {
  const testIsSuccess = true; // True - загрузка успешная, False - неуспешная
  const timeout = 5 * 1000;
  
  // Условный запрос на сервер
  new Promise((res, rej) => {
    hasError.value = '';
    uploading.value = true;

    setTimeout(() => {
      const data = {};

      testIsSuccess ? res(data) : rej(data);
    }, timeout);
  })
  .then(() => {
    if (!uploading.value) return;  // Условное прерывание запроса

    fileIsUploaded.value = true;
    console.log('Файл успешно загружен', file.name)
  })
  .catch((data) => {
    if (!uploading.value) return; // Условное прерывание запроса

    fileIsUploaded.value = false;
    hasError.value = 'Произошла ошибка при загрузке файла.';
    console.error('Ошибка:', data);
  })
  .finally(() => {
    uploading.value = false;
  })
}

const stopUploadingHandler = () => {
  // Условное прерывание запроса
  uploading.value = false;
  hasError.value = '';
}

const removeFileHandler = () => {
  // Фетч на удаление / другой функционал / ...
  fileIsUploaded.value = false;
}
</script>

<style>
#app {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
</style>
