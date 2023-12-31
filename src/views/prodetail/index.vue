<template>
  <div class="prodetail">
    <van-nav-bar fixed title="商品详情页" left-arrow @click-left="$router.go(-1)" />

    <!-- 顶部商品轮播图 -->
    <van-swipe :autoplay="3000" @change="onChange">
      <van-swipe-item v-for="image in detail.goods_images" :key="image.file_id">
        <img :src="image.preview_url" />
      </van-swipe-item>

      <template #indicator>
        <div class="custom-indicator">{{ current + 1 }} / {{ images.length }}</div>
      </template>
    </van-swipe>

    <!-- 商品说明 -->
    <div class="info">
      <div class="title">
        <div class="price">
          <span class="now">￥{{detail.goods_price_min}}</span>
          <span class="oldprice">￥{{detail.goods_price_min}}</span>
        </div>
        <div class="sellcount">已售{{detail.goods_sales}}件</div>
      </div>
      <div class="msg text-ellipsis-2">
         {{detail.goods_name }}
      </div>

      <div class="service">
        <div class="left-words">
          <span><van-icon name="passed" />七天无理由退货</span>
          <span><van-icon name="passed" />48小时发货</span>
        </div>
        <div class="right-icon">
          <van-icon name="arrow" />
        </div>
      </div>
    </div>

    <!-- 商品评价 -->
    <div class="comment">
      <div class="comment-title">
        <div class="left">商品评价 ({{commentTotal}}条)</div>
        <div class="right">查看更多 <van-icon name="arrow" /> </div>
      </div>
      <div class="comment-list">
        <div class="comment-item" v-for="item in commentList" :key="item.comment_id">
          <div class="top">
            <img :src="item.user.avatar_url || defaultAvatar" alt="">
            <div class="name">{{item.user.nick_name}}</div>
            <van-rate :size="16" :value="item.score / 2 " color="#ffd21e" void-icon="star" void-color="#eee"/>
          </div>
          <div class="content">
            {{item.content}}
          </div>
          <div class="time">
            {{item.create_time}}
          </div>
        </div>
      </div>
    </div>

    <!-- 商品描述 -->
    <div class="desc" v-html='detail.content'>
    </div>

    <!-- 底部 -->
    <div class="footer">
      <div class="icon-home" @click="$router.push('/')">
        <van-icon name="wap-home-o" />
        <span>首页</span>
      </div>
      <div class="icon-cart" @click="$router.push('/cart')">
        <van-icon v-if='$store.getters.token' name="cart-o" :badge='cartTotal' />
        <van-icon v-else name="cart-o" />
        <span>购物车</span>
      </div>
      <div class="btn-add" @click="changeMode('cart')">加入购物车</div>
      <div class="btn-buy" @click="changeMode('buy')">立刻购买</div>
    </div>

    <!-- 加入购物车的弹层 -->
    <van-action-sheet v-model="showPannel" :title="mode === 'cart' ? '加入购物车' : '立刻购买'" >
      <div class="product">
        <div class="product-title">
          <div class="left">
            <img :src="detail.goods_image" alt="">
          </div>
          <div class="right">
            <div class="price">
              <span>¥</span>
              <span class="nowprice">{{detail.goods_price_min}}</span>
            </div>
              <div class="count">
                <span>库存</span>
                <span>{{detail.stock_total}}</span>
              </div>
          </div>
        </div>

        <div class="num-box">
          <span>数量</span>
          <CountBox v-model="value"></CountBox>
        </div>

        <!-- 有库存才显示提交按钮 -->
        <div class="showbtn" v-if="detail.stock_total !== 0">
          <div class="btn" v-if="mode === 'cart'" @click="addCart">加入购物车</div>
          <div class="btn now" v-else @click='settle'>立刻购买</div>
        </div>

        <div class="btn-none" v-else>该商品已抢完</div>
      </div>
    </van-action-sheet>
  </div>
</template>

<script>
import { getGoodsDetail, getComData } from '@/api/product'
import defaultAvatar from '@/assets/default-avatar.png'
import CountBox from '@/components/CountBox.vue'
import { addCart, getCartTocal } from '@/api/cart'
import loginConfirm from '@/mixins/loginConfirm'

