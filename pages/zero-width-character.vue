<template>
    <p class="text-xl text-opacity-10 opacity-50">Zero width character</p>
    <Hr />
    <UTabs :items="items">
        <template #default="{ item, index }">
            <div class="flex items-center gap-2 relative truncate">
                <UIcon :name="item.icon" class="w-4 h-4 flex-shrink-0" />
                <span class="truncate">{{ item.label }}</span>
            </div>
        </template>
        <template #item="{ item }">
            <UCard class="text-xs">
                <template #header>
                    <h3 class="text-base font-semibold leading-6 text-gray-900 dark:text-white">
                        {{ item.label }}
                    </h3>
                    <p class="mt-1 text-sm text-gray-500 dark:text-gray-400">
                        {{ item.description }}
                    </p>
                </template>
                <!-- Encode Hidden Part -->
                <div v-if="item.key == 'encode'">
                    <p class="text-xs my-2">Hidden Text: {{ encodeHidden.length > 0 ? `(${encodeHidden.length})` :
                        '' }}</p>
                    <UTextarea class="mb-3" :rows="2" variant="outline" name="textarea" placeholder="Hidden Text"
                        v-model="encodeHidden" />
                </div>
                <div class="grid md:grid-cols-2 gap-4 divide-dashed grid-cols-1">
                    <!-- Filter Part -->
                    <div v-if="item.key == 'filter'">
                        <p class="text-xs my-2">Original Text: {{ filterOriginal.length > 0 ? `(${filterOriginal.length})` :
                            '' }}</p>
                        <UTextarea :rows="10" variant="outline" name="textarea" placeholder="Original Text"
                            v-model="filterOriginal" />
                    </div>

                    <div v-if="item.key == 'filter'">
                        <p class="text-xs my-2">{{ item.processedTitle }} {{ filterAfter.length > 0 ?
                            `(${filterAfter.length})` : '' }}</p>
                        <UTextarea readonly :rows="10" variant="outline" name="textarea" :placeholder="item.processedTitle"
                            :value="filterAfter" @click="copyToClipboard(filterAfter)" />
                    </div>
                    <!-- Encode Part -->
                    <div v-if="item.key == 'encode'">
                        <p class="text-xs my-2">Original Text: {{ encodeOriginal.length > 0 ? `(${encodeOriginal.length})` :
                            '' }}</p>
                        <UTextarea :rows="10" variant="outline" name="textarea" placeholder="Original Text"
                            v-model="encodeOriginal" />
                    </div>

                    <div v-if="item.key == 'encode'">
                        <p class="text-xs my-2">{{ item.processedTitle }} {{ encodeAfter.length > 0 ?
                            `(${encodeAfter.length})` : '' }}</p>
                        <UTextarea readonly :rows="10" variant="outline" name="textarea" :placeholder="item.processedTitle"
                            :value="encodeAfter" @click="copyToClipboard(encodeAfter)" />
                    </div>
                    <!-- Decode Part -->
                    <div v-if="item.key == 'decode'">
                        <p class="text-xs my-2">Original Text: {{ decodeOriginal.length > 0 ? `(${decodeOriginal.length})` :
                            '' }}</p>
                        <UTextarea :rows="10" variant="outline" name="textarea" placeholder="Original Text"
                            v-model="decodeOriginal" />
                    </div>

                    <div v-if="item.key == 'decode'">
                        <p class="text-xs my-2">{{ item.processedTitle }} {{ decodeAfter.length > 0 ?
                            `(${decodeAfter.length})` : '' }}</p>
                        <UTextarea readonly :rows="10" variant="outline" name="textarea" :placeholder="item.processedTitle"
                            :value="decodeAfter" @click="copyToClipboard(decodeAfter)" />
                    </div>
                </div>
            </UCard>
        </template>
    </UTabs>
</template>

<script setup lang="ts">

import { ref, watch } from "vue";
// import unicodeSteganography from 'unicode_steganography';
import { UnicodeSteganography } from '../composables/UnicodeSteganography';

const toast = useToast();

const items = [{
    key: 'filter',
    label: 'Sensitive Word Filter',
    description: 'This is used to prevent basic sensitive detection in Chinese internets. It will add a zero width character after every word.',
    processedTitle: 'Filtered Text:',
    icon: 'i-heroicons-funnel'
}, {
    key: 'encode',
    label: 'Encode',
    description: 'Hide information in your text.',
    processedTitle: 'Encoded Text:',
    icon: 'i-heroicons-lock-closed'
},
{
    key: 'decode',
    label: 'Decode',
    description: 'Decode information in you text.',
    processedTitle: 'Decoded Text:',
    icon: 'i-heroicons-lock-open'
}
];

const filterOriginal = ref('');
const filterAfter = ref('');

const encodeOriginal = ref('')
const encodeAfter = ref('')
const encodeHidden = ref('')


const decodeOriginal = ref('')
const decodeAfter = ref('')

const steg = UnicodeSteganography.getInstance();

watch(filterOriginal, () => {
    filterAfter.value = filterOriginal.value.length > 0 ? steg.sensitiveFilter(filterOriginal.value) : '';
});

watch(encodeOriginal, () => {
    encodeAfter.value = encodeOriginal.value.length > 0 ? steg.encodeText(encodeOriginal.value, encodeHidden.value) : '';
});

watch(encodeHidden, () => {
    encodeAfter.value = encodeOriginal.value.length > 0 ? steg.encodeText(encodeOriginal.value, encodeHidden.value) : '';
});

watch(decodeOriginal, () => {
    if (decodeOriginal.value.length > 0) decodeAfter.value = steg.decodeText(decodeOriginal.value).hiddenText;
    else decodeAfter.value = '';
});


const copyToClipboard = (e: string) => {
    if (e && e.length > 0) {
        navigator.clipboard.writeText(e);
        toast.add({ 'title': 'Have copied to clipboard.', timeout: 2000 })
    }
}

</script>