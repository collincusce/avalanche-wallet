<template>
    <p
        class="spinner"
        :style="{
            color: color,
        }"
    >
        <fa icon="spinner"></fa>
    </p>
</template>
<script>
export default {
    props: {
        color: {
            type: String,
            default: '#fff',
        },
    },
}
</script>
<style scoped lang="scss">
.spinner {
    width: 20px;
    height: 20px;
    animation-name: spin;
    animation-duration: 1s;
    animation-iteration-count: infinite;
    transform-origin: center;
    display: flex;
    justify-content: center;
    align-items: center;
}
@keyframes spin {
    from {
        transform: rotateZ(0deg);
    }
    to {
        transform: rotateZ(360deg);
    }
}
</style>
