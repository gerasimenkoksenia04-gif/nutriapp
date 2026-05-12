<template>
    <div>
        {{ res }}
    </div>
    <button @click="change">Change</button>
</template>

<script setup>
import { useLocalStorage } from '@vueuse/core'

const res = useLocalStorage("test", "nope")

function change() {
    res.value = "hello cahanges";
}
</script>