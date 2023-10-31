<template>
    <table class="table-auto font-sans border-spacing-2">
        <caption class="text-slate-300 pb-2 text-sm caption-top" v-if="caption && caption?.length > 0">
            {{ props.caption }}
        </caption>
        <tbody class="text-xs">
            <tr v-for="(r, idx) in props.rows" :key="idx">
                <td class="text-slate-500 dark:text-slate-400 break-words">{{ r.key }}</td>
                <td class="text-slate-500 dark:text-slate-400 break-all">{{ r.value }}</td>
            </tr>
        </tbody>
    </table>
</template>

<script setup lang="ts">
import type { Entry } from '~/composables/models/Entry';

const props = defineProps({
    caption: String,
    rows: {
        type: Array<Entry>,
        required: true
    }
})
</script>