<template>
    <UCard class="min-h-screen min-w-full">
        <template #header>
            <p class="text-center text-xl">
                <NuxtLink to="/">My Web Utils</NuxtLink>
            </p>
        </template>

        <div class="grid grid-cols-12">
            <div class="lg:col-start-2 lg:col-end-12 col-start-1 col-end-13">
                <slot />
            </div>
        </div>
    </UCard>
</template>