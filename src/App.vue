<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab-wrapper">
      <Tab :tabs="tabs"></Tab>
    </div>
  </div>
</template>

<script>
import qs from 'query-string'
import VHeader from 'components/v-header/v-header'
import Tab from 'components/tab/tab'
import Goods from 'components/goods/goods'
import Ratings from 'components/ratings/ratings'
import Seller from 'components/seller/seller'
import { getSeller } from 'api'
export default {
  name: 'app',
  data() {
    return {
       seller: {
          id: qs.parse(location.search).id
        }
    }
  },
  computed: {
      tabs() {
        return [
          {
            label: '商品',
            comment: Goods,
            data: {
              seller: this.seller
            }
          },
          {
            label: '评论',
            comment: Ratings,
            data: {
              seller: this.seller
            }
          },
          {
            label: '商家',
            comment: Seller,
            data: {
              seller: this.seller
            }
          }
        ]
      }
  },
  methods: {
    _getSeller() {
        getSeller({
          id: this.seller.id
        }).then((seller) => {
        this.seller = seller
          // console.log(this.seller)
     })
    }
  },

  created() {
     this._getSeller()
  },
  mounted() {
    console.log()
  },
  components: {
    VHeader,
    Tab

  }
}
</script>

<style lang="stylus">
#app
  .tab-wrapper
    position fixed
    top 136px
    left 0
    bottom 0
    right  0
</style>
