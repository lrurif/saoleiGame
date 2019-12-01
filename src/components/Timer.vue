<template>
    <div class="timer">
        {{hours}}小时{{minutes}}分钟{{seconds}}秒
    </div>
</template>

<script>
import eventBus from "../eventBus"
export default {
    name: "timer",
    data() {
        return {
            hours: "00",
            minutes: "00",
            seconds: "00",
            startTime: 0,
            endTime: 0,
            timeId: 0
        }
    },
    created() {
        eventBus.$on("boom-end", ()=> {
            console.log(Date.now());
            clearInterval(this.timeId);
            this.endTime = Date.now();
        }),
        eventBus.$on("start-time", ()=> {
            this.startTime = Date.now();
            let _this = this;
            this.timeId = setInterval(()=> {
                let s = Date.now() - _this.startTime;//获取时间差
                _this.seconds = String(Math.trunc(s/1000) %60).padStart(2,"0");
                _this.minutes = String(Math.trunc(s/(1000 * 60)) %60).padStart(2,"0");
                _this.hours = String(Math.trunc(s/(1000 * 60 * 60)) %60).padStart(2,"0");
            },1000)
        })
        eventBus.$on("init-time",()=> {
            this.hours = "00";
            this.minutes = "00";
            this.seconds = "00";
            clearInterval(this.timeId);
        })
    }
}
</script>

<style lang="scss">

</style>