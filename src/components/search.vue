<template>
  <!-- 搜索 -->
  <div class="search" :class="{focused: focused}">
    <!-- 搜索框 -->
    <div class="input-wrap" @click="goSearch">
      <input type="text" @confirm="storage()" v-model="key" @input="searchList()" :placeholder="placeholder">
      <span class="cancle" @click.stop="cancleSearch">取消</span>
    </div>
    <!-- 搜索结果 -->
    <div class="content">
      <div class="title">搜索历史<span class="clear" @click="clearHistory"></span></div>
      <div class="history">
        <navigator :url="'/pages/list/main?query=' + item " :key="i" v-for="(item,i) in history">{{item}}</navigator>
      </div>
      <!-- 结果 -->
      <scroll-view v-show="keyList.length" scroll-y class="result">
        <navigator :url="'/pages/goods/main?query=' + item.goods_id " :key="item.goods_id" v-for="item in keyList">{{item.goods_name}}</navigator>
      </scroll-view>
    </div>
  </div>
</template>
<script>
import request from '@/utils/request'
  export default {
    data () {
      return {
        focused: false,
        placeholder: '',
        // 输入的关键字
        key: '',
        keyList: [],
        history: []
      }
    },
    methods: {
      goSearch (ev) {
        this.focused = true
        this.placeholder = '请输入您要搜索的内容'
        // 触发父组件自定义事件
        this.$emit('search', mpvue.getSystemInfoSync().windowHeight + 'px')
        // 隐藏tab栏
        mpvue.hideTabBar()
      },
      // 取消按钮
      cancleSearch () {
        this.focused = false
        this.placeholder = ''
        this.key = ''
        this.keyList = []
        // 触发父组件自定义事件
        this.$emit('search', 'auto')
        // 显示 tab栏
        mpvue.showTabBar()
      },
      // 监听 input事件
      async searchList () {
        if (!this.key) return this.keyList = []
        const {message} = await request({url: '/api/public/v1/goods/qsearch', data: {query: this.key}})
        this.keyList = message
        console.log(message)
      },
      // 储存本地数据
      storage () {
        this.history = mpvue.getStorageSync('history') || []
        this.history.push(this.key)
        // 去重
        this.history = [...new Set(this.history)]
        mpvue.setStorageSync('history', this.history)
        mpvue.navigateTo({url: '/pages/list/main?query=' + this.key})
      },
      // 清除历史记录
      clearHistory () {
        mpvue.removeStorageSync('history')
        this.history = []
      }
    },
    created () {
      this.history = mpvue.getStorageSync('history') || []
    }
  }

</script>
<style lang="less" scoped>
  .search {
    display: flex;
    flex-direction: column;
    
    // 搜索框
    .input-wrap {
      display: flex;
      height: 100rpx;
      padding: 20rpx 30rpx;
      background-color: #ea4451;
      box-sizing: border-box;
      position: relative;

      &::before,
      &::after {
        height: 44rpx;
        line-height: 1;
        background-image: url(https://ugo2.oss-cn-hangzhou.aliyuncs.com/images/icon_search%402x.png);
        background-size: 32rpx;
        background-position: 6rpx center;
        background-repeat: no-repeat;

        position: absolute;
        top: 28rpx;
        z-index: 9;
      }

      &::before {
        content: '搜索';
        display: block;

        width: 100rpx;
        padding: 11rpx 0 10rpx 44rpx;
        box-sizing: border-box;
        color: #666;
        font-size: 24rpx;
        left: 325rpx;
      }

      &::after {
        display: none;
        content: '';
        width: 44rpx;
        left: 40rpx;
      }

      input {
        flex: 1;
        height: 60rpx;
        padding: 0 20rpx 0 64rpx;
        background-color: #fff;
        border-radius: 8rpx;
        font-size: 24rpx;
        color: #666;
      }

      span.cancle {
        display: none;
        width: 80rpx;
        text-align: right;
        line-height: 60rpx;
        font-size: 27rpx;
        color: #666;
      }
    }

    // 搜索结果
    .content {
      display: none;
      flex: 1;
      padding: 27rpx;
      background-color: #fff;
      position: relative;

      .title {
        font-size: 27rpx;
        line-height: 1;
        color: #333;
      }

      .clear {
        display: block;
        width: 27rpx;
        height: 27rpx;
        float: right;
        background-image: url(https://ugo2.oss-cn-hangzhou.aliyuncs.com/images/clear.png);
        background-size: cover;
      }

      .history {
        padding-top: 30rpx;

        navigator {
          display: inline-block;
          line-height: 1;
          padding: 15rpx 20rpx 12rpx;
          background-color: #ddd;
          font-size: 24rpx;
          margin-right: 20rpx;
          margin-bottom: 15rpx;
          color: #333;
        }
      }

      .result {
        // display: none;
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        background-color: #fff;

        navigator {
          line-height: 1;
          padding: 20rpx 30rpx;
          font-size: 24rpx;
          color: #666;
          border-bottom: 1px solid #eee;

          &:last-child {
            border-bottom: none;
          }
        }
      }
    }
    
    // 获得焦点状态
    &.focused {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      z-index: 9;

      .input-wrap {
        background-color: #eee;

        &::before {
          display: none;
        }

        &::after {
          display: block;
        }
      }

      span.cancle {
        display: block;
      }

      .content {
        display: block;
      }
    }
  }
</style>