<template>
    <div ref="list" :style="{ height }" class="infinite-list-container" @scroll="scrollEvent($event)">
        <div ref="phantom" class="infinite-list-phantom"></div>
        <div ref="content" class="infinite-list">
            <div class="infinite-list-item" ref="items" :id="item.id" :key="item.id" v-for="item in visibleData">
                <slot ref="slot" :item="item"></slot>
            </div>
        </div>
    </div>
</template>
  
  
<script>

export default {
    name: 'VirtualList',
    props: {
        //所有列表数据
        listData: {
            type: Array,
            default: () => []
        },
        //预估高度
        estimatedItemSize: {
            type: Number,
            default: 100,
        },
        //容器高度 100px or 50vh
        height: {
            type: String,
            default: '100%'
        }
    },
    computed: {
        visibleCount() {
            return Math.ceil(this.screenHeight / this.estimatedItemSize);
        },
        //获取真实显示列表数据
        visibleData() {
            return this.listData.slice(this.start, Math.min(this.end, this.listData.length));
        }
    },
    created() {
        this.initPositions();
        window.vm = this;
    },
    mounted() {
        this.screenHeight = this.$el.clientHeight;
        this.start = 0;
        this.end = this.start + this.visibleCount;
    },
    updated() {
        this.$nextTick(function () {
            if (!this.$refs.items || !this.$refs.items.length) {
                return;
            }
            //获取真实元素大小，修改对应的尺寸缓存
            this.updateItemsSize();
            //更新列表总高度
            let height = this.positions[this.positions.length - 1].bottom;
            this.$refs.phantom.style.height = height + 'px'
        })
    },
    data() {
        return {
            //可视区域高度
            screenHeight: 0,
            //起始索引
            start: 0,
            //结束索引
            end: 0,
            // 记录位置
            positions: []
        };
    },
    methods: {
        initPositions() {
            this.positions = this.listData.map((d, index) => ({
                index,
                height: this.estimatedItemSize,
                bottom: (index + 1) * this.estimatedItemSize
            })
            );
        },
        //获取列表起始索引
        getStartIndex(scrollTop = 0) {
            let item = this.positions.find(i => i && i.bottom > scrollTop);
            return item.index;
        },

        //获取列表项的当前尺寸
        updateItemsSize() {
            let nodes = this.$refs.items;
            nodes.forEach((node) => {
                let rect = node.getBoundingClientRect();
                let height = rect.height;
                let index = node.id
                let oldHeight = this.positions[index].height;
                let dValue = oldHeight - height;
                //存在差值
                if (dValue) {
                    this.positions[index].bottom = this.positions[index].bottom - dValue;
                    this.positions[index].height = height;
                }

            })
        },
        //获取当前的偏移量
        setStartOffset() {
            this.$refs.content.style.transform = `translate3d(0,${this.startOffset}px,0)`
        },
        //滚动事件
        scrollEvent() {
            //当前滚动位置
            let scrollTop = this.$refs.list.scrollTop;
            //此时的开始索引
            this.start = this.getStartIndex(scrollTop);
            //此时的结束索引
            this.end = this.start + this.visibleCount;

            if (this.start >= 1) {
                this.startOffset = this.positions[this.start - 1].bottom
            } else {
                this.startOffset = 0;
            }
            //此时的偏移量
            this.setStartOffset();
        }
    }
};
</script>
  
  
<style scoped>
.infinite-list-container {
    overflow: auto;
    position: relative;
    -webkit-overflow-scrolling: touch;
}

.infinite-list-phantom {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    z-index: -1;
}

.infinite-list {
    left: 0;
    right: 0;
    top: 0;
    position: absolute;
}

.infinite-list-item {
    padding: 5px;
    color: #555;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
    /* height:200px; */
}
</style>