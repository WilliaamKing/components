<template>
    <div class="v-scroller scroller-wrapper" ref="wrapper">
        <div class="scroller" ref="scrollContainer"
             @touchstart.prevent="handleStart" 
             @touchmove.prevent="handleMove" 
             @touchend.prevent="handleEnd">
            <slot></slot>
        </div>
         <div class="scrollbar-wrapper">
             <div class="scrollbar" :style="scrollbarStyle"></div>
         </div>
    </div>
</template>

<script>
export default {
    name: "v-scroller",
    data (){
        return {
            wrapper: {},
            scroller: {},
            maxMoveY: null,
            startCoords: null,
            scrollbarLength: 0,
            scrollbarTransformValue: 0,
        }
    },
    computed: {
        scrollbarStyle (){
            return {
                height: `${this.scrollbarLength}%`,
                transform: `translateY(${this.scrollbarTransformValue}%)`,
            }
        }
    },
    methods: {
        handleStart (event){
            const {pageX: x, pageY: y} = event.changedTouches[0];
            this.startCoords = {
                x: x - this.wrapper.x, 
                y: y - this.wrapper.y
            };
        },
        handleMove (event){
            const {pageX: x, pageY: y} = event.changedTouches[0];
    
            if (this.isCursorInScroller (x, y)) {
                 const currentCoords = {
                     x: x - this.wrapper.x,
                     y: y - this.wrapper.y,
                 };

                if (this.startCoords.y >= currentCoords.y) {
                    const moveValue = (this.startCoords.y - currentCoords.y);
                    this.scroller.y = (this.scroller.y - moveValue >= this.maxMoveY) ? this.scroller.y - moveValue : this.maxMoveY;
                }
                else {
                    const moveValue = currentCoords.y - this.startCoords.y;
                    this.scroller.y = (this.scroller.y + moveValue <= 0) ? this.scroller.y + moveValue : 0;
                }  

                this.scrollbarTransformValue = (this.scroller.y / this.scroller.height * 100) * -1;
                console.log(this.scroller.height, this.scroller.y);
                console.log(this.scrollbarTransformValue);
                this.$refs.scrollContainer.style.transform = `translateY(${this.scroller.y}px)`;
            }
        },
        handleEnd (){
            this.startCoord = null;
        },
        isCursorInScroller (x, y) {
            const startCoords = {
                x: this.wrapper.x,
                y: this.wrapper.y
            };

            const endCoords = {
                x: this.wrapper.x + this.wrapper.width,
                y: this.wrapper.y + this.wrapper.height,
            }

            const conditions = [x >= startCoords.x, y >= startCoords.y, x <= endCoords.x, y <= endCoords.y];
            return conditions.every(el => el);
        },
        refresh (){
            const { wrapper, scrollContainer } = this.$refs;
            const wrapperData = wrapper.getBoundingClientRect();
            const scrollData = scrollContainer.getBoundingClientRect();
        
            this.wrapper.x = wrapperData.x - 1;
            this.wrapper.y = wrapperData.y - 1;
            this.wrapper.width = wrapperData.width;
            this.wrapper.height = wrapperData.height;

            this.scroller.width = scrollData.width;
            this.scroller.height = scrollData.height;

            this.maxMoveY = (this.scroller.height - this.wrapper.height) * -1;
            this.scrollbarLength = this.wrapper.height / this.scroller.height  * 100;
        }
    },
    mounted (){
        this.refresh();
        this.scroller.x = 0;
        this.scroller.y = 0;
    }
}
</script>

<style scoped>
    .scroller-wrapper {
        position: relative;
        width: 100%;
        height: 100%;
        overflow: hidden;
    }

    .scroller {
        position: absolute;
        top: 0px;
        left: 0px;
        width: 100%;
        min-height: 100%;
    }

    .scrollbar-wrapper {
        position: absolute;
        top: 50%;
        right: 5px;
        transform: translateY(-50%);
        height: 98%;
        width: 10px;
        overflow: hidden;
        background: #DED9D9;
    }

    .scrollbar-wrapper, .scrollbar  {
          border-radius: 10px;
    }

    .scrollbar {
        width: 100%;
        height: 100%;
        background: #29A5F7;
    }
</style>