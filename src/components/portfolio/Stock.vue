<template>
  <drag class="col-sm-6 col-md-4" :transferData="{
    type: 'portfolio',
    stock: stock,
    quantity: quantity,
  }"
  :draggable="(quantity<=0 || quantity>stock.quantity) ? false : true"
  @dragend="quantity=0"
  >
    <div class="panel panel-success">
      <div class="panel-heading">
        <h3 class="panel-title">
          <router-link :to="{
            path: '/stock-details/' + stock.id,
          }">
            {{stock.name}}
            <small>{{ stock.quantity }} @ {{stock.price | currency }}</small>
            <span v-html="getStockMargin(stock.id, stock.quantity, stock.price)"></span>
          </router-link>
        </h3>
      </div>
      
      <div class="panel-body">
        <div class="pull-left">
          <input type="number" class="form-control" placeholder="Quantity" v-model="quantity" min="0" />
        </div>
        <div class="pull-right">
          <button class="btn btn-success" @click="sellStock" :disabled="quantity<=0 || quantity > stock.quantity">Sell</button>
        </div>
      </div>
    </div>
  </drag>
</template>

<script>
  import {mapGetters, mapActions} from 'vuex';
  import { Drag } from '../../library/vue-drag-drop.browser';
  export default {
    props: ['stock'],
    data: () => {
      return {
        quantity: 0,
      }
    },
    components: {
      Drag,
    },
    computed: {
      ...mapGetters({
        stocks: 'stocks',
      })
    },
    mounted(){
    },
    methods: {
      ...mapActions({ sellStockAction: 'sellStock' }),
      sellStock() {
        const order = {
          stockId: this.stock.id,
          stockPrice: this.stock.price,
          quantity: parseInt(this.quantity)
        }

        // use mapActions as alternative to dispatch
        this.sellStockAction(order);
        // which is the same as:
        // this.$store.dispatch('sellStock',order);
        this.quantity = 0;
      },
      getStockMargin(id, quantity, ownedPrice) {
        const currentPrice = this.stocks.find(element => element.id === this.stock.id).price;
        const marginNum = (ownedPrice - currentPrice) / ownedPrice * 100;
        const marginText = marginNum !== 0 ?
                          '<span class="' + (marginNum < 0 ? 'text-danger' : '') + '">' + Number(marginNum).toFixed(2)  + '%</span>' :
                          '';

        return marginText;
      }
    }
  }
</script>