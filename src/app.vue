<template>
  <div class="container">
  </div>
</template>

<script>
import { defineComponent } from 'vue';


export default defineComponent({
  components: {
    draggable,
  },
  data(): VueData {
    console.log('app init');
    const c = defaultConfig();
    return {
      loaded: false,
      config: c,
      jobOrder: jobConfigToVueJobData(c.jobOrder),
    };
  },
  created() {
    loadConfigFromOverlayPlugin().then((c) => {
      [{ jobID: 1 }, { jobID: 2 }].sort(sortByJobID);
      this.jobOrder = jobConfigToVueJobData(c.jobOrder);
      this.config = c;
      setTimeout(() => {
        this.loaded = true;
      }, 100);
    });
  },
  watch: {
    config: {
      handler() {
        if (this.loaded) {
          configChange(this.$data);
        }
      },
      deep: true,
    },
    jobOrder() {
      if (this.loaded) {
        configChange(this.$data);
      }
    },
  },
  computed: {
    dragOptions() {
      return {
        animation: 250,
        group: 'people',
        disabled: false,
        ghostClass: 'ghost',
      };
    },
  },
  methods: {
    jobIDToClass(job: string) {
      const id = util.jobEnumToJob(parseInt(job));

      if (util.isTankJob(id)) {
        return ['list-group-item-primary'];
      }

      if (util.isHealerJob(id)) {
        return ['list-group-item-success'];
      }

      if (util.isDpsJob(id)) {
        return ['list-group-item-danger'];
      }

      return [];
    },
  },
});

function jobConfigToVueJobData(jobOrder: Record<string, number>): vueJobData[] {
  return Object.entries(jobOrder)
    .map(([id, order]) => ({ id, order, name: jobIDToCN[id] }))
    .sort((a, b) => a.order - b.order);
}

console.log(jobConfigToVueJobData(defaultConfig().jobOrder));

function VueJobDataToJobConfig(jobOrder: vueJobData[]): Record<string, number> {
  return Object.fromEntries(jobOrder.map((x, i) => [x.id, i]));
}

async function configChange(v: VueData) {
  const data = JSON.parse(JSON.stringify(v.config));
  data.jobOrder = VueJobDataToJobConfig(v.jobOrder);

  await callOverlayHandler({ call: 'saveData', key: overlayPluginKey, data });
  await callOverlayHandler({ call: 'cactbotReloadOverlays' });
  console.log('data change', JSON.stringify(data));
}
</script>
