<template>
  <div>
    <base-dialog
      :show="!!error"
      title="An error occurred!"
      @close="handleError"
    >
      <p>{{ error }}</p>
    </base-dialog>
    <section>
      <base-card>
        <coach-filter @change-filter="setFilters"></coach-filter>
        <div class="controls">
          <base-button mode="outline" @click="loadCoaches(true)"
            >Refresh</base-button
          >

          <base-button
            v-if="isloggedIn && !isCoach && !isloading"
            link
            to="/register"
            >Register as Coach</base-button
          >
          <base-button link to="/auth?redirect=register" v-if="!isloggedIn"
            >Login to Register as Coach</base-button
          >
        </div>
        <div v-if="isloading">
          <base-spinner></base-spinner>
        </div>

        <ul v-else-if="hasCoaches">
          <coach-item
            v-for="coach in filteredCoaches"
            :key="coach.id"
            :id="coach.id"
            :first-name="coach.firstName"
            :last-name="coach.lastName"
            :rate="coach.hourlyRate"
            :areas="coach.areas"
          ></coach-item>
        </ul>

        <h3 v-else>No coaches found.</h3>
      </base-card>
    </section>
  </div>
</template>

<script>
import CoachItem from '../../components/coaches/CoachItem.vue';
import CoachFilter from '../../components/coaches/CoachFilter.vue';

export default {
  components: {
    CoachItem,
    CoachFilter,
  },
  data() {
    return {
      isloading: false,
      error: null,
      activeFilters: {
        frontend: true,
        backend: true,
        career: true,
      },
    };
  },
  computed: {
    isloggedIn() {
      return this.$store.getters.isAuthenticated;
    },
    isCoach() {
      return this.$store.getters['coaches/isCoach'];
    },
    filteredCoaches() {
      const coaches = this.$store.getters['coaches/coaches'];
      return coaches.filter((coach) => {
        if (this.activeFilters.frontend && coach.areas.includes('frontend')) {
          return true;
        }
        if (this.activeFilters.backend && coach.areas.includes('backend')) {
          return true;
        }
        if (this.activeFilters.career && coach.areas.includes('career')) {
          return true;
        }
        return false;
      });
    },
    hasCoaches() {
      return !this.isloading && this.$store.getters['coaches/hasCoaches'];
    },
  },
  created() {
    this.loadCoaches();
  },
  methods: {
    setFilters(updateFilters) {
      this.activeFilters = updateFilters;
    },
    async loadCoaches(refresh = false) {
      this.isloading = true;
      try {
        await this.$store.dispatch('coaches/loadCoaches', {
          forceRefresh: refresh,
        });
      } catch (error) {
        this.error = error.message || 'Something went wrong!';
      }
      this.isloading = false;
    },
    handleError() {
      this.error = null;
    },
  },
};
</script>

<style scoped>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.controls {
  display: flex;
  justify-content: space-between;
}
</style>
