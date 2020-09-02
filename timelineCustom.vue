
<!--
  reverse：翻转,默认为false
  data:数据
  title:标题字段 必填
  main:看是图片列表还是文字列表，如果是图片设置image为true, example :main="{field:'image',image:true}";样式设置:style:{'font-size':'16px','color':'#333'}
  colorCustom: 自定义节点颜色数组  时间线每个节点的原点的颜色
  fold:是否可折叠 默认为false
  maxWidth:最大宽度 单位用rem
  more:步长 Number类型 默认全部展示
  @breakUp:点击拆时的回调方法
  @detailsToJump：点击图片想要跳入详情时的回调方法
  @dataSupplement: 补充数据的回调方法
  也可用插槽: 优先度大于main 与main字段互斥  建议选其一即可
  <template slot-scope="dat">
    <span @click="handelDat(dat)">123</span>
  </template>
  
  <timeline-custom :reverse="reverseState"
                    :data="data"
                    :title="['operator','time']"
                    :colorCustom="['#00bcd4','#4CAF50','#e91e63','#ffc107']"
                    :fold="true"
                    maxWidth="15rem"
                    :main="{field:'image',image:true}"
                    :more="Number(more)"
                    @dataSupplement="dataSupplement"
                    @breakUp="breakUp"
                    @detailsToJump="details">
  </timeline-custom>
-->
<template>
  <div class="timeline-custom">
    <div class="timeline-main"
         v-if="data && data.length>0">

      <!-- 优化视觉体验 -->
      <div class="added-block"></div>

      <!-- 主体 -->
      <!-- 只显示index小于moreNum的子项 -->
      <div v-for="(dat,ind) in data.filter((item,ind)=>ind<moreNum)"
           :key="ind"
           class="single-chunk"
           :style="(ind>switchInd)&&switState?'display:none':''">
        <div class="single-dot-outer"
             :style="switchInd===ind&&switState?'transform: scale(1.5);':''"
             @click="foldFuc(ind)"
             title="折叠">
          <div :style="{backgroundColor:colorCustom?colorCustom[ind%colorCustom.length]:''}"
               class="single-dot">
          </div>
        </div>

        <div>
          <div class="title">
            <!-- 看title传入的是哪种类型  最初设计的初衷是用作一个高可配的插件 现在可要 可不要 -->
            <div v-if="typeof title === 'object'">
              <span v-for="tit in title"
                    :key="tit">{{dat[tit]}}</span>
            </div>
            <div v-else>
              <span>{{dat[title]}}</span>
            </div>
            <!-- 拆分按钮 -->
            <div style="position:relative;">
              <span class="iconfont icon-chaibao excrete-class"
                    @click.stop="breakUp(dat,[dat[main.field]])"></span>
            </div>
          </div>

          <!-- 内容 -->
          <div v-if="!slotState"
               class="main"
               :style="main.image?'overflow: auto;width: '+maxWidth:''"
               @click="detailsToJump(dat)">
            <!-- 如果是图片类型的数据 -->
            <div v-if="main.image"
                 class="clear">
              <div v-if="Array.isArray(dat[main.field])"
                   class="clear"
                   style="display: inline-flex;">
                <div v-for="(url,ind) in dat[main.field]"
                     :key="ind"
                     class="item-img">
                  <img class="material-img"
                       :src="url">
                  <span class="iconfont icon-chaibao excrete-class"
                        @click.stop="breakUp(dat,[url],ind)"></span>
                </div>
              </div>
              <div v-else
                   class="item-img">
                <img class="material-img"
                     :src="dat[main.field]">
                <span class="iconfont icon-chaibao excrete-class"
                      @click.stop="breakUp(dat,[dat[main.field]])"></span>
              </div>
            </div>
            <!-- 如果是文字类型的数据 -->
            <div v-else
                 class="clear">
              <div class="item-text">
                <span :style="main.style">{{dat[main.field]}}</span>
              </div>
            </div>
          </div>
          <!-- 如果用户使用slot -->
          <slot v-if="slotState"
                :data="dat"
                @click="detailsToJump(dat)"></slot>
        </div>

      </div>
      <!-- 查看更多按钮 -->
      <div v-if="more"
           style="font-size:0;padding-left:40px;">
        <button class="button-text"
                @click="showMore"><span>查看更多</span></button>
      </div>
    </div>

    <span style="color:#aaa;font-size:14px"
          v-else>暂无数据</span>
  </div>
