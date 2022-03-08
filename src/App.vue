<template>
  <div id="container">
    <div id="header">
      <img
        v-on:click="toggleCart()"
        v-if="checkoutButtonShown"
        id="checkoutButton"
        src="https://wad-cw2.herokuapp.com/images/checkout-icon.png"
      />
    </div>
    <div id="search-container">
      <input type="text" v-model="searchString" v-on:keyup="searchFn()" placeholder="Search..." />
    </div>
    <div id="filters">
      <h1>Sort</h1>
      <select id="sorting-criteria" v-model="selectedCriteria">
        <option
          v-for="criteria in sortingCriteria"
          v-bind:value="criteria"
          v-bind:key="criteria"
        >{{ criteria }}</option>
      </select>
      <select id="sorting-order" v-model="selectedOrder">
        <option v-for="order in sortingOrder" v-bind:value="order" v-bind:key="order">{{ order }}</option>
      </select>
    </div>
    <ul id="lesson-list" class="lesson-container" v-if="lessonListShown">
      <li v-for="lesson in sortedLessons" v-bind:key="lesson.subject">
        <div>
          <p>Subject: {{ lesson.subject }}</p>
          <p>Location: {{ lesson.location }}</p>
          <p>Price: £{{ lesson.price }}</p>
          <p>Spaces: {{ lesson.space }}</p>
          <button v-on:click="addToCart(lesson)">Add to cart</button>
        </div>
        <img class="lesson-image" :src="'https://wad-cw2.herokuapp.com/' + lesson.imageURL" alt />
      </li>
    </ul>
    <LessonComponent></LessonComponent>
    <CheckoutComponent v-if="cartShown" v-bind:apiUrl="this.apiUrl" v-bind:cart="this.cart"
    v-on:removeFromCart="removeFromCart" v-on:checkout="checkout"></CheckoutComponent>
  </div>
</template>

