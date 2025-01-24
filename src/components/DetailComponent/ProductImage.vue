<template>
  <div class="ProductImage-container">
    <div class="image-section">
      <div class="row">
        <img
          v-for="(image, index) in images"
          :key="index"
          :src="image"
          alt="product_image"
          @click="setLargeImage(index)"
          :class="[
            'product_image',
            productImage,
            { active: index === activeIndex },
          ]"
        />
      </div>

      <div class="large-section">
        <img :src="images[activeIndex]" alt="large_image" />
        <div class="button-container">
          <button @click="prevImage">
            <iconify-icon
              icon="mingcute:arrow-left-fill"
              width="32px"
              height="32px"
            />
          </button>
          <button @click="nextImage">
            <iconify-icon
              icon="mingcute:arrow-right-fill"
              width="32px"
              height="32px"
            />
          </button>
        </div>
      </div>
    </div>

    <div class="popup-overlay" v-if="showPopup">
      <div class="popup">
        <div class="popup-header">
          <h1>Unlock Your True Artistic Potential</h1>
        </div>

        <h2 class="popuptitle">
          Join the culture
        </h2>
        <button class="popup-btn" @click="gotoRegistration">Login or Sign up</button>
        <button class="later-btn" @click="closePopUp">Maybe later</button>
      </div>
    </div>

    <div class="SizePrice-section">
      <div class="title-section">
        <hgroup>
          <h3 class="brandname">{{ brandname }}</h3>
          <h1 class="productname">{{ productname }}</h1>
        </hgroup>
        <button class="x-button" @click="$router.go(-1)">
          <iconify-icon icon="maki:cross" width="20px" height="20px" />
        </button>
      </div>
      <div class="func-section">
        <div class="size-selector" @click="showList">
          <span>Size: </span>
          <div class="dropper">
            <span :class="[activated ? size_activated : 'null']">
              {{
                activated
                  ? productSizes[selectedIndex].size || "Select Size"
                  : "All"
              }}
            </span>

            <iconify-icon
              v-if="show_list"
              icon="ri:arrow-drop-down-line"
              style="transform: scaleY(-1)"
              width="24px"
              height="24px"
            />
            <iconify-icon
              v-else
              icon="ri:arrow-drop-down-line"
              width="24px"
              height="24px"
            />
          </div>
        </div>

        <div class="dropped-list" v-if="show_list">
          <h3>Size (US)</h3>
          <div class="sizes">
            <div
              :class="[selectedIndex === index ? active_state : normal_state]"
              v-for="(item, index) in productSizes"
              :key="index"
              @click="selectedSizeIndex(index)"
            >
              <p class="size-price">
                <span>{{ item.size }}</span>
                <br />
                <span>{{ formatter.format(item.price) }}</span>
              </p>
            </div>
          </div>
        </div>

        <div class="buy-section">
          <hgroup>
            <h3 style="margin-bottom: 10px">Buy now for:</h3>
            <h1 class="price" :class="[activated ? price_activated : 'null']">
              {{activated ? `${formatter.format(productSizes[selectedIndex].price) || formatter.format(price_tag)}` : `${price_tag}`}}
            </h1>
          </hgroup>
          <div class="buy-container">
            <button class="buy-btn" @click="gotoCartPage">BUY NOW</button>
          </div>
        </div>

        <div class="miscellaneous">
          <div class="kVeri-head" @click="showDesc">
            <div class="sec-1">
              <h4>Kravan Verified</h4>
              <iconify-icon
                icon="solar:verified-check-outline"
                width="24px"
                height="24px"
              />
            </div>
            <iconify-icon
              icon="ri:arrow-drop-down-line"
              v-if="show_desc"
              style="transform: scaleY(-1)"
              width="24px"
              height="24px"
            />
            <iconify-icon
              icon="ri:arrow-drop-down-line"
              v-else
              width="24px"
              height="24px"
            />
            <hr v-if="show_desc" width="100%" />
          </div>
          <div class="desc" v-if="show_desc">
            <p>
              "Kravan Verified" represents our commitment to excellence,
              guaranteeing every item undergoes a meticulous inspection, every
              time. <router-link to="/Privacy&Policy">Learn more.</router-link>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { useCartStore } from "@/store/cart";
