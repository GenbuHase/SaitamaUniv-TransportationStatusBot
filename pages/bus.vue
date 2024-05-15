<template>
  <VContainer id="busPage">
    <VRow>
      <VCol cols="12">
        <VSheet>
          <VForm id="busPage-fetchForm" class="v-container">
            <VRow>
              <VCol cols="12">
                <VSelect
                  id="busPage-fetchForm-intervalSelector"
                  label="区間" :items="__INTERVALS"
                  item-title="label" item-value="value"
                  v-model="intervalSelector.selected"
                  @update:model-value="saveInterval($event)"

                  variant="outlined" hide-details
                  prepend-inner-icon="mdi-source-commit-start" />
              </VCol>
            </VRow>

            <VRow>
              <VCol cols="12">
                <VBtn
                  id="busPage-fetchForm-submitBtn"
                  block color="primary" prepend-icon="mdi-database-search"
                  @click="fetchServices(intervalSelector.selected)">
                  検索
                </VBtn>
              </VCol>
            </VRow>
          </VForm>
        </VSheet>
      </VCol>

      <VCol cols="12">
        <VSheet rounded="lg" class="overflow-hidden" >
          <VProgressLinear color="primary" v-if="loading" indeterminate class="custom-progress-bar" />
          <div color="primary" v-else class="loading-placeholder" ></div>
          <VTabs id="busPage-fetchResult-tabs" align-tabs="center" v-model="fetchResultTabs.selected">
            <VTab>到着予定時刻順</VTab>
            <VTab>時刻表順</VTab>
            <VBtn variant="text" icon="mdi-reload" @click="fetchServices(intervalSelector.selected)"/>
          </VTabs>

          <VWindow id="busPage-fetchResult-panels" v-model="fetchResultTabs.selected">
            <VWindowItem>
              <VList>
                <VListItem v-for="(service, index) of arrivalTimeSortedServices" :key="index">
                  <ArrivalTimeSortedService :="service" />
                </VListItem>
              </VList>
            </VWindowItem>

            <VWindowItem>
              <VList>
                <VListItem v-for="(service, index) of plannedTimeSortedServices" :key="index">
                  <PlannnedTimeSortedService :="service" />
                </VListItem>
              </VList>
            </VWindowItem>
          </VWindow>
        </VSheet>
      </VCol>
    </VRow>
  </VContainer>
</template>

<style lang="scss" scoped>
  #busPage {
    padding: 0;
  }
  .loading-placeholder {
    height: 4px;
  }
</style>
<style>
  .custom-progress-bar .v-progress-linear__background {
    background-color: rgba(0, 0, 0, 0) !important;
  }
</style>

<script lang="ts" setup>
  import { ref, computed } from "vue";
  import Bus from "@/utils/Bus";
  import LocalStorage from "@/utils/LocalStorage";

  const storage = new LocalStorage("WakameNavi.BusPage");

  // ########## Components ##########
  import ArrivalTimeSortedService from "@/components/bus/ArrivalTimeSortedService.vue";
  import PlannnedTimeSortedService from "@/components/bus/PlannnedTimeSortedService.vue";
  // ########## Components ##########

  // ########## SEO ##########
  const route = useRoute();

  useHead({
    title: `${route.meta.title}｜わかめナビ🌱`,

    meta: [
      { hid: "og:title", property: "og:title", content: `${route.meta.title}｜わかめナビ🌱` },
    ]
  });

  definePageMeta({
    name: "BusPage",
    title: "バス検索"
  });
  // ########## SEO ##########

  // ########## Constants ##########
  const __INTERVALS = [
    { label: "埼玉大学 → 北浦和駅", value: "SaitamaUniv-KitaUrawa" },
    { label: "埼玉大学 → 南与野駅", value: "SaitamaUniv-MinamiYono" },
    { label: "埼玉大学 → 志木駅", value: "SaitamaUniv-Shiki" },
    { label: "埼玉大学 → 北朝霞駅", value: "SaitamaUniv-KitaAsaka" },

    { label: "北浦和駅 → 埼玉大学", value: "KitaUrawa-SaitamaUniv" },
    { label: "南与野駅 → 埼玉大学", value: "MinamiYono-SaitamaUniv" },
    { label: "志木駅 → 埼玉大学", value: "Shiki-SaitamaUniv" },
    { label: "北朝霞駅 → 埼玉大学", value: "KitaAsaka-SaitamaUniv" },

    { label: "南与野駅北入口 → 埼玉大学", value: "MinamiYonoKita-SaitamaUniv" },
    { label: "南与野駅北入口 → 北浦和駅", value: "MinamiYonoKita-KitaUrawa" },

    { label: "埼大裏 → 浦和駅", value: "SaitamaUnivUra-Urawa" },
    { label: "桜区役所 → 浦和駅", value: "SakuraWardOffice-Urawa" },

    { label: "浦和駅 → 埼大裏", value: "Urawa-SaitamaUnivUra" },
    { label: "浦和駅 → 桜区役所", value: "Urawa-SakuraWardOffice" }
  ]
  // ########## Constants ##########

  // ########## Reactives ##########
  const intervalSelector = ref({
    selected: ""
  });

  const fetchResultTabs = ref({
    selected: 0
  });

  const services = ref<Bus.Service[]>([]);
  const loading = ref<boolean>(false);

  /*services.value = [
    {
      arrivalTime: '15:11',
      companyCode: 'KokusaiKogyo',
      delay: 16,
      destination: '南与野駅西口',
      location: 6,
      plannedTime: '14:54',
      route: '志03-3'
    },

    {
      arrivalTime: '15:08',
      companyCode: 'KokusaiKogyo',
      delay: 0,
      destination: '南与野駅西口',
      location: 5,
      plannedTime: '15:08',
      route: '北朝02'
    }
  ]*/
  // ########## Reactives ##########

  // ########## Computed ##########
  const arrivalTimeSortedServices = computed(() => {
    return Array.from(services.value).sort((a, b) => {
      if ((a.arrivalTime) < (b.arrivalTime)) return -1;
      if ((a.arrivalTime) > (b.arrivalTime)) return 1;
      return 0;
    });
  });

  const plannedTimeSortedServices = computed(() => {
    return Array.from(services.value).sort((a, b) => {
      if ((a.plannedTime) < (b.plannedTime)) return -1;
      if ((a.plannedTime) > (b.plannedTime)) return 1;
      return 0;
    });
  });
  // ########## Computed ##########

  // ########## Methods ##########
  async function fetchServices (interval: string) {
    const [ start, goal ] = interval.split("-");
    loading.value = true;
    services.value = await fetch(`/api/v1/bus/services?start=${start}&goal=${goal}`).then(res => res.json());
    loading.value = false;
  }

  function loadInterval () {
    console.log(storage.data);
    intervalSelector.value.selected = storage.get("interval");
  }

  function saveInterval (value: string) {
    storage.set("interval", value);
  }
  // ########## Methods ##########

  onMounted(() => {
    loadInterval();
  });
</script>