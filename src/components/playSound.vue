<template>
    <div class="playsound">
        <audio :src="srcSound" ref="sound"></audio>
    </div>
</template>

<script>
import eventBus from "../eventBus.js";
export default {
    name:"playsound",
    props: {
        srcSound: {
            type: String,
            required: true
        }
    },
    created() {
        eventBus.$on("click-cell",()=> {
            this.$refs.sound.play();
        })
    },
    data() {
        return {

        }
    }
}
</script>

<style lang="scss">
    .playsound {
        height: 0;
        width: 0;
        display: none;
    }
</style>