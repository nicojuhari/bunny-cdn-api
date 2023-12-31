<script setup >
import { ref, computed, watch, watchEffect, onBeforeUnmount } from "vue"
import useStorageZones, { storageFiles, pullZones, activePullZoneURL } from "../../use/useStorageZones";

import usePath from '../../use/usePath';
import useFiles from "../../use/useFiles";

const { currentPath, changePath } = usePath();
const { deleteFileFromServer } = useFiles();

const objectGuid = ref('');
const showImageModal = ref(false)
const showImages = ref(false)


const { getStorageInfo, storageInfo, getStorageFiles, isLoading } = useStorageZones();

getStorageInfo()

watch(currentPath, () => {
    getStorageFiles(currentPath.value)
}, { immediate: true })

watchEffect(() => {
    storageInfo?.value?.PullZones.forEach((element) => {
        if (element?.Hostnames) pullZones.value.push(element?.Hostnames?.[0].Value);

        activePullZoneURL.value = storageInfo?.value?.PullZones?.[0]?.Hostnames?.[0].Value;
    });
});

onBeforeUnmount(() => {
    storageFiles.value = [];
    pullZones.value = [];
    activePullZoneURL.value = "";
});

const baseURL = computed(() => {
    return `https://${activePullZoneURL.value}/${currentPath.value}`
})

const deleteFolder = async () => {

    if (!currentPath.value) return;
    
    let paths = currentPath.value.split('/');
    // get the last folder name
    let folderName = paths[paths.length - 2];

    if (!confirm("Are you sure you want to delete this folder?")) return;
     isLoading.value = true;
    let result =  await deleteFileFromServer(currentPath.value);

    if (result.HttpCode == 200) {
        let newVal = currentPath.value.replace(folderName + '/', '')

        changePath(newVal, true)
    }

    isLoading.value = false;
}

</script>
<template>
    <div class="container my-6">
        <ClientOnly>
            <StorageBreadCrumbs />
        </ClientOnly>
        <StorageMenu v-model="showImages"/>
        <div v-if="!activePullZoneURL && !isLoading" class="my-6">
            <div class="alert alert-error text-white">
                Please connect your Bunny.net storage to a Pull Zone.
            </div>
            <!-- <img src="/bunny.net-connect-pull-zone.png" class="my-4 rounded-lg border"> -->
        </div>
        <UiLoading v-if="isLoading" />
        <section v-else class="my-4">
            <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4"
                v-if="storageFiles?.length && activePullZoneURL">
                <div v-for="item in storageFiles" class="rounded-sm overflow-hidden cursor-pointer" :key="item.Guid">
                    <div v-if="!item.IsDirectory" @click.prevent="() => { showImageModal = true; objectGuid = item.Guid }"
                        class="w-full h-full">
                        <StorageFileType v-if="showImages" :fileName="item.ObjectName" :baseURL="baseURL"></StorageFileType>
                        <div v-else class="aspect-square flex flex-col justify-center p-2">
                            <Icon name="ph:image-light" class="w-2/3 mx-auto h-auto text-gray-300" />
                            <span class="truncate text-center text-md">{{ item.ObjectName }}</span>
                        </div>
                    </div>
                    <div v-else @click.prevent="changePath(item.ObjectName)"
                        class="w-full h-full aspect-square cursor-pointer text-xl flex flex-col relative justify-center">
                        <UiIconFolder class="w-2/3 mx-auto h-auto" />
                        <span class="truncate text-center text-md">{{ item.ObjectName }}</span>
                    </div>
                </div>
            </div>
            <div v-else class="text-center my-16">No Files in This Directory!</div>
        </section>
        <div class="mt-6 border-t py-4">
            <div v-if="currentPath == ''">You are currently in the root folder</div>
            <div v-else class="flex items-center">
                <div>You are currently in the folder: <span class="font-bold">{{ currentPath }}</span></div>
                <UButton color="red" size="xs" class="ml-auto !text-white cursor-pointer shrink-0"
                    @click="deleteFolder" title="Delete this folder">
                    <Icon name="ph:trash-light" class="w-6 h-6" /> Delete this Folder
                </UButton>
            </div>
        </div>
        <UModal v-model="showImageModal">
            <ModalsViewFile  :objectGuid="objectGuid" @close="showImageModal = false"/>
        </UModal>
    </div>
</template>