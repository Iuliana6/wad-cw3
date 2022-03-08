<template>
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
</template>
<script>
export default {
  name: 'LessonComponent',
  props: {
    cart: Object,
  },
}
</script>