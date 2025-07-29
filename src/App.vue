<script setup></script>

<template>
  <div v-if="withRender">
    <input type="text" v-model="inputValue" placeholder="Type something..." />

    <button data-site-key="8g8z65gaom" data-callback="handleFunction1" class="btn btn-primary btn-lg arcaptcha">
      Submit
    </button>
  </div>

  <!-- with Render function  -->
  <div v-else>
    <div id="captcha"></div>

    <button @click="calculate">Submit with render</button>
  </div>
</template>

<script>

window.handleFunction1 = function (token) {
  console.log(token);
};

export default {
  data() {
    return {
      withRender: false, // can switch here 
    };
  },

  mounted() {
    if (!this.withRender) {
      window.arcaptcha.render('#captcha', {
        site_key: '8g8z65gaom',
        size: 'invisible',
        callback: this.handleSubmit2,
      });
    }
  },
  methods: {
    handleSubmit2(token) {
      console.log(token);
    },

    calculate() {
      arcaptcha.execute()
    }
  },
};
</script>

<style scoped></style>
