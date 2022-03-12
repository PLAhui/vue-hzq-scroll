<template>
  <div class="main">
    <div class="scroll-container">
      <div v-for="(item,index) in colorList"  :key="index" class="scroll-item" :style="styleFormatter(index - 1)" ref="scrollItem" > {{ index }}</div>
    </div>
    <div class="controller">
      <span>滚动速度：</span><input v-model="speed"/>
      <span>单步停顿时间：</span><input v-model="stopTime"/>
      <button class="btn" @click="changeStatus">{{ status ? "暂停滚动" : "开始滚动" }}</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      colorList: [ "gray", "antiquewhite", "aquamarine", "cadetblue",  "darkgoldenrod","mediumaquamarine","violet", "royalblue",],
      stopTime:200,//单步暂停时间(秒)
      height: 144,// 每个元素的高度
      speed: 1000,// 滚动速度
      isScrolling: false,
      status:false,//滚动状态
    };
  },
  mounted() {
    this.initListener();
  },
  methods: {
    //改变滚动状态
    changeStatus(){
      this.status = !this.status;
      if(this.status){
        this.toggleClick();
      }else{
        this.toggleClick();
      }
    },
    /**
     * 启动滚动&暂停滚动
     */
    toggleClick() {
      if (this.isScrolling) {
        this.isScrolling = false;
        this.pause();
      } else {
        this.isScrolling = true;
        this.scroll();
      }
    },
 
    // 根据索引计算当前元素所在位置
    styleFormatter(index) {
      return {backgroundColor: this.colorList[index], top: `${index * this.height}px`,};
    },

    // 控制滚动开始
    scroll() {
      const elementList = this.$refs.scrollItem;
      elementList.forEach((ele) => {
        ele.style.transitionDuration = `${ (ele.offsetTop + this.height) / this.speed}s`;ele.style.top = `-${this.height}px`;
      });
    },

    // 暂停滚动
    pause() {
      const elementList = this.$refs.scrollItem;
      elementList.forEach((ele) => {const styles = getComputedStyle(ele, null);ele.style.transitionDuration = "0s"; ele.style.top = `${styles.top}`;});
    },

    // 初始化listener，监听滚动动画
    // 当元素完全滚动至显示范围外时，则将该元素插入队尾
    initListener() {
      const elementList = this.$refs.scrollItem;
      var that = this;
      elementList.forEach((ele) => {
        // 当动画结束后会触发transitionend事件
        ele.addEventListener("transitionend", () => {
          // 计算队尾位置
          const lastestElementPosition = this.lastestElementPosition();
          ele.style.transitionDuration = `0s`;
          ele.style.top = `${lastestElementPosition + this.height}px`;
  
          //渲染完毕之后开始滚动
          this.$nextTick(() => {
            ele.style.transitionDuration = `${ (ele.offsetTop + 144) / this.speed }s`;
            ele.style.top = `-${this.height}px`;
            that.toggleClick();
            setTimeout(() => {that.toggleClick();}, that.stopTime);
          });
        });
      });
    },

    // 计算最后一个元素的位置
    lastestElementPosition() {
      const elementList = this.$refs.scrollItem;
      return Math.max(...elementList.map((ele) => { const styles = getComputedStyle(ele, null); return parseFloat(styles.top);})
      );
    },
  },
};
</script>

<style scoped>
.main {
  align-items: center;
  display: flex;
  flex-direction: column;
}
.scroll-container {
  width: 50%;
  height: 800px;
  display: flex;
  flex-direction: row;
  position: relative;
  overflow: auto;
}

.scroll-item {
  display: flex;
  flex-shrink: 0;
  width: 100%;
  height: 144px;
  position: absolute;
  transition-property: top;
  transition-timing-function: linear;
}

.controller{
  margin: 20px;
  padding: 20px;
  font-size: 34px;
  display: grid;
}
input{
    font-size: 34px;
  }
.btn {
  font-size: 36px;
  height: 50px;
}
</style>
