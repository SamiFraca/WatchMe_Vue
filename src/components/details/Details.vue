<template>
  <div class="flex mt-8 lg:px-48 md:px-16 px-8 text-left flex-col">
    <div class="flex md:flex-row flex-col w-full">
      <img
        :src="this.retrievedInfo.imageUrl"
        class="img-size md:ml-0 mb-8 md:mb-0"
      />
      <div class="md:ml-8 flex flex-col gap-8">
        <h2 class="text-xl font-semibold">{{ this.retrievedInfo.name }}</h2>
        <p>Location: {{ this.retrievedInfo.location }}</p>
        <p>Capacity: {{ this.retrievedInfo.capacity }}</p>
        <p>{{ this.retrievedInfo.description }}</p>
      </div>
    </div>
    <h2 class="text-lg font-medium border-b pb-2 mt-8 mb-4">Shows</h2>
    <div v-if="this.retrievedInfo.shows.length > 0">
      <div
        v-for="shows in this.retrievedInfo.shows"
        :key="shows.id"
        class="flex flex-col gap-4 border p-4 rounded-md mb-4"
      >
        <h2 class="text-lg text-black">{{ shows.title }}</h2>
        <h2 class="text-md text-black">Schedule</h2>
        <p class="text-md text-black">
          From <span class="font-medium">{{ shows.startTime }}</span> to
          <span class="font-medium">{{ shows.startTime }}</span>
        </p>
        <p>Maximum capacity: {{ shows.maxCap }}</p>
        <p>Remaining capacity: {{ shows.maxCap - shows.actualCap }}</p>
        <div class="wrapper"></div>
        <a
          class="button w-32 rounded-md"
          :key="shows.id"
          :disabled="shows.disabled || shows.maxCap == shows.actualCap"
          @click="updateActualCapRequest(shows)"
          :class="{ 'fill-success': success && clickedShow == shows.id }"
        >
          <span>Reserve</span>
        </a>

        <div v-if="errorCapacity" class="shake text-red">
          Show is already full capacity
        </div>
      </div>
    </div>
    <div v-else>No shows available</div>
  </div>
</template>

<script>
export default {
  name: "details",
  data() {
    return {
      retrievedInfo: [],
      name: "",
      errorCapacity: false,
      success: false,
      clickedShow: 0,
      token:null
    };
  },
  methods: {
    async updateActualCapRequest(show) {
      const actualCap = show.actualCap + 1;
      const showId = show.id;
      this.clickedShow = show.id;
      const userToken = sessionStorage.getItem("token");
      if (userToken) {
        if (actualCap >= show.maxCap || showId == null) {
          this.errorCapacity = true;
          return false;
        } else {
          const patchOperation = [
            {
              op: "replace",
              path: "/actualCap",
              value: actualCap,
            },
          ];
          console.log(patchOperation);
          try {
            await this.$store.dispatch("UpdateActualCap", {
              patchOperation,
              showId,
            });
            this.success = true;
            show.disabled = true; // Disable the button after successful response
          } catch (error) {
            this.searchError = true;
            console.log(error);
          }
        }
      }else{
        this.$router.push('/login')
      }
    },
  },

  created() {
    if (window.location.href.indexOf("locations") > -1) {
      const locationSearch = localStorage.getItem("locationSearch");
      const locations = JSON.parse(locationSearch);
      const filterLocation = locations.filter(
        (location) => location.id === parseInt(this.$route.params.itemId)
      );
      this.retrievedInfo = filterLocation[0];
      console.log(this.retrievedInfo.shows.length);
    }
    if (window.location.href.indexOf("sports") > -1) {
      const nameSearch = localStorage.getItem("sportSearch");
      const locations = JSON.parse(nameSearch);
      const filterLocation = locations.filter(
        (location) => location.id === parseInt(this.$route.params.itemId)
      );
      this.retrievedInfo = filterLocation[0];
      console.log(this.retrievedInfo);
    }
    if (window.location.href.indexOf("names") > -1) {
      const nameSearch = localStorage.getItem("nameSearch");
      const locations = JSON.parse(nameSearch);
      const filterLocation = locations.filter(
        (location) => location.id === parseInt(this.$route.params.itemId)
      );
      this.retrievedInfo = filterLocation[0];
      console.log(this.retrievedInfo.shows.length);
    }
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Mukta:wght@600&display=swap");

h1 {
  font-family: "Mukta", sans-serif;
}
.bar-pics {
  height: 15rem;
  width: 20rem;
}

.wrapper {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

a {
  display: block;
  height: 40px;
  line-height: 40px;
  font-size: 18px;
  font-family: sans-serif;
  text-decoration: none;
  color: #333;
  border: 1px solid #333;
  letter-spacing: 2px;
  text-align: center;
  position: relative;
  transition: all 0.35s;
  cursor: pointer;
}

a span {
  position: relative;
}

a:after {
  position: absolute;
  content: "";
  top: 0;
  left: 0;
  width: 0;
  height: 100%;
  background: #59d184;
  transition: width 0.35s ease;
}

a.fill-success:after {
  width: 100%;
}

a:before {
  content: "✓";
  color: white;
  position: absolute;
  z-index: 10;
  left: 45%;
  transform: translateX(-50%);
  opacity: 0;
  transition: opacity 0.35s ease 0.1s, transform 0.35s ease 0.1s;
  transform: scale(0.25);
}

a.fill-success:before {
  opacity: 1;
  transform: scale(1);
}

.mt-8 {
  margin-top: 2rem;
}
</style>
