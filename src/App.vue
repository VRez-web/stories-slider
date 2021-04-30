<template>
  <div class="flex flex-col h-screen w-full bg-gray-900">
    <div class="flex flex-row justify-center items-center my-8 h-screen">
      <div
        v-for="(item, index) in items"
        :key="index"
        class="absolute mx-2 rounded-lg bg-gray-900"
        style="width: 350px; height: 500px; transition: transform 1s"
        :style="
          index == indexSelected
            ? `transform: translate(${380 * (index + difference)}px) scale(0.9)`
            : `transform: translate(${
                280 * (index + difference)
              }px) scale(0.4); cursor:pointer`
        "
        @click="index != indexSelected ? selectSlide(index) : ''"
      >
        <div class="bg-contain bg-no-repeat h-full rounded-lg">
          <div class="h-full">
            <img
              :src="
                index == indexSelected ? item.images[i].url : item.images[0].url
              "
              class="h-full rounded-lg w-full"
            />
          </div>

          <div class="w-full pt-4 absolute top-0" v-if="index == indexSelected">
            <div class="w-11/12 flex flex-row m-auto">
              <div
                class="w-full rounded-lg mr-2 relative h-auto"
                v-for="(el, indexo) in item.images.length"
                :key="indexo"
              >
                <div
                  class="absolute w-full rounded-lg shadow-md"
                  style="
                    height: 4px;
                    background-color: rgba(255, 255, 255, 0.7);
                  "
                ></div>
                <div
                  class="absolute w-full rounded-lg"
                  style="height: 4px; background-color: #fff"
                  :style="
                    indexo == i
                      ? `width:${percent}%`
                      : i > indexo
                      ? `width: 100%`
                      : `width:0%`
                  "
                ></div>
              </div>
            </div>

            <div class="flex flex-row w-11/12 mt-4 m-auto">
              <div class="flex justify-start items-center w-1/2">
                <div style="width: 35px; height: 35px" class="rounded-full">
                  <img
                    :src="item.picture"
                    class="rounded-full"
                    style="height: inherit; width: inherit"
                  />
                </div>
                <div class="ml-4">
                  <p class="text-sm text-white font-semibold">
                    {{ item.username }}
                  </p>
                </div>
              </div>
              <div class="flex justify-end items-center w-1/2">
                <svg
                  @click="isPaused ? startPlay() : stopPlay()"
                  version="1.1"
                  id="Capa_1"
                  xmlns="http://www.w3.org/2000/svg"
                  xmlns:xlink="http://www.w3.org/1999/xlink"
                  x="0px"
                  y="0px"
                  width="20px"
                  height="20px"
                  viewBox="0 0 163.861 163.861"
                  fill="#fff"
                  xml:space="preserve"
                  class="cursor-pointer"
                >
                  <g v-if="isPaused">
                    <path
                      d="M34.857,3.613C20.084-4.861,8.107,2.081,8.107,19.106v125.637c0,17.042,11.977,23.975,26.75,15.509L144.67,97.275
		c14.778-8.477,14.778-22.211,0-30.686L34.857,3.613z"
                    />
                  </g>
                  <g v-else>
                    <path
                      d="M17.991,40.976c0,3.662-2.969,6.631-6.631,6.631l0,0c-3.662,0-6.631-2.969-6.631-6.631V6.631C4.729,2.969,7.698,0,11.36,0
		l0,0c3.662,0,6.631,2.969,6.631,6.631V40.976z"
                    />
                    <path
                      d="M42.877,40.976c0,3.662-2.969,6.631-6.631,6.631l0,0c-3.662,0-6.631-2.969-6.631-6.631V6.631
		C29.616,2.969,32.585,0,36.246,0l0,0c3.662,0,6.631,2.969,6.631,6.631V40.976z"
                    />
                  </g>
                </svg>
              </div>
            </div>
          </div>

          <div v-else>
            <div
              class="absolute top-1/2 left-1/2"
              style="
                z-index: 99;
                transform: translate(-50%, -50%) scale(2.5);
                transition: transform 1.5s;
              "
            >
              <div class="flex flex-col items-center">
                <div
                  class="rounded-full border-2 border-blue-500 overflow-hidden"
                  style="width: 50px; height: 50px"
                >
                  <img :src="item.picture" class="rounded-full" />
                </div>
                <div class="mt-2">
                  <p class="text-sm text-white font-semibold">
                    {{ item.username }}
                  </p>
                </div>
              </div>
            </div>

            <div class="absolute inset-0 rounded-lg item-shadow"></div>
          </div>

          <div
            v-if="index == indexSelected"
            class="absolute top-1/2"
            style="left: -60px"
          >
            <i
              @click="prev(index)"
              class="icon-left-circle text-gray-500 cursor-pointer hover:text-gray-200"
              style="font-size: 36px"
            ></i>
          </div>
          <div
            v-if="index == indexSelected"
            class="absolute top-1/2"
            style="right: -60px"
          >
            <i
              @click="next(index)"
              class="icon-right-circle text-gray-500 cursor-pointer hover:text-gray-200"
              style="font-size: 36px"
            ></i>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import axios from "axios";
