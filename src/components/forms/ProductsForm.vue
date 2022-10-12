<template>
  <div>
    <title-bar :title-stack="titleStack" />
    <hero-bar>
      {{ heroTitle }}
      <router-link
        slot="right"
        :to="heroRouterLinkTo"
        class="button"
      >
        {{ heroRouterLinkLabel }}
      </router-link>
    </hero-bar>
    <section class="section is-main-section">
      <notification class="is-info">
        <div>
          <span><b>Efficient teams are made of 5 or less members.</b>Team work makes the dream work!</span>
        </div>
      </notification>
      <tiles>
        <card-component
          :title="formCardTitle"
          icon="account-edit"
          class="tile is-child"
        >
          <form @submit.prevent="submit">
            <b-field
              label="ID"
              horizontal
            >
              <b-input
                v-model="form.id"
                custom-class="is-static"
                readonly
              />
            </b-field>
            <hr>
            <b-field
              label="Team Avatar"
              horizontal
            >
              <file-picker type="is-info" />
            </b-field>
            <hr>
            <b-field
              label="Name"
              message="Team name"
              horizontal
            >
              <b-input
                v-model="form.name"
                placeholder="e.g. Team One"
                required
              />
            </b-field>
            <b-field
              label="Description"
              message="Team description"
              horizontal
            >
              <b-input
                v-model="form.description"
                placeholder="e.g. This team solely works on Project A"
                required
              />
            </b-field>
            <b-field
              label="Members"
              message="Members of the team"
              horizontal
            >
              <b-input
                v-model="form.members"
                placeholder="e.g. Prudence Wanjau & John Kamau"
                required
              />
            </b-field>
            <b-field
              label="Leader"
              message="Team's captain"
              horizontal
            >
              <b-input
                v-model="form.leader"
                placeholder="e.g. Prudence Wanjau"
                required
              />
            </b-field>
            <b-field
              label="Responsibilities"
              message="Duties of the team"
              horizontal
            >
              <b-input
                v-model="form.responsibilities"
                placeholder="e.g. Devops & SSR"
                required
              />
            </b-field>
            <hr>
            <b-field horizontal>
              <b-button
                type="is-info"
                :loading="isLoading"
                native-type="submit"
              >
                Submit
              </b-button>
              <b-button
                type=""
                :loading="isLoading"
                @click="$router.push('/dashboard/teams')"
              >
                Back
              </b-button>
            </b-field>
          </form>
        </card-component>
        <card-component
          v-if="isProfileExists"
          title="Team Profile"
          icon="account"
          class="tile is-child"
        >
          <user-avatar
            :avatar="form.avatar"
            class="image has-max-width is-aligned-center"
          />
          <hr>
          <b-field label="Name">
            <b-input
              :value="form.name"
              custom-class="is-static"
              readonly
            />
          </b-field>
          <b-field label="Description">
            <b-input
              :value="form.description"
              custom-class="is-static"
              readonly
            />
          </b-field>
          <b-field label="Members">
            <b-input
              :value="form.members"
              custom-class="is-static"
              readonly
            />
          </b-field>
          <b-field label="Leader">
            <b-input
              :value="form.leader"
              custom-class="is-static"
              readonly
            />
          </b-field>
          <b-field label="Responsibilities">
            <b-input
              :value="form.responsibilities"
              custom-class="is-static"
              readonly
            />
          </b-field>
        </card-component>
      </tiles>
    </section>
  </div>
</template>

<script>
import { defineComponent } from '@vue/composition-api'
import { mapState } from 'vuex'
// import find from 'lodash/find'
import TitleBar from '@/components/BaseTitleBar.vue'
import HeroBar from '@/components/BaseHeroBar.vue'
import Tiles from '@/components/BaseTiles.vue'
import CardComponent from '@/components/BaseCardComponent.vue'
import FilePicker from '@/components/BaseFilePicker.vue'
import UserAvatar from '@/components/BaseUserAvatar.vue'
import Notification from '@/components/BaseNotification.vue'

export default defineComponent({
  name: 'ProductsForm',
  components: {
    UserAvatar,
    FilePicker,
    CardComponent,
    Tiles,
    HeroBar,
    TitleBar,
    Notification
  },
  props: {
    id: {
      type: [String, Number],
      default: null
    }
  },
  data () {
    return {
      isProfileExists: false,
      isLoading: false,
      form: {
        id: '',
        name: '',
        description: '',
        members: '',
        leader: '',
        responsibilities: ''
      },
      createdReadable: null
    }
  },
  computed: {
    titleStack () {
      return [
        'Products',
        this.isProfileExists ? this.form.name : 'New Product'
      ]
    },
    heroTitle () {
      return this.isProfileExists ? this.form.name : 'Create Product'
    },
    heroRouterLinkTo () {
      return this.isProfileExists ? { name: 'product.new' } : { name: 'Products' }
    },
    heroRouterLinkLabel () {
      return this.isProfileExists ? 'New Product' : 'Dashboard'
    },
    formCardTitle () {
      return this.isProfileExists ? 'Edit Product' : 'Create Product'
    },
    ...mapState({
      products: state => state.products.products
    })
  },
  watch: {
    id (newValue) {
      this.isProfileExists = false

      if (!newValue) {
        this.form.id = ''
        this.form.name = ''
        this.form.description = ''
        this.form.members = ''
        this.form.leader = ''
        this.form.responsibilities = ''
        this.createdReadable = new Date().toLocaleDateString()
      } else {
        this.getData()
      }
    }
  },
  mounted () {
    this.getData()
  },
  methods: {
    getData () {
      if (this.$route.params.id) {
        const item = this.products.find((product) => product.id === this.$route.params.id)

        if (item) {
          this.isProfileExists = true
          this.form.id = item.id
          this.form.name = item.name
          this.form.description = item.description
          this.form.leader = item.leader
          this.form.members = item.members
          this.form.responsibilities = item.responsibilities

          this.createdReadable = new Date(item.created_mm_dd_yyyy).toLocaleDateString()
        }
      } else {
        this.$router.push({ name: 'product.new' })
      }
    },
    dateInput (v) {
      this.createdReadable = new Date(v).toLocaleDateString()
    },
    submit () {
      const productData = {
        description: this.form.description,
        members: this.form.members,
        leader: this.form.leader,
        responsibilities: this.form.responsibilities,
        name: this.form.name

      }
      const updateProduct = {
        productId: this.$route.params.id,
        product: this.form
      }
      if (this.$route.params.id) {
        this.$store.dispatch('products/updateProduct', updateProduct)
        this.$buefy.snackbar.open({
          message: 'Successfully updated the product',
          queue: true
        })
      } else {
        this.$store.dispatch('products/createProduct', productData)
        this.$buefy.snackbar.open({
          message: 'Successfully created the product',
          queue: true
        })
      }
    }
  }
})
</script>