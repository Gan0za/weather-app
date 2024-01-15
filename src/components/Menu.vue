<template>
    <div class="fullscreen">
        <div class="fixet_full justify-right" style="padding: 24px;" @click.self="menu_confirm = !menu_confirm">
            <div class="dialog">
                <div class="justify-center">
                    test
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref } from 'vue';

const props = defineProps(['menu_confirm']);
const emit = defineEmits(['close']);

const menu_confirm = computed({
    get() {
        return props.menu_confirm;
    },
    set(new_value) {
        if (new_value === false) {
            emit('close');
        }
    }
});
</script>

<style scoped></style>