<script setup lang="ts">
import { type Ref } from 'vue'
import { type ReplStore, type VersionKey } from '@/composables/store'
import {
  getSupportedEpVersions,
  getSupportedTSVersions,
  getSupportedVueVersions,
} from '@/utils/dependency'

const appVersion = import.meta.env.APP_VERSION
const replVersion = import.meta.env.REPL_VERSION

const emit = defineEmits<{
  (e: 'refresh'): void
}>()
const nightly = ref(false)
const dark = useDark()
const toggleDark = useToggle(dark)

const { store } = defineProps<{
  store: ReplStore
}>()

interface Version {
  text: string
  published: Ref<string[]>
  active: string
}

const versions = reactive<Record<VersionKey, Version>>({
  elementPlus: {
    text: 'Element Plus',
    published: getSupportedEpVersions(nightly),
    active: store.versions.elementPlus,
  },
  vue: {
    text: 'Vue',
    published: getSupportedVueVersions(),
    active: store.versions.vue,
  },
  typescript: {
    text: 'TypeScript',
    published: getSupportedTSVersions(),
    active: store.versions.typescript,
  },
})

async function setVersion(key: VersionKey, v: string) {
  versions[key].active = `loading...`
  await store.setVersion(key, v)
  versions[key].active = v
}

const toggleNightly = () => {
  store.toggleNightly(nightly.value)
  setVersion('elementPlus', 'latest')
}

async function copyLink() {
  await navigator.clipboard.writeText(location.href)
  ElMessage.success('Sharable URL has been copied to clipboard.')
}

function refreshView() {
  emit('refresh')
}
</script>

<template>
  <nav>
    <div leading="[var(--nav-height)]" m-0 flex items-center font-medium>
      <img
        h-24px
        relative
        mr-2
        v="mid"
        top="-2px"
        alt="logo"
        src="../assets/logo.svg"
      />
      <div lt-sm-hidden flex="~ gap-1" items-center>
        <div text-xl>Element Plus Playground</div>
        <el-tag size="small">v{{ appVersion }}, repl v{{ replVersion }}</el-tag>
        <el-tag v-if="store.pr" size="small">PR {{ store.pr }}</el-tag>
      </div>
    </div>

    <div flex="~ gap-2" items-center>
      <div
        v-for="(v, key) of versions"
        :key="key"
        flex="~ gap-2"
        items-center
        lt-lg-hidden
      >
        <span>{{ v.text }}:</span>
        <el-select
          :model-value="v.active"
          size="small"
          fit-input-width
          w-36
          @update:model-value="setVersion(key, $event)"
        >
          <el-option v-for="ver of v.published" :key="ver" :value="ver">
            {{ ver }}
          </el-option>
        </el-select>

        <el-checkbox
          v-if="key === 'elementPlus'"
          v-model="nightly"
          @change="toggleNightly"
        >
          nightly
        </el-checkbox>
      </div>

      <div flex="~ gap-4" text-lg>
        <button hover:color-primary i-ri-refresh-line @click="refreshView" />
        <button hover:color-primary i-ri-share-line @click="copyLink" />
        <button
          hover:color-primary
          i-ri-sun-line
          dark:i-ri-moon-line
          @click="toggleDark()"
        />
        <a
          href="https://github.com/element-plus/element-plus-playground"
          target="_blank"
          flex
          hover:color-primary
        >
          <button title="View on GitHub" i-ri-github-fill />
        </a>

        <el-popover trigger="click" width="300px">
          <Settings />
          <template #reference>
            <button hover:color-primary i-ri:settings-line />
          </template>
        </el-popover>
      </div>
    </div>
  </nav>
</template>

<style lang="scss">
nav {
  --bg: #fff;
  --bg-light: #fff;
  --border: #ddd;

  --at-apply: 'box-border flex justify-between px-4 z-999 relative';

  height: var(--nav-height);
  background-color: var(--bg);
  box-shadow: 0 0 6px var(--el-color-primary);
}

.dark nav {
  --bg: #1a1a1a;
  --bg-light: #242424;
  --border: #383838;

  --at-apply: 'shadow-none';
  border-bottom: 1px solid var(--border);
}
</style>