export default {
  name: "App",
  setup() {
    const indexSelected = ref(0),
      difference = ref(0),
      items = ref([]),
      i = ref(0),
      percent = ref(0),
      timer = ref(0),
      progress = ref(0),
      duration = ref(6000),
      interval = ref(0),
      isPaused = ref(false),
      newDuration = ref(0),
      pausePercent = ref(0);

    function selectSlide(index) {
      difference.value += indexSelected.value - index;
      indexSelected.value = index;
      i.value = 0;

      resetPlay();
    }

    function getItems() {
      axios.get("./items.json").then((res) => {
        items.value = res.data.items;
        play();
      });
    }

    function next(index) {
      let arrItems = items.value[indexSelected.value].images;
      if (
        indexSelected.value >= items.value.length - 1 &&
        i.value >= arrItems.length - 1
      ) {
        //
        setTimeout(() => {
          difference.value = 0;
          indexSelected.value = 0;
          i.value = 0;
        });
        //
      } else if (i.value >= arrItems.length - 1) {
        //
        setTimeout(() => {
          difference.value += index - (index + 1);
          indexSelected.value++;
          i.value = 0;
        });
        //
      } else {
        i.value++;
      }

      resetPlay();
    }

    function prev(index) {
      if (indexSelected.value <= 0 && i.value <= 0) {
        //
        i.value = 0;
        //
      } else if (i.value <= 0) {
        //
        setTimeout(() => {
          difference.value += index - (index - 1);
          indexSelected.value--;
          i.value = 0;
        });
        //
      } else {
        i.value--;
      }
      resetPlay();
    }

    function going() {
      let time = new Date().getTime();

      if (newDuration.value > 0) {
        percent.value =
          pausePercent.value +
          Math.floor((100 * (time - timer.value)) / duration.value);
      } else {
        percent.value = Math.floor(
          (100 * (time - timer.value)) / duration.value
        );
      }
    }

    function autoPlay() {
      let arrItems = items.value[indexSelected.value].images;
      if (
        indexSelected.value >= items.value.length - 1 &&
        i.value >= arrItems.length - 1
      ) {
        //

        stopPlay();
        return;

        //
      } else if (i.value >= arrItems.length - 1) {
        //

        difference.value += indexSelected.value - (indexSelected.value + 1);
        indexSelected.value++;
        i.value = 0;

        //
      } else {
        i.value++;
      }

      resetPlay();
    }

    function play() {
      timer.value = new Date().getTime();

      progress.value = setInterval(going, duration.value / 100);

      if (newDuration.value > 0) {
        interval.value = setInterval(autoPlay, newDuration.value);
      } else {
        interval.value = setInterval(autoPlay, duration.value);
      }
    }

    function stopPlay() {
      isPaused.value = true;
      pausePercent.value = percent.value;
      clearInterval(progress.value);
      clearInterval(interval.value);

      newDuration.value =
        duration.value - (pausePercent.value * duration.value) / 100;
    }

    function startPlay() {
      isPaused.value = false;

      play();
    }

    function resetPlay() {
      percent.value = 0;

      if (isPaused.value) {
        isPaused.value = false;
      }

      newDuration.value = 0;
      clearInterval(interval.value);
      clearInterval(progress.value);
      play();
    }

    onMounted(() => {
      getItems();
    });

    return {
      indexSelected,
      selectSlide,
      difference,
      items,
      getItems,
      next,
      prev,
      i,
      percent,
      going,
      play,
      timer,
      progress,
      duration,
      resetPlay,
      interval,
      autoPlay,
      isPaused,
      stopPlay,
      startPlay,
      newDuration,
      pausePercent,
    };
  },
};
</script>

<style>
body {
  overflow: hidden;
}
.item-shadow {
  z-index: 10;
  background: linear-gradient(
    180deg,
    rgba(38, 38, 38, 0.5973739837731968) 0%,
    rgba(255, 255, 255, 0.18280815744266454) 100%
  );
}
</style>