import { useProductStore } from "@/store/ProductStore/product";
import { useUserStore } from "@/store/user";
import { mapState } from "pinia";

export default {
  props: {
    images: {
      type: Array,
      required: true,
    },
    brandname: {
      type: String,
      required: true,
    },
    productname: {
      type: String,
      required: true,
    },
    price_tag: {
      type: Number,
      required: true,
    },
    productSizes: {
      type: Array,
      required: true,
    },
    productId: {
      type: String,
      required: true,
    },
    productImage: {
      type: String,
      default: "medium",
    },
  },

  data() {
    return {
      activeIndex: 0,
      show_list: false,
      show_desc: true,
      isActive: false,

      // size button handling
      normal_state: "size-button",
      active_state: "size-active",
      selectedIndex: null,
      activated: false,
      price_activated: "price-activated",
      size_activated: "size-activated",

      formatter: new Intl.NumberFormat("en-us", {
        style: "currency",
        currency: "USD",
        trailingZeroDisplay: "stripIfInteger",
      }),

      //Pop up thing
      showPopup: false,
    };
  },

  setup() {
    const productStore = useProductStore();
    const cartStore = useCartStore();
    const userStore = useUserStore();

    return {
      productStore,
      cartStore,
      userStore
    };
  },

  computed: {
    selectedSize() {
      return this.productStore.selectSize;
    },
    ...mapState(useCartStore, {
      cartItems: "cartItems",
    }),
  },

  methods: {
    selectedSizeIndex(index) {
      this.selectedIndex = index;
      (this.show_list = false), (this.activated = true);

      this.productStore.selectSize(this.productSizes[index]);
    },
    showList() {
      this.show_list = !this.show_list;
    },
    showDesc() {
      this.show_desc = !this.show_desc;
      console.log(this.show_desc);
    },
    setLargeImage(index) {
      this.activeIndex = index;
    },
    prevImage() {
      this.activeIndex =
        (this.activeIndex - 1 + this.images.length) % this.images.length;
    },
    nextImage() {
      this.activeIndex = (this.activeIndex + 1) % this.images.length;
    },
    gotoCartPage() {
      if (!this.activated || this.selectedIndex === null) {
        alert("Please select a size before adding to the cart.");
        return;
      }

      if(!this.userStore.isLogIn){
        this.showPopup = true;
        return;
      }

      const cartItem = {
        productId: this.productId,
        name: this.productname,
        brand: this.brandname,
        image: this.images[0],
        size: this.productSizes[this.selectedIndex]?.size,
        price: this.productSizes[this.selectedIndex]?.price,
      };

      this.cartStore.addToCart(cartItem);
      this.cartStore.calculateTotalPrice();
      console.log("Cart Items:", this.cartItems);
      this.$router.push("/cart");
    },
    gotoRegistration(){
      this.$router.push("/Registration");
    },
    closePopUp(){
      this.showPopup = false;
    }
  },
};
</script>

<style scoped>
.ProductImage-container {
  font-family: "Inter";
  display: flex;
  position: relative;
  flex-direction: row;
  justify-content: space-between;
  gap: 2rem;
  width: 100%;
  height: fit-content;
}

.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
  z-index: 100;
}

.popup {
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 15px black;
  width: 50%;
  justify-content: center;
  align-items: center;
  display: flex;
  flex-direction: column;
}
.popup-header {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #0d0907;
  border-radius: 10px 10px 0px 0px;
  width: 100%;
  height: 100px;
  color: white;
  font-family: "Inter";
  font-size: 18px;
}

.popuptitle {
  font-size: 30px;
}

.popup-btn{
  width: 30%;
  height: 45px;
  border: 1px solid black;
  background-color: black;
  color: white;
  font-family: "Inter";
  font-weight: bold;
  font-size: 20px;
  border-radius: 100px;
  margin: 20px;
  cursor: pointer;
}

.later-btn {
  background-color: transparent;
  border: none;
  margin-bottom: 20px;
  font-weight: bold;
  font-size: 18px;
  color: grey;
  opacity: 70%;
  cursor: pointer;
}

/* LEFT SECTION */
.ProductImage-container .image-section {
  display: flex;
  flex-direction: row;
  gap: 7px;
  width: 60%;
  max-width: 100%;
  height: 100%;
}

