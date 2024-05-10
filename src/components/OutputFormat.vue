<template>
    <div style="cursor: pointer">
        <v-menu>
            <template v-slot:activator="{ props }">
                <div v-bind="props" class="text-uppercase ml-2">{{ store.outputFormat }}</div>
            </template>
            <v-list style="min-width: 250px;" lines="three" label="output formats" density="compact" hide-details flat placeholder="format" :items="outputFormats" item-props @update:selected="updateHandler">
                <template v-slot:prepend="{ item }">
                    <div style="width: 140px" class="d-flex align-center text-uppercase text-h2 mr-2">{{ item.value }}</div>
                </template>
                <template v-slot:subtitle="{ subtitle, item }">
                    <div v-html="subtitle"></div>
                </template>
            </v-list>
        </v-menu>
    </div>
</template>
<style scoped>
:deep() .v-overlay__content {
    border-radius: 24px !important;
}
</style>
<script setup>
import { useAppStore } from '../store/app'

const emit = defineEmits(['update'])
const store = useAppStore()
const outputFormats = [{
    value: 'png',
    title: 'Portable Network Graphics',
    subtitle: 'https://www.w3.org/TR/png/'
}, {
    value: 'webp',
    title: 'WebP',
    subtitle: 'https://developers.google.com/speed/subtitlep/'
}, {
    value: 'gif',
    title: 'Graphics Interchange Format',
    subtitle: 'https://www.w3.org/Graphics/GIF/'
}, {
    value: 'jp2',
    title: 'JPEG 2000',
    subtitle: 'https://www.w3.org/Graphics/JPEG/jfif3.pdf'
}, {
    value: 'tiff',
    title: 'Tagged Image File Format',
    subtitle: 'https://www.w3.org/TR/tiff/'
}, {
    value: 'avif',
    title: 'AV1 Image File Format',
    subtitle: 'https://aomediacodec.github.io/av1-avif/'
}, {
    value: 'heif',
    title: 'High Efficiency Image File Format',
    subtitle: 'https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format'
}, {
    value: 'jxl',
    title: 'JPEG XL',
    subtitle: 'https://jpegxl.info/'
}, {
    value: 'raw',
    title: 'Raw',
    subtitle: 'https://en.wikipedia.org/wiki/Raw_image_format'
}]
function updateHandler(format) {
    store.outputFormat = format[0]
    emit('update')
}
</script>