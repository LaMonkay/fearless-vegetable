<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import Greet from "./components/Greet.vue";
import { ref } from 'vue'
import { checkUpdate, installUpdate } from '@tauri-apps/api/updater'
import { relaunch } from '@tauri-apps/api/process'
import { writeTextFile, BaseDirectory, readTextFile } from '@tauri-apps/api/fs';
import { merge } from 'lodash';
import { listen } from '@tauri-apps/api/event'
import { appConfigDir, appDataDir } from '@tauri-apps/api/path';


import { open } from '@tauri-apps/api/dialog';

async function openFile() {
  try {
    const selectedPath: string|string[]|null = await open()
    const contents = await readTextFile(selectedPath as string);
    console.log(contents)
  } catch (err) {
    console.error(err)
  }
}


async function dragDrop() {
  listen('tauri://file-drop', event => {
    console.log(event)
    fileDropped(event)
  })
}

dragDrop()

async function fileDropped(event: any) {
  const contents = await readTextFile(event.payload[0]);
  console.log(contents)
} 

async function update() {
  try {
    const { shouldUpdate, manifest } = await checkUpdate()
    if (shouldUpdate) {
      // display dialog
      await installUpdate()
      // install complete, restart the app

      await myWriteTextFile({'recent_update': {'just_updated': true}})

      await relaunch()
    }
  } catch (error) {
    console.log(error)
  }
}

update()

const newUpdate = ref('')

const myReadTextFile = async () => {
  const contents = await readTextFile('storage.json', { dir: BaseDirectory.AppConfig });
  const parsedContents = JSON.parse(contents);
  return parsedContents;
}

const myWriteTextFile =  async (paramPath: object) => {
  const contents = await myReadTextFile()
  const newStorage = merge(contents, paramPath)

  writeTextFile('storage.json', JSON.stringify(newStorage), { dir: BaseDirectory.AppConfig })
}

async function updateMessage() {
  const contents = await myReadTextFile()
  
  if (contents.recent_update.just_updated == true) {
    newUpdate.value = contents.recent_update.update_message
    myWriteTextFile({'recent_update': {'just_updated': false}})
  }
}

updateMessage()
</script>

<template>
  <div class="container">
    <h1>Welcome to Tauri!</h1>

    <div class="row">
      <a href="https://vitejs.dev" target="_blank">
        <img src="/vite.svg" class="logo vite" alt="Vite logo" />
      </a>
      <a href="https://tauri.app" target="_blank">
        <img src="/tauri.svg" class="logo tauri" alt="Tauri logo" />
      </a>
      <a href="https://vuejs.org/" target="_blank">
        <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
      </a>
    </div>

    {{ newUpdate }}

    <p>Click on the Tauri, Vite, and Vue logos to learn more.</p>

    <p>
      Recommended IDE setup:
      <a href="https://code.visualstudio.com/" target="_blank">VS Code</a>
      +
      <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
      +
      <a href="https://github.com/tauri-apps/tauri-vscode" target="_blank"
        >Tauri</a
      >
      +
      <a href="https://github.com/rust-lang/rust-analyzer" target="_blank"
        >rust-analyzer</a
      >
    </p>

    <button @click="openFile()">Browse Files</button>

    <Greet />
  </div>
</template>

<style scoped>
.logo.vite:hover {
  filter: drop-shadow(0 0 2em #747bff);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #249b73);
}
</style>
