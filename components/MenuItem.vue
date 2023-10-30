<template>
    <NuxtLink :to="props.path">
        <div
            class="hover:bg-gradient-to-r hover:from-cyan-700 hover:to-cyan-800 hover:bg-opacity-50 h-20 rounded-xl cursor-pointer select-none relative border border-cyan-300 border-opacity-50">
            <span class="text-center w-full inline-block align-text-bottom absolute top-1/3">
                <slot />
            </span>
        </div>
    </NuxtLink>
</template>

<script setup>
let props = defineProps({
    path: {
        type: String,
        required: true
    }
})
</script>