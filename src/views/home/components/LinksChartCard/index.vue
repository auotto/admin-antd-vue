<template>
  <a-spin :spinning="loading" size="large">
      <a-card :title="t('page.home.linkschartcard.card-title')" class="homeBoxCard">
          <template #extra><a-tag color="error">{{t('page.home.text-years')}}</a-tag></template>
          <div class="num">{{num.toLocaleString()}}</div>
          <div class="height40" ref="linksChartRef" />
          <a-divider />
          <a-row>
            <a-col :span="12">{{t('page.home.text-total')}}</a-col>
            <a-col class="text-align-right" :span="12">
              {{total.toLocaleString()}}
            </a-col>
          </a-row>
      </a-card>
  </a-spin>
</template>
<script lang="ts">
import { computed, ComputedRef, defineComponent, onMounted, Ref, ref, watch } from "vue";
import { useStore } from "vuex";
import { useI18n } from "vue-i18n";
import { EChartOption } from 'echarts';
import useEcharts from '@/composables/useEcharts';
import { StateType as HomeStateType } from "../../store";
import { ChartDataType } from "../../data";

const linksChartOption: EChartOption = {
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'shadow',
    },
  },
  grid: {
    left: '0',
    right: '0',
    top: '0',
    bottom: '0',
  },
  xAxis: {
    show: false,
    data: [
      /* '2019-04', '2019-05', '2019-06','2019-07', '2019-08', '2019-09', '2019-10', '2019-11', '2019-12', '2020-01', '2020-02', '2020-03' */
    ],
  },
  yAxis: {
    show: false,
  },
  series: [
    {
      name: '新增',
      type: 'bar',
      data: [
        /* 5888, 3838, 15880, 12888, 18888, 16888,5888, 3838, 15880, 12888, 18888, 16888 */
      ],
    },
  ],
};

interface LinksChartCardSetupData {
    t: (key: string | number) => string;
    loading: Ref<boolean>;
    linksChartRef: Ref;
    total: ComputedRef<number>;
    num: ComputedRef<number>;
}

export default defineComponent({
    name: 'LinksChartCard',
    setup(): LinksChartCardSetupData {
        const store = useStore<{ Home: HomeStateType}>();
        const { t } = useI18n();

        // 总数
        const total = computed<number>(() => store.state.Home.linksChartData.total);
        // num
        const num = computed<number>(() => store.state.Home.linksChartData.num);
        // chart Data
        const chartData = computed<ChartDataType>(()=> store.state.Home.linksChartData.chart);

        // echarts 图表
        const linksChartRef = ref<HTMLDivElement>();
        const echarts = useEcharts(linksChartRef, linksChartOption);      
        watch([echarts, chartData],()=> {
          if(echarts.value) {
              const option: EChartOption = {
                xAxis: {
                  // data: ["03-01", "03-01", "03-01", "03-01", "03-01", "03-01", "03-01"]
                  data: chartData.value.day,
                },
                series: [
                  {
                    name: '新增',
                    // data: [3, 1, 1, 2, 2, 2, 2]
                    data: chartData.value.num,
                  },
                ],
              };              
              echarts.value.setOption(option);
          }
        }); 


        // 读取数据 func
        const loading = ref<boolean>(true);
        const getData = async () => {
            loading.value = true;
            await store.dispatch('Home/queryLinksChartData');
            loading.value = false;
        };

        onMounted(()=> {
           getData();
        });


        return {
            t,
            loading,
            linksChartRef,
            total,
            num
        }
    }
})
</script>
<style lang="less" scoped>
.homeBoxCard {
  margin-bottom: 24px;
  ::v-deep(.ant-card-head) {
    padding-left: 12px;
    padding-right: 12px;
  }
  ::v-deep(.ant-card-body) {
    padding: 12px;
  }
  ::v-deep(.ant-divider-horizontal) {
    margin: 8px 0;
  }
  .num {
    font-size: 30px;
    color: #515a6e;
  }
  .height40 {
    height: 40px;
  }
}
</style>