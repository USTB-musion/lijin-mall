<template>
  <div>
    <nav-header></nav-header>
    <nav-bread>
        <span>Goods</span>
    </nav-bread>
    <div class="accessory-result-page accessory-page">
      <div class="container">
        <div class="filter-nav">
          <span class="sortby">Sort by:</span>
          <a href="javascript:void(0)" class="default cur">Default</a>
          <a href="javascript:void(0)" class="price" @click="sortGoods">
            Price 
            <svg class="icon icon-arrow-short" v-bind:class="{'sort-up':!sortFlag}">
              <use xlink:href="#icon-arrow-short"></use>
            </svg>
          </a>
          <a href="javascript:void(0)" class="filterby stopPop" @click.stop="showFilterPop">Filter by</a>
        </div>
        <div class="accessory-result">
          <!-- filter -->
          <div class="filter stopPop" id="filter" :class="{'filterby-show': filterBy}">
            <dl class="filter-price">
              <dt>Price:</dt>
              <dd><a href="javascript:void(0)" 
                    :class="{'cur': priceChecked == 'all'}" 
                    @click="priceChecked == 'all'">All
                    </a>
              </dd>
              <dd v-for="(price,index) in priceFilter" 
                  @click="priceChecked = index">
                <a href="javascript:void(0)" 
                  @click = "setPriceFilter(index)"
                  :class="{'cur':priceChecked == index}">
                  {{price.startPrice}} - {{price.endPrice}}
                </a>
              </dd>
            </dl>
          </div>

          <!-- search result accessories list -->
          <div class="accessory-list-wrap">
            <div class="accessory-list col-4">
              <ul>
                <li v-for="(item,index) in goodsList">
                  <div class="pic">
                    <a href="#"><img v-lazy="'/static/'+item.productImage" alt=""></a>
                  </div>
                  <div class="main">
                    <div class="name">{{item.productName}}</div>
                    <div class="price">{{item.salePrice}}</div>
                    <div class="btn-area">
                      <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                    </div>
                  </div>
                </li>
              </ul>
              <div v-infinite-scroll="loadMore" 
                   infinite-scroll-disabled="busy" 
                   infinite-scroll-distance="30">
                <img src="./../assets/loading-spinning-bubbles.svg" v-show="loading">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="md-overlay" v-show="overLayFlag" @click.stop="closePop"></div>
    <modal :mdShow="mdShow" @close = "closeModal">
      <p slot="message">
        请先登录,否则无法加入到购物车中
      </p>
      <div slot="btnGroup">
        <a class="btn btn--m" href="javascript:;" @click="mdShow=false">关闭</a>
      </div>
    </modal>
    <modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
        <p slot="message">
          <svg class="icon-status-ok">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
          </svg>
          <span>加入购物车成功!</span>
        </p>
        <div slot="btnGroup">
          <a class="btn btn--m" href="javascript:;" @click="mdShowCart = false">继续购物</a>
          <router-link class="btn btn--m btn--red" href="javascript:;" to="/cart">查看购物车</router-link>
        </div>
    </modal>
    <nav-footer></nav-footer>
  </div>
</footer>
  </div>
</template>
<script>
  import './../assets/css/base.css'
  import './../assets/css/product.css'
  import NavHeader from './../components/NavHeader'
  import NavFooter from './../components/NavFooter'
  import NavBread from './../components/NavBread'
  import Modal from './../components/Modal'
  import axios from 'axios'

  export default {
    data() {
      return {
        goodsList: [],
        priceFilter: [
          {
            startPrice: '0.00',
            endPrice: '500.00'
          },
          {
            startPrice: '100.00',
            endPrice: '500.00'
          },
          {
            startPrice: '500.00',
            endPrice: '1000.00'
          },
          {
            startPrice: '1000.00',
            endPrice: '2000.00'
          }
        ],
        priceChecked: 'all',
        filterBy: false,
        overLayFlag: false,
        sortFlag: true,
        mdShowCart: false,
        page: 1,
        pageSize: 8,
        busy: true,
        loading:false,
        mdShow: false

      }
    },
    components: {
      NavHeader,
      NavFooter,
      NavBread,
      Modal
    },
    mounted() {
      this.getGoodsList();
    },
    methods: {
      getGoodsList(flag) {
        var param = {
          page: this.page,
          pageSize: this.pageSize,
          sort: this.sortFlag?1:-1,
          priceLevel:this.priceChecked
        };
        this.loading = true;
        axios.get('/goods/list',{
          params: param
        }).then((result) => {
          var res = result.data;
          this.loading = false;
          if(res.status == "0"){
            if(flag){
              this.goodsList = this.goodsList.concat(res.result.list);
              if(res.result.count == 0){
                this.busy = true;
              }else{
                this.busy = false;
              }
            }else{
              this.goodsList = res.result.list;
              this.busy = false;
            }
          }else {
            this.goodsList = [];
          }
          
        })
      },
      sortGoods() {
        this.sortFlag = !this.sortFlag;
        this.page = 1;
        this.getGoodsList();
      },
      loadMore() {
        this.busy = true;
        setTimeout(() => {
           this.page++;
           this.getGoodsList(true);
        }, 600);
      },
      showFilterPop() {
        this.filterBy = true;
        this.overLayFlag = true;
      },
      closePop() {
        this.filterBy = false;
        this.overLayFlag = false;
      },
      setPriceFilter(index) {
        this.priceChecked = index;
        this.page = 1;
        this.getGoodsList();
      },
      addCart(productId) {
        axios.post('/goods/addCart',{
          productId: productId
        }).then((res) => {
          var res = res.data;
          if(res.status == 0){
            this.mdShowCart = true;
          }else{
            this.mdShow = true;
          }
        })
      },
      closeModal() {
        this.mdShow = false;
        this.mdShowCart = false;
      }
    }
  }
</script>

