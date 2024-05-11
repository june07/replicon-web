<template>
    <v-card :class="smAndDown ? 'mobile' : ''" class="h-100 d-flex flex-column align-center justify-center mx-auto" flat>
        <v-card-title :class="smAndDown ? 'text-h5' : 'text-h2'" class="d-flex font-weight-bold pb-0">
            SVG to <output-format @update="complete = false" />
        </v-card-title>
        <v-card-subtitle :class="{ 'text-body-1': smAndDown, animate__headShake: showInstallButtons, 'animate__delay-0': showInstallButtons, 'animate__animated animate__headShake': showInstallButtons }" class="d-flex text-wrap text-h5">
            Convert your SVG quick and easy.
        </v-card-subtitle>
        <v-card-text class="d-flex align-center justify-center">
            <v-container fluid>
                <div class="text-body-1 text-center mb-8">
                    This tool uses the <a href="https://www.npmjs.com/package/@667/replicon" target="_blank">Replicon</a> npm library.
                </div>
                <div class="d-flex align-center justify-center">
                    <div id="dropzone" class="dropzone" @success="onSuccess" @error="onError" @file-loaded="onFileLoaded" @file-error="onFileError">
                    </div>
                </div>
                <div v-show="showOptions" class="d-flex flex-column align-center">
                    <v-sheet min-width="500" class="pa-4 mb-8">
                        <v-row class="d-flex align-center">
                            <v-col :cols="3" class="text-end">sizes</v-col>
                            <v-col :cols="9" class="d-flex">
                                <div class="d-flex" v-for="resolution of resolutionsOption">
                                    <v-checkbox density="compact" v-model="store.resolutions" :label="resolution.name" :value="resolution.value" multiple hide-details>
                                        <template v-slot:label></template>
                                    </v-checkbox>
                                    <div class="labels text-caption" :class="store.resolutions.includes(resolution.value) ? 'font-weight-bold' : ''">{{ resolution.name.split('x')[1] + 'px' }}</div>
                                </div>
                            </v-col>
                        </v-row>
                    </v-sheet>
                    <v-btn @click="submitHandler" variant="tonal" color="deep-purple-darken-2" rounded="lg" class="font-weight-light" :ripple="false" text="replicate" :loading="loading" :disabled="complete" />
                </div>
            </v-container>
        </v-card-text>
        <v-card-actions class="w-100 d-flex justify-center align-center" style="height: 100px;">
            <v-btn v-for="link of links" variant="text" color="deep-purple-darken-2" :href="link.href" :text="link.text" rounded="lg" class="font-weight-light" :ripple="false" stacked>
                <template v-slot:prepend>
                    <v-img :src="link.image" width="64" height="64" class="mx-1" style="border-radius: 12px;" :cover="/charles-duck-unitas-IZWSIyb3kgo-unsplash./.test(link.image)" />
                </template>
            </v-btn>
        </v-card-actions>
    </v-card>
</template>
<style scoped>
.labels {
    width: 50px;
    margin-top: 45px;
    margin-left: 10px;
    position: absolute;
    transform: rotate(45deg);
}

.dropzone {
    border: 0;
    border-radius: 24px;
    background-color: #EDE7F6;
}

:deep() .dz-preview {
    border-radius: 24px;
}

:deep() img[data-dz-thumbnail] {
    width: -webkit-fill-available;
}
</style>
<script setup>
import 'animate.css'
import 'dropzone/dist/dropzone.css'
import { ref, computed, onMounted, getCurrentInstance } from 'vue'
import { useDisplay } from 'vuetify/lib/framework.mjs'
import { useAppStore } from '../store/app'
import { Dropzone } from "dropzone"
import OutputFormat from '@/components/OutputFormat.vue'

const { $api } = getCurrentInstance().appContext.config.globalProperties
const { VITE_APP_API_SERVER } = import.meta.env
const store = useAppStore()
const showOptions = computed(() => store.serverId)
const defaultResolutions = [
    { name: '16x16', value: 16, },
    { name: '32x32', value: 32, },
    { name: '48x48', value: 48, },
    { name: '64x64', value: 64, },
    { name: '96x96', value: 96, },
    { name: '128x128', value: 128, },
    { name: '256x256', value: 256, },
    { name: '384x384', value: 384, },
    { name: '512x512', value: 512, },
]
const resolutionsOption = ref(defaultResolutions)
const dropzone = ref()
const loading = ref(false)
const complete = ref(false)
const showInstallButtons = ref(false)
const { smAndDown } = useDisplay()
const links = [{
    name: 'Read', href: 'https://www.amazon.com/Tom-All-Trades-Lanette-Wardell-ebook/dp/B019D8UEDM', text: 'read', image: '/charles-duck-unitas-IZWSIyb3kgo-unsplash.webp'
}, {
    name: 'Blog', href: 'https://june07.com', text: 'blog', image: '/SSPX0088-webp',
}, {
    name: 'NPM', href: 'https://www.npmjs.com/package/@667/replicon', text: 'replicon', image: '/Npm-logo.svg'
}]
async function submitHandler() {
    try {
        loading.value = true

        const zip = await $api.replicate(store.serverId, {
            options: {
                baseResolutions: store.resolutions,
                outputFormat: store.outputFormat,
            }
        })

        if (zip) {
            const blob = new Blob([zip], { type: 'application/zip' })
            const url = URL.createObjectURL(blob)
            const link = document.createElement('a')
            link.href = url
            link.download = 'replicon.zip'
            link.click()
        }
        complete.value = true
    } catch (error) {
        console.error(error)
        if (/429/.test(error.response.status)) {
            console.log('Too many requests. Please try again later.')
        } else {
            dropzone.value.removeAllFiles(true)
        }
    } finally {
        loading.value = false
    }
}
onMounted(() => {
    dropzone.value = new Dropzone("#dropzone", {
        url: `${VITE_APP_API_SERVER}/v1/replicon/upload`,
        maxFiles: 1,
        acceptedFiles: "image/svg+xml",
        thumbnailMethod: 'contain',
        dictDefaultMessage: 'Drop svg file here or click to upload',
        success: (file, serverId) => {
            store.file = file
            store.serverId = serverId
        },
    })
    /**
    if (store.file && store.serverId) {
        dropzone.value.emit('addedfile', store.file)
        dropzone.value.emit('thumbnail', store.file, store.file.dataURL)
        dropzone.value.emit('complete', store.file)
    }
     */
})
</script>