</template>
<script>
export default {
  name: 'timelineCustom',
  props: {
    data: {
      type: Array,
      default: () => [] // [{time,image,operator},{]}]
    },
    reverse: {
      type: Boolean,
      default: false
    },
    title: {
      type: [String, Array]
    },
    main: {
      type: Object,
      default: () => ({})
    },
    maxWidth: {
      type: String,
      default: '5rem'
    },
    colorCustom: {
      type: Array,
      default: () => ['#333']
    },
    fold: {
      type: Boolean,
      default: false
    },
    more: {
      type: Number,
      default: Infinity
    }
  },
  mounted () {
    // 看用户用的是哪版的slot
    if (this.$scopedSlots.default) this.slotState = this.$scopedSlots.default()[0].tag || this.$scopedSlots.default()[0].text || this.$slots.default()[0].tag || this.$slots.default()[0].text
  },
  data () {
    return {
      moreNum: null, // 显示的数量
      switchInd: '', // 折叠的子项的index
      switState: false, // 控制是否折叠
      slotState: false, // 控制是否显示slot的状态
      i: 1 // 每次点击加载更多时自动++的变量
    }
  },
  methods: {
    foldFuc (index) {
      // 也是优化用户体验的 不必深入
      if (this.fold) {
        if (this.switchInd !== index) {
          this.switState = true
        } else {
          this.switState = !this.switState
        }
        this.switchInd = index
      }
    },
    breakUp (dat, formatArr, ind) { // 有的会传ind参数 有的不会 看用户是点击整行删除还是点击一行中的其中一个删除  都会传入父组件
      // dat是子项的数据   itemArr 是子项里images[]的数据 ind是其再这一行中的序号
      this.$emit('breakUp', dat, formatArr, ind)
    },
    detailsToJump (dat) {
      this.$emit('detailsToJump', dat)
    },
    showMore () {
      this.i++
      if (this.moreNum >= this.data.length) {
        this.$emit('dataSupplement')
      }
    }
  },
  watch: {
    more: {
      handler (val) {
        this.i = 1
        // 优化用户体验
        this.moreNum = Number(val) === 0 ? 0 : (val || Infinity) * this.i
      },
      immediate: true
    },
    i: {
      handler (val) {
        this.moreNum = this.more * val
      },
      immediate: true
    },
    reverse () {
      this.data = this.data.reverse()
    }
  }
}
</script>
<style scoped lang='scss'>
.timeline-custom {
  .timeline-main {
    display: flex;
    margin: 0 auto;
    display: inline-block;
  }
}
.added-block {
  padding: 0.2rem;
  border-left: 2px solid rgb(224, 224, 224);
}
.single-chunk {
  box-sizing: border-box;
  padding: 0.2rem 0.4rem;
  border-left: 2px solid rgb(224, 224, 224);
  position: relative;
  // .excrete-classes {
  //   position: absolute;
  //   width: 0.25rem;
  //   top: 50%;
  //   left: 0.075rem;
  //   transform: translateY(-50%);
  //   opacity: 0;
  //   cursor: pointer;
  // }
  // &:hover .excrete-classes {
  //   opacity: 1;
  // }
  .single-dot-outer {
    width: 0.2rem;
    height: 0.2rem;
    border-radius: 50%;
    background: #eee;
    position: absolute;
    top: 0px;
    left: -0.11rem;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2;
    cursor: pointer;
    box-shadow: 0px 2px 1px -1px rgba(0, 0, 0, 0.2),
      0px 1px 1px 0px rgba(0, 0, 0, 0.14), 0px 1px 3px 0px rgba(0, 0, 0, 0.12);
    .single-dot {
      width: 0.15rem;
      height: 0.15rem;
      border-radius: 50%;
    }
  }
  .clear::after {
    content: "";
    display: block;
    clear: both;
  }
  .title {
    font-size: 0.14rem;
    font-weight: 700;
    color: #657180;
    transform: translateY(-0.18rem);
    display: flex;
    & > div {
      span {
        margin-right: 0.05rem;
      }
    }
    .excrete-class {
      width: 18px;
      position: absolute;
      top: 0px;
      right: -20px;
      opacity: 0;
      cursor: pointer;
    }
    &:hover .excrete-class {
      opacity: 1;
    }
  }
  .main {
    cursor: pointer;
    .item-text {
      font-size: 0;
    }
    .item-img {
      width: 2rem;
      height: 1.5rem;
      padding: 5px;
      border: 1px solid #ccc;
      margin-right: 10px;
      background: #fff;
      position: relative;
      .material-img {
        height: 100%;
        width: 100%;
        object-fit: contain;
      }
      .excrete-class {
        width: 18px;
        position: absolute;
        top: 0px;
        right: 0px;
        opacity: 0;
      }
      &:hover .excrete-class {
        opacity: 1;
      }
    }
  }
}
.button-text {
  display: inline-block;
  line-height: 1;
  white-space: nowrap;
  cursor: pointer;
  border: 1px solid #dcdfe6;
  -webkit-appearance: none;
  text-align: center;
  box-sizing: border-box;
  outline: none;
  margin: 0;
  transition: 0.1s;
  font-weight: 500;
  padding: 12px 20px;
  font-size: 14px;
  border-radius: 4px;
  border-color: transparent;
  color: #409eff;
  background: transparent;
  padding-left: 0;
  padding-right: 0;
}
</style>
