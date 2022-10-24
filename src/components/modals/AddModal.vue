<template>
  <ui-modal
    v-model:show="isModalModel"
    :title="title"
    @submit="onAdd"
  >
    <div class="add-modal">
      <div class="add-modal__inputs">
        <n-input placeholder="Title" v-model:value="form.name"/>
        <n-tabs
          v-model:value="selectedTab"
        >
          <n-tab-pane
            tab="File"
            name="file"
          >
            <n-upload
              ref="uploader"
              :on-change="onFileUpload"
              :max="1"
              accept=".png,.jpg,.jpeg,.gif"
            >
              <n-upload-dragger style="background: #E3DED6;">
                <div style="margin-bottom: 12px">
                  <n-icon size="48" :depth="3">
                    <archive-outlined />
                  </n-icon>
                </div>
                <n-text>
                  Click or drag an image to this area to upload
                </n-text>
              </n-upload-dragger>
            </n-upload>
          </n-tab-pane>
          <n-tab-pane
            tab="Link"
            name="link"
          >
            <n-input
              placeholder="Image link"
              v-model:value="form.image"
            />
          </n-tab-pane>
        </n-tabs>
        <n-input-number
          v-if="props.count"
          placeholder="Count"
          v-model:value="form.count"
          :min="1"
          :max="999"
        />
        <n-input
          type="textarea"
          placeholder="Description"
          :resizable="false"
          v-model:value="form.description"
        />
        <n-dynamic-input
          v-if="props.variables"
          v-model:value="form.variables"
          preset="pair"
        />
      </div>
      <div class="add-modal__preview">
        <img
          v-if="previewImage"
          class="card-preview"
          :src="previewImage"
          alt="preview"
        />
        <div class="card-preview card-preview--unsetted" v-else></div>
      </div>
    </div>
  </ui-modal>
</template>

<script setup>
import UiModal from '@/components/ui/uiModal.vue';
import {
  computed,
  defineEmits,
  defineProps,
  reactive,
  watch,
  toRef,
  ref,
} from 'vue';
import { ArchiveOutlined } from '@vicons/material';

const emit = defineEmits(['update:show', 'submit']);
const props = defineProps({
  show: {
    type: Boolean,
    default: false,
  },
  initialData: {
    type: Object,
    default: () => {},
  },
  entityType: {
    type: String,
    default: '',
  },
  count: {
    type: Boolean,
    default: false,
  },
  variables: {
    type: Boolean,
    default: false,
  },
});

const isModalModel = computed({
  get() {
    return props.show;
  },
  set(val) {
    emit('update:show', val);
  },
});

const form = reactive({
  name: '',
  image: '',
  description: '',
  count: 1,
  variables: [],
  imageFile: null,
});

const initialDataRef = toRef(props, 'initialData');
const entityTypeRef = toRef(props, 'entityType');

const mode = computed(() => (Object.keys(initialDataRef.value).length ? 'edit' : 'create'));
const uploader = ref(null);

const clearFile = () => {
  form.imageFile = null;
  if (uploader.value) {
    uploader.value.clear();
  }
};

const selectedTab = ref('');

watch(selectedTab, () => {
  clearFile();
});

const previewImage = computed(() => {
  if (form.imageFile) {
    return URL.createObjectURL(form.imageFile);
  }
  if (form.image === initialDataRef?.value?.image) {
    return initialDataRef?.value?.cachedImage;
  }
  return form.image;
});

const title = computed(() => (
  `${mode.value[0].toUpperCase() + mode.value.slice(1)} `
  + `${entityTypeRef.value.slice(0, entityTypeRef.value.length - 1)}`
));

watch(isModalModel, (val) => {
  if (val) {
    selectedTab.value = initialDataRef.value?.image ? 'link' : 'file';
  }
  form.name = initialDataRef.value?.name || '';
  form.image = initialDataRef.value?.image || '';
  form.description = initialDataRef.value?.description || '';
  form.count = initialDataRef.value?.count || 1;
  form.variables = initialDataRef.value?.variables
    ? Object.entries(initialDataRef.value?.variables).map(([key, value]) => ({ key, value }))
    : [];
  clearFile();
});

const onFileUpload = ({ file }) => {
  if (file?.status === 'removed') {
    form.imageFile = null;
    return;
  }
  form.imageFile = file.file;
};

const onAdd = () => {
  const variables = form.variables.reduce((acc, cur) => {
    acc[cur.key] = cur.value;
    return acc;
  }, {});
  const formData = new FormData();
  // eslint-disable-next-line no-restricted-syntax
  for (const [key, value] of Object.entries(form)) {
    if (form.imageFile && key === 'image') {
      // eslint-disable-next-line no-continue
      continue;
    }
    if (key === 'variables') {
      if (Object.keys(variables).length) {
        formData.append(key, JSON.stringify(variables));
      }
      // eslint-disable-next-line no-continue
      continue;
    }
    formData.append(key, value);
  }
  emit('submit', { mode: mode.value, data: formData });
};
</script>

<style lang="scss">
.add-modal {
  height: 100%;
  display: grid;
  grid-template-columns: 50% 50%;
  &__inputs {
    padding: 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 5px;
  }
  &__preview {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
.card-preview {
  max-width: 205px;
  border: 2px #138b44 solid;
  border-radius: 8px;
  user-select: none;
  &:hover {
    animation: .3s scalePreview forwards;
    animation-delay: 300ms;
  }
  &--unsetted {
    width: 205px;
    aspect-ratio: 0.71;
    background-color: #138b44;
    border: 2px #138b44 solid;
    border-radius: 8px;
  }
}

@keyframes scalePreview {
  from {
    transform: scale(1);
  }
  to {
    transform: scale(1.6);
  }
}

.n-image-preview-toolbar {
  //Remove rotate buttons
  i:nth-child(1), i:nth-child(2) {
    width: 100px;
    display: none;
  }
}
</style>