export default {
  mixins: [loginConfirm],
  components: { CountBox },
  name: 'ProDetail',
  data () {
    return {
      images: [
        'https://img01.yzcdn.cn/vant/apple-1.jpg',
        'https://img01.yzcdn.cn/vant/apple-2.jpg'
      ],
      current: 0, // 轮播图当前页数
      detail: {}, // 商品的详细数据
      commentList: {}, // 评论列表
      commentTotal: '', // 评论总数
      defaultAvatar, // 默认头像
      showPannel: false, // 加入购物车弹层的显示模式
      mode: '', // 弹层的功能模式
      value: 1, // 弹层的商品数量
      cartTotal: 0 // 购物车角标
    }
  },
  computed: {
    // 获取动态路由传参的商品id
    goodsId () {
      return this.$route.params.id
    }
  },
  methods: {
    onChange (index) {
      this.current = index
    },

    // 获取商品请求的响应的数据
    async getDetail () {
      const {
        data: {
          detail
        }
      } = await getGoodsDetail(this.goodsId)
      this.detail = detail
      this.images = detail.goods_images
    },

    // 获取请求的商品总数量
    async getCartTotal () {
      const res = await getCartTocal()
      this.cartTotal = res.data.cartTotal
    },

    // 获取评论请求的响应数据
    async getComment () {
      const {
        data: {
          list, total
        }
      } = await getComData(this.goodsId, 3)
      this.commentList = list
      this.commentTotal = total
    },

    // 弹层的模式切换
    changeMode (mode) {
      this.mode = mode
      this.showPannel = true
    },

    // 加入购物车
    async addCart () {
      if (this.loginConfirm()) {
        return
      }

      // 有 token加入购物车
      const res = await addCart(this.detail.goods_id, this.value, this.detail.skuList[0].goods_sku_id)
      // 添加成功弹窗
      this.$toast.success('加入购物车成功')
      this.cartTotal = res.data.cartTotal
      // 关闭弹页
      this.showPannel = false
    },

    // 结算
    settle () {
      if (this.loginConfirm()) {
        return
      }

      // 有 token跳转到订单页面
      this.$router.push({
        path: '/pay',
        query: {
          mode: 'buyNow',
          obj: JSON.stringify({
            cartIds: this.totalCheckedId,
            goodsId: this.goodsId,
            goodsNum: this.value,
            goodsSkuId: 0
          })
        }
      })
    }
  },
  created () {
    this.getDetail()
    this.getComment()
    this.getCartTotal()
  }
}
</script>

<style lang="less" scoped>
.prodetail {
  padding-top: 46px;
  ::v-deep .van-icon-arrow-left {
    color: #333;
  }
  img {
    display: block;
    width: 100%;
  }
  .custom-indicator {
    position: absolute;
    right: 10px;
    bottom: 10px;
    padding: 5px 10px;
    font-size: 12px;
    background: rgba(0, 0, 0, 0.1);
    border-radius: 15px;
  }
  .desc {
    width: 100%;
    overflow: scroll;
    ::v-deep img {
      display: block;
      width: 100%!important;
    }
  }
  .info {
    padding: 10px;
  }
  .title {
    display: flex;
    justify-content: space-between;
    .now {
      color: #fa2209;
      font-size: 20px;
    }
    .oldprice {
      color: #959595;
      font-size: 16px;
      text-decoration: line-through;
      margin-left: 5px;
    }
    .sellcount {
      color: #959595;
      font-size: 16px;
      position: relative;
      top: 4px;
    }
  }
  .msg {
    font-size: 16px;
    line-height: 24px;
    margin-top: 5px;
  }
  .service {
    display: flex;
    justify-content: space-between;
    line-height: 40px;
    margin-top: 10px;
    font-size: 16px;
    background-color: #fafafa;
    .left-words {
      span {
        margin-right: 10px;
      }
      .van-icon {
        margin-right: 4px;
        color: #fa2209;
      }
    }
  }

  .comment {
    padding: 10px;
  }
  .comment-title {
    display: flex;
    justify-content: space-between;
    .right {
      color: #959595;
    }
  }

  .comment-item {
    font-size: 16px;
    line-height: 30px;
    .top {
      height: 30px;
      display: flex;
      align-items: center;
      margin-top: 20px;
      img {
        width: 20px;
        height: 20px;
      }
      .name {
        margin: 0 10px;
      }
    }
    .time {
      color: #999;
    }
  }

  .footer {
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 55px;
    background-color: #fff;
    border-top: 1px solid #ccc;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    .icon-home, .icon-cart {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      font-size: 14px;
      .van-icon {
        font-size: 24px;
      }
    }
    .btn-add,
    .btn-buy {
      height: 36px;
      line-height: 36px;
      width: 120px;
      border-radius: 18px;
      background-color: #ffa900;
      text-align: center;
      color: #fff;
      font-size: 14px;
    }
    .btn-buy {
      background-color: #fe5630;
    }
  }
}

.tips {
  padding: 10px;
}

.product {
  .product-title {
    display: flex;
    .left {
      img {
        width: 90px;
        height: 90px;
      }
      margin: 10px;
    }
    .right {
      flex: 1;
      padding: 10px;
      .price {
        font-size: 14px;
        color: #fe560a;
        .nowprice {
          font-size: 24px;
          margin: 0 5px;
        }
      }
    }
  }

  .num-box {
    display: flex;
    justify-content: space-between;
    padding: 10px;
    align-items: center;
  }

  .btn, .btn-none {
    height: 40px;
    line-height: 40px;
    margin: 20px;
    border-radius: 20px;
    text-align: center;
    color: rgb(255, 255, 255);
    background-color: rgb(255, 148, 2);
  }
  .btn.now {
    background-color: #fe5630;
  }
  .btn-none {
    background-color: #cccccc;
  }
}
</style>
