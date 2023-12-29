<template>
  <div class="card-wrapper">
    <div class="card">
      <div class="img">
        <img src="../assets/car.png" alt="">
      </div>
      <div class="text-content">
        <p>Hersteller: {{propCarObject.hersteller}}</p>
        <p>Modell: {{propCarObject.modell}}</p>
        <p>KM: {{propCarObject.kilometerstand}}</p>
        <p>Preis: {{propCarObject.preis}}</p>
        <div class="link">
        <router-link to="/">Mehr Details</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {defineComponent, PropType} from "vue";
import {CarResponseData} from "../classes/CarResponseData";
export default defineComponent( {
  name: "Card",
  props: {
    propCarObject: {
      type: Object as PropType<CarResponseData>,
      required: true
    }
  },
  setup() {
    return {

    }
  }

})
</script>

<style scoped>
.card-wrapper {
  width: 200px;
  background: rgba(0, 0, 0, 0.2);
}

.card {
  display: flex;
  flex-direction: column;
}

.card .img img {
  object-fit: contain;
  width: 100%;
}

.link {
  display: flex;
  justify-content: center;
  border-radius: 5px;
}

.link a {
  text-decoration: none;
  background: white;
  margin: 10px;
  padding: 5px 20px;
  border-radius: inherit;
  position: relative;
}

.link a::before {
  content: "";
  position: absolute;
  top: 0;
  left:0;
  height: 100%;
  width: 0;
  background-color: rgba(0,0,0,0.1);
  transition: 0.5s all ease-in-out;
}


.link a:hover::before{
  width: 100%;
}


</style>