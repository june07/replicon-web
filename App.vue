<template>
    <v-app>
        <v-main>
            <v-container class="h-100 d-flex justify-center">
                <replicon-main />
            </v-container>
        </v-main>
        <v-snackbar text :timeout="-1" v-model="snackbar.active" style="opacity: 0.9" @click="snackbarCloseHandler">
            <v-row>
                <v-col cols="1" class="d-flex align-center justify-center">
                    <v-icon :icon="snackbar.icon" :color="snackbar.iconColor" />
                </v-col>
                <v-col cols="10" class="d-flex align-center justify-center">
                    <span v-if="error">{{ snackbar.message }}</span>
                    <span v-else @click="reload" class="font-weight-light" v-bind:class="smAndDown ? 'caption' : ''" style="cursor: pointer">App update available.</span>
                </v-col>
                <v-col cols="1" class="d-flex align-center justify-center">
                    <v-btn variant="plain" :size="smAndDown ? 'x-small' : ''" @click="snackbarCloseHandler"> x </v-btn>
                </v-col>
            </v-row>
        </v-snackbar>
    </v-app>
</template>
<style scoped>
:deep() .v-icon {
    color: #673AB7 !important;
}
</style>
<script setup>
import { ref, getCurrentInstance } from "vue"

import RepliconMain from "./src/components/RepliconMain.vue"

const { $api } = getCurrentInstance().appContext.config.globalProperties
const version = ref()
const snackbarDefault = {
    active: false,
    icon: 'info',
    message: undefined,
}
const snackbar = ref({ ...snackbarDefault })
const lastBuild = ref()
const versionCheckIntervalId = ref()
const buildInfo = ref()

const checkVersion = async () => {
    buildInfo.value = await $api.buildInfo()

    if (!buildInfo.value?.build_date) {
        return
    }
    version.value = buildInfo.value.commit_sha
    if (lastBuild.value && lastBuild.value?.build_date !== buildInfo.value.build_date) {
        snackbar.value.active = true
        // TODO: put some extra check here...
        resetLocalStorage.value = true
    } else {
        lastBuild.value = buildInfo.value
    }
}
function snackbarCloseHandler() {
    snackbar.value.active = false
    setTimeout(() => {
        snackbar.value = { ...snackbarDefault }
        if (error) {
            error.value = false
        }
    })
}
function reload() {
    const url = new URL(window.location.href)
    url.searchParams.set('cache', Date.now())
    window.location.href = url.toString()
}
checkVersion()
versionCheckIntervalId.value = setInterval(checkVersion, 60000)
</script>
