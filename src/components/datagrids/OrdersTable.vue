<template>
  <div>
    <modal-box
      :is-active="isModalActive"
      :trash-object-name="trashObject ? trashObject.name : null"
      @confirm="trashConfirm"
      @cancel="trashCancel"
    />
    <b-table
      :checked-rows.sync="checkedRows"
      :paginated="paginated"
      :per-page="perPage"
      :data="orders"
      default-sort="name"
      striped
      hoverable
    >
      <b-table-column
        v-slot="props"
        label="ID"
        field="id"
        sortable
      >
        {{ props.row.id }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Total Cost"
        field="cost"
        sortable
      >
        {{ props.row.cost }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Product "
        field="product_id"
        sortable
      >
        {{
          props.row.product
            ? "[" + props.row.product_id + "] " + props.row.product.name
            : props.row.product_id + ":" + props.row.product.name
        }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Product Quantity"
        field="quantity"
        sortable
      >
        {{ props.row.quantity }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Customer"
        field="userId"
        sortable
      >
        {{
          props.row.buyer
            ? "[" + props.row.buyer.id + "] " + props.row.buyer.firstName
            : props.row.product_id
        }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Seller"
        field="sellerId"
        sortable
      >
        {{ props.row.seller ? "[" + props.row.seller.id + "] " + props.row.seller.firstName : props.row.product_id }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Payment Status"
        field="payment_status"
        sortable
      >
        {{ props.row.payment_status > 0 ? " Paid" : "Not Paid" }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Dispatch Status"
        field="dispatch_status"
        sortable
      >
        {{ props.row.dispatch_status > 0 ? " Dispatched" : "Not Dispatched" }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        label="Delivery Status"
        field="delivery_status"
        sortable
      >
        {{ props.row.delivery_status > 0 ? " Delivered" : "Not Delivered" }}
      </b-table-column>
      <b-table-column
        v-slot="props"
        custom-key="actions"
        cell-class="is-actions-cell"
      >
        <div
          v-if="userRole == 'Super-Admin' || userRole == 'Seller'"
          class="buttons is-right no-wrap"
        >
          <b-button
            class="button is-small"
            @click="editOrder(props.row)"
          >
            <b-icon
              icon="account-edit"
              size="is-small"
            />
          </b-button>
          .<b-button
            type=""
            size="is-small"
            @click="deleteItem(props.row)"
          >
            <b-icon
              icon="trash-can"
              size="is-small"
            />
          </b-button>
        </div>
      </b-table-column>

      <section
        slot="empty"
        class="section"
      >
        <div class="content has-text-grey has-text-centered">
          <p>
            <b-icon
              icon="emoticon-sad"
              size="is-large"
            />
          </p>
          <p>Nothing's here&hellip;</p>
        </div>
      </section>
    </b-table>
  </div>
</template>

<script>
import { defineComponent } from '@vue/composition-api'
import { mapState } from 'vuex'
import ModalBox from '@/components/BaseModalBox.vue'

export default defineComponent({
  name: 'OrdersTable',
  components: { ModalBox },
  props: {
    checkable: Boolean,
    isEmpty: Boolean,
    perPage: {
      type: Number,
      default: 10
    }
  },
  data () {
    return {
      userRole: this.$store.state.authentication.role,
      checkedRows: [],
      isModalActive: false,
      trashObject: null
    }
  },

  computed: {
    paginated () {
      return this.$store.state.orders.orders.length > this.perPage
    },

    ...mapState({
      orders: (state) => state.orders.orders
    })
  },
  created () {
    this.getOrders()
  },
  methods: {
    editOrder (row) {
      this.$router.push({
        name: 'order.edit',
        params: {
          id: row.id,
          productId: row.product.id,
          productName: row.product.name,
          sellerId: row.seller_id,
          buyerId: row.buyer_id,
          cost: row.cost,
          sellerName: row.seller.firstName + ' ' + row.seller.lastName
        }
      })
    },
    getOrders () {
      const role = this.$store.state.authentication.role
      const userId = this.$store.state.authentication.userId
      if (role === 'Super-Admin') {
        this.$store.dispatch('orders/getAllOrders')
      } else if (role === 'Seller') {
        this.$store.dispatch('orders/getSellersOrders', userId)
      } else if (role === 'Buyer') {
        this.$store.dispatch('orders/getBuyersOrders', userId)
      }
    },

    trashModalOpen (obj) {
      this.trashObject = obj
      this.isModalActive = true
    },
    trashConfirm (id) {
      this.isModalActive = false
      this.$store.dispatch('orders/deleteOrder', id)
      this.$buefy.snackbar.open({
        message: 'Confirmed',
        queue: false
      })
    },
    trashCancel () {
      this.isModalActive = false
    },
    deleteItem (obj) {
      this.$store.dispatch('orders/deleteOrder', obj.id)
      this.$buefy.snackbar.open({
        message: 'Deleted order ' + obj.name,
        queue: true
      })
      this.getOrders()
    }
  }
})
</script>