<script>
import CheckoutComponent from './components/CheckoutComponent.vue'
import LessonComponent from './components/LessonComponent.vue'
const apiUrl = "https://wad-cw2.herokuapp.com"
export default {
  name: 'App',
  components: {
    CheckoutComponent,
    LessonComponent
  },
  data: () => {
    return {
      apiUrl: apiUrl,
      lessons: [],
      cart: [],
      lessonListShown: true,
      cartShown: false,
      checkoutButtonShown: false,
      sortingCriteria: ["Subject", "Location", "Price", "Availability"],
      sortingOrder: ["Ascending", "Descending"],
      selectedCriteria: "Subject",
      selectedOrder: "Ascending",
      searchString: "",
      sortedLessons: [],
    }
  },
  created: function () {
    let that = this
    fetch(apiUrl + '/lessons').then(
      function (response) {
        response.json().then(
          function (json) {
            that.lessons = json.result;
            that.sortedLessons = json.result;
          }
        )
      })
  },
  methods: {
    searchFn: function f() {
      let that = this
      fetch(apiUrl + '/search?searchString=' + that.searchString, {
      }).then(
        function (response) {
          response.json().then(
            function (json) {
              that.sortedLessons = json.result
            }
          )
        })
    },
    addToCart: function f(lesson) {
      if (lesson.space === 0) {
        return
      }

      //where the lesson is found in the cart
      let lessonIndex = -1
      this.cart.forEach((cartLesson, i) => {
        if (cartLesson.subject === lesson.subject) {
          lessonIndex = i
        }
      })
      //if item was not found
      if (lessonIndex === -1) {
        //add lesson to the cart
        this.cart.push({
          subject: lesson.subject,
          location: lesson.location,
          price: lesson.price,
          quantity: 1,
          imageURL: lesson.imageURL
        })
      } else {
        //if already found increase quantity
        this.cart[lessonIndex].quantity += 1
      }

      lesson.space = lesson.space - 1

      //show the checkout button
      this.checkoutButtonShown = true
    },
    removeFromCart: function f(lesson) {
      this.lessons.forEach(l => {
        //add quantity back to lesson list
        if (l.subject === lesson.subject) {
          l.space += lesson.quantity
        }
      })
      //remove lesson from cart
      this.cart = this.cart.filter(l => l.subject !== lesson.subject)
    },
    toggleCart: function f() {
      this.cartShown = !this.cartShown;
      this.lessonListShown = !this.lessonListShown;
    },
    checkout: function f() {
      let name = document.getElementById("name-input").value
      let phoneNumber = document.getElementById("phone-input").value
      if (name === "" || phoneNumber === "") {
        alert("Please fill out all the fields")
        return;
      }
      for (let i = 0; i < name.length; i++) {
        let c = name.charAt(i)
        if (!((c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c === ' ')) {
          alert("Invalid name")
          return;
        }
      }
      for (let i = 0; i < phoneNumber.length; i++) {
        let c = phoneNumber.charAt(i)
        if (!(c >= '0' && c <= '9')) {
          alert("Invalid phone number")
          return
        }
      }
      this.cart.forEach(lesson => {
        fetch(apiUrl + "/order", {
          method: 'POST', // *GET, POST, PUT, DELETE, etc.
          mode: 'cors', // no-cors, *cors, same-origin
          cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
          credentials: 'same-origin', // include, *same-origin, omit
          headers: {
            'Content-Type': 'application/json'
            // 'Content-Type': 'application/x-www-form-urlencoded',
          },
          redirect: 'follow', // manual, *follow, error
          referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
          body: JSON.stringify({
            name: name,
            phoneNumber: phoneNumber,
            subject: lesson.subject,
            location: lesson.location,
            price: lesson.price,
            quantity: lesson.quantity,
            imageURL: lesson.imageURL
          }) // body data type must match "Content-Type" header
        });
        let newQuantity = this.lessons.filter(l => l.subject == lesson.subject)[0].space
        console.log(newQuantity)
        fetch(apiUrl + "/lessons", {
          method: 'PUT', // *GET, POST, PUT, DELETE, etc.
          mode: 'cors', // no-cors, *cors, same-origin
          cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
          credentials: 'same-origin', // include, *same-origin, omit
          headers: {
            'Content-Type': 'application/json'
            // 'Content-Type': 'application/x-www-form-urlencoded',
          },
          redirect: 'follow', // manual, *follow, error
          referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
          body: JSON.stringify({
            subject: lesson.subject,
            location: lesson.location,
            price: lesson.price,
            quantity: newQuantity,
            imageURL: lesson.imageURL
          }) // body data type must match "Content-Type" header
        });

      })
      this.cart = []
      alert("Checkout successfull")

    }
  }
}
</script>

<style>
#header {
  padding: 5px;
  width: 100%;
  height: 35px;
  background-color: cornflowerblue;
}

#checkoutButton {
  margin-left: 15px;
  height: 30px;
  width: 30px;
}

#checkout {
  margin: 10px 30%;
  display: flex;
  flex-direction: column;
}
* > button {
  padding: 10px 10px;
  border-radius: 7px;
  font-weight: bold;
}
* > button:hover {
  background-color: #f56fa7;
  color: white;
  border-color: #f56fa7;
}
#checkout > button {
  margin: 10px 0;
}

.lesson-image {
  width: 150px;
  height: 150px;
  padding: 10px;
  margin: 0;
}
#search-container {
  width: 100%;
}
#search-container > input {
  width: 100%;
}
#filters {
  margin: 5px 0;
  display: flex;
  flex-direction: row;
}
#filters > h1 {
  font-size: 25px;
  margin: 0 5px;
}
#filters > select {
  margin: 0 5px;
}
.lesson-container {
  margin-top: 30px;
  height: 100%;
  flex-wrap: wrap;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 15px;
}
.lesson-container > li {
  width: 360px;
  background-color: aliceblue;
  border-style: solid;
  border-color: dimgray;
  border-radius: 20px;
  padding: 10px;
  display: flex;
  flex-direction: row;
}
.lesson-container > li > div {
  width: 180px;
  flex-direction: column;
}

.lesson-container > li > div > button {
  margin: 10px 0;
}
* {
  margin: 0;
  padding: 0;
}
body * {
  font-family: Verdana, sans-serif;
}
</style>
