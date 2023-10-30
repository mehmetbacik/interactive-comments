<template>
    <div class="confirmation-popup overlay" v-if="confirmDelete">
        <div class="confirmation-popup-content">
            <span>Delete comment</span>
            <p>Are you sure you want to delete this comment? This will remove the comment and can't be undone.</p>
            <div class="popup-button-group">
            <button @click="cancelDeleteAction" class="cancel">{{ buttonTexts.cancel }}</button>
            <button @click="confirmDeleteAction" class="confirm">{{ buttonTexts.confirm }}</button>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        props: {
            confirmDelete: {
                type: Boolean,
                required: true
            },
            buttonTexts: {
                type: Object,
                required: true
            }
        },
        methods: {
            cancelDeleteAction() {
                this.$emit("cancel");
            },
            confirmDeleteAction() {
                this.$emit("confirm");
            }
        },
    };
</script>  