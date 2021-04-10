<template>
  <div class="Orders">
    <Header/>

    <div class="Orders__container">
      <div class="Orders__title">Заказы</div>

      <OrdersFilters :data="filters"/>

      <div v-custom-loading="loading">
        <OrderCardInfo
          v-for="order in orders"
          :key="order.id"
          :order="order"
          @click="$router.push({ name: 'OrdersView', params: { id: order.id } })"
        />
      </div>
    </div>

    <Pagination
      :current-page="+filters.page"
      :page-size="perPage"
      :total="count"
      @current-change="filters.page = $event"
    />
  </div>
</template>

<script>
import { useStore } from 'vuex'
import { useRoute } from 'vue-router'
import { onMounted, ref, computed, watch, onUnmounted } from 'vue'
import useRouterQueryBinder from '@shared/util/routerQueryBinder'
import { getErrorHandler } from '@shared/util'
import Header from '@/pages/desktop/Header'
import OrdersFilters from '@/components/desktop/orders/OrdersFilters'
import OrderCardInfo from '@/components/desktop/orders/OrderCardInfo'
import Pagination from '@shared/components/desktop/Pagination'

export default {
  name: 'Orders',

  components: {
    Pagination,
    OrderCardInfo,
    OrdersFilters,
    Header,
  },

  setup () {
    const store = useStore()
    const route = useRoute()

    const loading = ref(false)
    const perPage = 15
    const {
      query: filters,
      watcher: filtersWatcher,
    } = useRouterQueryBinder({
      page: 1,
      brand: '',
      model: null,
      year: null,
      showWithMyApplication: false,
      showMyOrders: false,
    })

    const orders = computed(() => store.getters['orders/data'])
    const count = computed(() => store.getters['orders/count'])

    const fetchData = async () => {
      loading.value = true

      await store.dispatch('orders/getItems', filters)
        .catch(getErrorHandler('Failed loading orders'))

      loading.value = false
    }

    filtersWatcher.add(() => {
      filters.page = 1
    }, { exclude: ['page'] })
    filtersWatcher.add((value) => {
      if (!('page' in value)) store.dispatch('orders/clearItems')
      fetchData()
    })

    watch(() => filters.brand, () => {
      filters.model = null
      filters.year = null
    })
    watch(() => filters.model, () => filters.year = null)

    onMounted(() => {
      if (!Object.keys(route.query).length > 0) fetchData()
      store.dispatch('timer/start')
    })
    onUnmounted(() => {
      store.dispatch('timer/halt')
    })

    return {
      loading,
      orders,
      filters,
      perPage,
      count,
    }
  },
}
</script>

<style lang="scss">
@import "~@shared/styles/_var.scss";

.Orders {
  background: $BgOrders;
  padding-bottom: 100px;

  &__container {
    max-width: 1280px;
    margin: 0 auto;
    padding: 0 85px;

    @media screen and (max-width: 1180px) {
      padding: 0 15px;
    }
  }

  &__title{
    font-weight: 600;
    font-size: 42px;
    line-height: 51px;
    margin: 59px 0 40px;
  }

  &__header {
    box-shadow: 0 4px 4px rgba(0, 0, 0, 0.04);
    padding-bottom: 10px;
    background: $white;

    &-wrap {
      &-nav {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        position: relative;
        padding: 0 0 0 0;
      }

      &-close {
        margin-bottom: 0;
      }
    }

    &-title {
      font-size: 32px;
      font-weight: 500;
      margin: 9px 0 0 0;
    }
  }

  &__btn-close {
    padding: 3px 0 0 0 !important;
    margin: 0 0 0 0;
  }

  &__filter {
    &-item {
      display: flex;
      flex-direction: row;
      justify-content: flex-end;
      padding: 29px 0 23px 0;

      &-title {
        margin: 0 15px 0 0;
        font-size: 16px;
      }
    }

    &-btn {
      padding: 0 !important;
    }
  }

  .el-dialog {
    border-radius: 30px;

    &__header {
      text-align: center;
      padding: 30px 16px 0 16px;
    }

    &__title {
      color: $dark;
      font-size: 20px;
      margin: 0 10px 0 0;
    }

    &__close {
      color: $dark !important;
      font-size: 20px;
      font-weight: 700;
    }

    &__headerbtn {
      top: 35px;
      color: $dark;
    }

    &__body {
      padding: 40px 16px 20px 16px;
    }

    .el-form-item {
      &__label {
        font-weight: 500;
        font-size: 20px;
        line-height: 24px;
        padding: 0 0 10px 16px !important;
        color: $dark;
      }

      .el-select {
        display: block;

        &__caret {
          color: $dark;
          font-weight: 700;
        }

        .el-input {
          &__inner {
            border: 0;
            color: $labelText;
            background: $placeholder;
            border-radius: 16px;
            height: 48px;
            padding: 0 16px;
          }

          &__suffix {
            right: 15px;
          }
        }
      }
    }
  }
}
</style>
