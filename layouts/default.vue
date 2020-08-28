<template>
  <div>
    <header-app class="d-none d-lg-block"></header-app>

    <div class="container-fluid front">
      <div class="row">
        <div class="col-lg-12 px-lg-0">

          <Nuxt />

        </div>
      </div>

      <menu-mobile class="d-lg-none"></menu-mobile>

      <div class="cart-btn d-none d-lg-block bg-light shadow" :class="$route.name != 'productos-slug' ? 'cart-btn--medium' : 'cart-btn--top'">
        <div class="d-flex alig-items-center py-3 px-4" @click="mostrarCarrito()">
          <span class="icon d-flex justify-content-center align-items-center pr-3">
            <i class="fas fa-shopping-cart"></i>
          </span>

          <div class="cart-btn__info">
            <p class="text-muted my-0">
              <span class="font-weight-bold">{{ nroItemsCarrito }}</span>
              items
            </p>
            <p class="my-0 font-weight-bold">Mi carrito</p>
          </div>
        </div>
        <div class="text-center py-2 cart-btn__item">
          <a href="" class="text-white d-inline-block w-100" @click.prevent="toAccount()">Mi cuenta</a>
        </div>
        <div class="text-center py-2 cart-btn__item" v-if="userLogged">
          <a href="" class="text-white d-inline-block w-100" @click.prevent="logout()">
            {{ loading ? 'Saliendo...' : 'Salir' }}
            <i class="fas fa-sign-out-alt"></i>
          </a>
        </div>
      </div>

      <transition
        name="custom-classes-transition"
        enter-active-class="animated zoomInDown"
        leave-active-class="animated fadeOut"
      >
        <cart-modal v-if="modalCarrito" @closeModal="$store.commit('setModalCarrito', false)"></cart-modal>
      </transition>

      <modal-auth></modal-auth>
    </div>
  </div>
</template>

<script>
import { appConfig } from "../env";

import { mapState } from 'vuex'

import HeaderApp from '@/components/global/Header'
import MenuMobile from "@/components/global/MenuMobile";
import CartModal from "@/components/global/carrito/CartModal";
import ModalAuth from "@/components/global/auth/ModalAuth";

export default {
  data() {
    return {
      showModalCart: false,
      loading: false
    }
  },
  components: {
    HeaderApp,
    MenuMobile,
    ModalAuth,
    CartModal
  },
  mounted() {
    // this.$cookies.remove(appConfig.nameToken)
    // this.$cookies.remove('k_user_data')

    let cart = localStorage.getItem('kira_cart')

    if(cart) {
      // Realiza el conteo de productos en local storage
      let nroItems = JSON.parse(localStorage.getItem('kira_cart'))
      this.$store.commit('setNroItemsCarrito', nroItems.length)
    }

    // Se muestra el modal de login desde iniciar la web
    if(!this.userLogged) {
      this.$bvModal.show('modal-auth')
    }

  },
  methods: {
    mostrarCarrito() {
      // Abrir modal de carrito
      this.$store.commit('setModalCarrito', true)
    },
    toAccount() {
      // Si no está logueado
      if(!this.userLogged) {

        this.$bvModal.show('modal-auth')

      } else {

        // Redirigir según el tipo de usuario
        if(this.userData.typeUser == 1) {
          this.$router.push('/admin/productos')
        } else if(this.userData.typeUser == 2){
          this.$router.push('/mi-cuenta')
        } else {
          this.$router.push('/')
        }

      }
    },
    logout() {
      this.loading = true

      setTimeout(() => {
        this.$cookies.remove(appConfig.nameToken)

        this.$cookies.remove('k_user_data')

        this.loading = false

        this.$router.push('/')

        // Se recarga la página para poder obtener las cookies
        setTimeout(() => {
          this.$store.commit('reloadPage')
        }, 1000)
      }, 2000)
    }
  },
  computed: {
    ...mapState(['modalCarrito', 'nroItemsCarrito', 'showCategoriesMobile']),
    userLogged: function () {
      return this.$cookies.get(appConfig.nameToken) ? true : false
    },
    userData: function () {
      return this.$cookies.get('k_user_data')
    }
  }
}
</script>

<style lang="scss">
.first-column {
  position: fixed;
}

.cart-btn {
  border-bottom: 4px solid $danger;

  position: fixed;
  right: 1rem;
  z-index: 100;

  cursor: pointer;

  &--medium {
    bottom: 35vh;
  }

  &--top {
    bottom: 50vh;
  }

  &__item {
    background-color: rgba($danger, .9);

    border-top: 1px solid rgba($dark, .1);

    &:hover {
      background-color: $danger;
    }
  }
}
</style>