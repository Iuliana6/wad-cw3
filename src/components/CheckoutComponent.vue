<template>
  <ul id="cart" class="lesson-container">
      <li v-for="lesson in cart" v-bind:key="lesson.subject">
        <div>
          <p>Subject: {{ lesson.subject }}</p>
          <p>Location: {{ lesson.location }}</p>
          <p>Price: £{{ lesson.price }}</p>
          <p>Quantity: {{ lesson.quantity }}</p>
          <button v-on:click="$emit('removeFromCart', lesson)">Remove from cart</button>
        </div>
        <img class="lesson-image" :src="'https://wad-cw2.herokuapp.com/' + lesson.imageURL" alt />
      </li>
    </ul>
    <div id="checkout">
      <h1>Checkout</h1>
      <p>Name:</p>
      <input type="text" id="name-input" />
      <p>Phone:</p>
      <input type="text" id="phone-input" />
      <button v-on:click="$emit('checkout')">Checkout</button>
    </div>
</template>

<script>
export default {
  name: 'CheckoutComponent',
  props: {
    cart: Object,
  },
}
</script>