<script setup>
import { ref } from "vue";
import usePath from "../../use/usePath"; 
import { prepareFileName } from "../../utils";
import useFiles from "../../use/useFiles";

const { uploadFileToServer } = useFiles();

const folderName = ref("");

const emit = defineEmits(['uploaded', 'close']);

const { currentPath } = usePath();

    const sendToServer = async () => {
        let result;

        if (folderName.value == "") return;

        result = await uploadFileToServer(currentPath.value + prepareFileName(folderName.value) + '/', null);

    if (result.HttpCode == 201) {

        folderName.value = "";

        setTimeout(() => {
            emit('uploaded');
        }, 300);
    }
};
</script>
<template>
    <div class="p-4">
        <div class="flex justify-between">
            <div class="font-medium">Create a folder</div>
            <Icon name="ph:x-light" class="w-6 h-6 shrink-0 ml-auto cursor-pointer" @click="$emit('close')"/>
        </div>
        <div class="mt-6">
            <label>Folder Name</label>
            <UInput type="text" v-model="folderName" class="input w-full"/>
        </div>
        <UButton @click.prevent="sendToServer" class="mt-6 btn btn-primary ml-auto">Create</UButton>
    </div>
</template>