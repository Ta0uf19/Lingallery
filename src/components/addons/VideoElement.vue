<template>
    <video class="awesome-video" :controls="item.video.controls" :autoplay="item.video.autoplay" :muted="item.video.muted">
        <source
                :src="item.video.src"
                :type="item.video.type ? item.video.type : false"
        />
    </video>
</template>

<script>
    export default {
        name: "VideoElement",
        props: {
            item: {
                type: Object,
                required: true,
            },
            controls: {
                type: Boolean,
                default: true
            },
            autoplay: {
                type: Boolean,
                default: false
            },
            muted: {
                type: Boolean,
                default: false
            }
        },
    }
</script>

<style>
    /* responsive */
    .awesome-video {
        max-width: 100%;
        height: 95%;
    }
</style>
