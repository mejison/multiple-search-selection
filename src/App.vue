<template>
  <div id="app" class="box">
    <multiple-search 
      label="Account category"
      description="E.g. makeup artist, marketing agency, etc."
      :required="true"
      :options="options"
      @change="onChangeHandler"
      />
  </div>
</template>

<script>
import MultipleSearch from './components/MultipleSearch.vue'

export default {
  name: 'app',
  components: {
    MultipleSearch
  },
  data() {
    return {
      options: [
        {
          id: 1, 
          title: 'Business',
          children: [
            {
              id: 2,
              title: 'Mining',
              children: [
                {
                  id: 3,
                  title: 'Traded Market',
                },
              ],
            },
            {
              id: 4,
              title: 'Financial',
              children: [
                {
                  id: 5,
                  title: 'Commercial',
                },
                {
                  id: 6,
                  title: 'Credit',
                },
                {
                  id: 9,
                  title: 'Other',
                },
              ],
            },
          ]
        },
        {
          id: 10, 
          title: 'Interest',
          children: [
            {
              id: 11,
              title: 'Science',
              children: [
                {
                  id: 12, 
                  title: 'Asteroid mining',
                },
                {
                  id: 13, 
                  title: 'Astronautics',
                },
                {
                  id: 14, 
                  title: 'Mars colonization',
                },
              ],
            },
            {
              id: 15,
              title: 'Science fiction',
              children: [],
            },
            {
              id: 16,
              title: 'Science research',
              children: [],
            }
          ]
        },
      ]
    }
  },
  methods: {
    onChangeHandler() {

    }
  },
}
</script>

<style>
  @import "./main.css";
</style>