.row {
  display: flex;
  flex-direction: column;
  gap: 7px;
  height: 860px;
  overflow-y: scroll;
}

.row > img {
  cursor: pointer;
  background-color: #e7e7e7;
  border: 1px solid #cfcfcf;
  border-radius: 5px;
}
.product_image.medium {
  width: 8rem;
  height: 8rem;
  max-width: 100%;
  max-height: 100%;
  object-fit: fill;
}
.product_image.large {
  width: 8rem;
  height: 8rem;
  max-width: 100%;
  max-height: 100%;
  object-fit: fill;
}

.large-section {
  position: relative;
  width: 860px;
  height: 860px;
}

.large-section > img {
  height: 100%;
  width: 100%;
  background-color: #e7e7e7;
  border-radius: 5px;
  border: 1px solid #cfcfcf;
}

.button-container {
  display: flex;
  right: 20px;
  bottom: 15px;
  gap: 5px;
  position: absolute;
}

.button-container > button {
  background-color: transparent;
  border: none;
  cursor: pointer;
}

/* RIGHT SECTION */
.ProductImage-container .SizePrice-section {
  width: 40%;
  height: 100%;
}

.SizePrice-section .title-section {
  width: 100%;
  display: flex;
  flex-direction: row;
  position: relative;
}

.title-section > button {
  position: absolute;
  right: 2rem;
  top: 2rem;
}

.title-section .x-button {
  background-color: transparent;
  border: none;
  cursor: pointer;
}

.brandname {
  margin-top: 0;
  margin-bottom: 15px;
}

.productname {
  margin-bottom: 15px;
}

.func-section {
  display: flex;
  position: relative;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 1.5rem;
  height: auto;
  margin-top: 4rem;
}

.size-selector {
  font-weight: 400;
  border: 2px solid black;
  width: 80%;
  height: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-radius: 15px;
  cursor: pointer;
}

.dropper {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 5px;
}

.buy-section {
  width: 80%;
  border: 2px solid black;
  padding: 1rem;
  border-radius: 15px;
  height: 230px;
}

.price {
  font-size: 50px;
  margin-top: 0;
}

.buy-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.buy-btn{
  width: 70%;
  height: 60px;
  font-weight: 900;
  font-size: x-large;
  border-radius: 30px;
  color: white;
  background-color: black;
  border: none;
  cursor: pointer;
}

.buy-btn:hover {
  color: white;
  background-color: #004100;
  transition: 100ms ease-in;
}

.buy-btn .link {
  text-decoration: none;
  color: white;
}

.dropped-list {
  border: 2px solid black;
  border-top: 0px;
  width: 80%;
  height: 100%;
  padding: 1rem;
  position: absolute;
  background-color: white;
  z-index: 10;
  top: 40px;
  border-radius: 0px 0px 15px 15px;
}

.dropped-list > h3 {
  margin-top: 0px;
  font-weight: 600;
}

.sizes {
  display: grid;
  grid-template-columns: auto auto auto;
  gap: 10px;
  width: 100%;
  font-weight: bold;
}

.size-button {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  border-radius: 5px;
  border: 2px solid black;
  font-weight: 600;
  background-color: white;
  color: black;
  cursor: pointer;
}

.size-button:hover {
  color: white;
  font-weight: 800;
  transition: 300ms ease-in;
  border: 2px solid #004100;
  background-color: #004100;
}

.size-active {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  border-radius: 5px;
  border: 2px solid black;
  font-weight: 800;
  background-color: #004100;
  color: white;
  cursor: pointer;
}

.miscellaneous {
  width: 80%;
  padding: 1rem;
  margin-top: 0;
}

.miscellaneous > h4 {
  margin: 0;
  padding: 0;
}

.sec-1 {
  display: flex;
  align-items: center;
  flex-direction: row;
  gap: 10px;
}

.kVeri-head {
  display: flex;
  align-items: center;
  flex-direction: row;
  justify-content: space-between;
  width: 100%;
  cursor: pointer;
}

.kVeri-head > hr {
  position: absolute;
  align-self: flex-end;
  width: 80%;
}

.size-price {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  margin: 0;
  padding: 0;
}

.price-activated,
.size-activated {
  font-weight: 700;
  color: #006e00;
}
</style>
