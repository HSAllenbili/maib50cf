<template>
  <n-config-provider :theme="theme" :locale="zhCN" :date-locale="dateZhCN">
    <n-global-style />
    <n-layout embedded content-style="padding: 24px 24px 0 24px;">
      <n-space vertical>
        <n-select v-model:value="optionvalue" :options="options" />
        <n-input v-model:value="input" type="text" :placeholder="optionvalue" />
        <n-button :onclick="compute">查询拟合b50</n-button>
        <br>
        B50 {{ b35 + b15 }}<br>B35 {{ b35 }}
        <n-data-table :columns="columns" :data="b35table" :bordered="false" />
        B15 {{ b15 }}
        <n-data-table :columns="columns" :data="b15table" :bordered="false" />
      </n-space>
    </n-layout>
  </n-config-provider>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref } from 'vue'
import { useOsTheme, darkTheme } from 'naive-ui'
import { zhCN, dateZhCN } from 'naive-ui'
import axios from 'axios'
const osThemeRef = useOsTheme();
const theme = computed(() => (osThemeRef.value === 'dark' ? darkTheme : null));
const input = ref("")
const optionvalue = ref("")
const b15 = ref(0)
const b35 = ref(0)
const options = ref([{
  label: "QQ号",
  value: 'qq'
},
{
  label: '用户名',
  value: 'username'
}])
var songs: any = []
var lvfit: any = []
const achievemnts = [0, 10, 20, 30, 40, 50, 60, 70, 75, 80, 90, 94, 97, 98, 99, 99.5, 100, 100.5, 101]
const dxk = [0, 1.6, 3.2, 4.8, 6.4, 8, 9.6, 11.2, 12.0, 12.8, 13.6, 16.8, 20, 20.3, 20.8, 21.1, 21.6, 22.4]
const b15table = ref([])
const b35table = ref([])
const columns = [
  {
    title: 'No',
    key: 'no'
  },
  {
    title: '歌名',
    key: 'name'
  }, {
    title: 'DXRA',
    key: 'dxra'
  }, {
    title: '拟合难度',
    key: 'fit'
  }, {
    title: '达成率',
    key: 'ac'
  }
]

var plate_to_version = {
  '初': 'maimai',
  '真': 'maimai PLUS',
  '超': 'maimai GreeN',
  '檄': 'maimai GreeN PLUS',
  '橙': 'maimai ORANGE',
  '暁': 'maimai ORANGE PLUS',
  '桃': 'maimai PiNK',
  '櫻': 'maimai PiNK PLUS',
  '紫': 'maimai MURASAKi',
  '菫': 'maimai MURASAKi PLUS',
  '白': 'maimai MiLK',
  '雪': 'MiLK PLUS',
  '輝': 'maimai FiNALE',
  '熊': 'maimai でらっくす',
  '華': 'maimai でらっくす PLUS',
  '爽': 'maimai でらっくす Splash',
  '煌': 'maimai でらっくす Splash PLUS',
  '宙': 'maimai でらっくす UNiVERSE',
  '星': 'maimai でらっくす UNiVERSE PLUS',
  '祭': 'maimai でらっくす FESTiVAL',
  '祝': 'maimai でらっくす FESTiVAL PLUS'
}

const computerating = (achieve: number, lv: number) => {
  var index = -1
  for (var i in achievemnts) {
    index = index + 1
    if (achieve < achievemnts[i]) {
      break;
    }
  }
  return Math.floor(achieve * dxk[index - 1] * lv * 0.01)
}

const getmusicinfo = (id: number) => {
  for (let i in songs) {
    if (id == songs[i].id) {
      return { ...songs[i], ...{ 'lvfit': lvfit[id] } }
    }
  }
}

const getSongs = async () => {
  songs = (await axios.get("https://www.diving-fish.com/api/maimaidxprober/music_data")).data
  lvfit = ((await axios.get("https://www.diving-fish.com/api/maimaidxprober/chart_stats")).data)["charts"]
}

const getScores = async () => {
  var scores = []
  var plates = []
  for (const [key, value] of Object.entries(plate_to_version)) {
    plates.push(value)
  }
  var data = {}
  if (optionvalue.value == "qq") data = { "qq": input.value, "version": plates }
  else data = { "username": input.value, "version": plates }
  scores = (await axios.post("https://www.diving-fish.com/api/maimaidxprober/query/plate", data) as any).data.verlist
  var score = []
  for (let i in scores) {
    score.push(scores[i])
  }
  return score
}

const compute = async () => {
  b15.value = 0
  b35.value = 0
  b15table.value = []
  b35table.value = []
  var scores = await getScores();
  var bestold = []
  var bestnew = []
  for (let i in scores) {
    var info = getmusicinfo(scores[i].id)
    var ra = computerating(scores[i].achievements, info.lvfit[(scores[i].level_index)].fit_diff)
    if (info.basic_info.is_new) {
      bestnew.push({ ...info, ...{ 'dxra': ra }, ...scores[i] })
    } else {
      bestold.push({ ...info, ...{ 'dxra': ra }, ...scores[i] })
    }
  }
  bestnew = [...new Set(bestnew)]
  bestold = [...new Set(bestold)]
  bestnew.sort((a, b) => b.dxra - a.dxra);
  bestold.sort((a, b) => b.dxra - a.dxra);
  console.log(bestnew)
  console.log(bestold)
  for (let index = 0; index < 15; index++) {
    b15.value = b15.value + bestnew[index].dxra;
    (b15table.value as any).push({ "no": index + 1, "name": bestnew[index].title, "dxra": bestnew[index].dxra, "fit": (bestnew[index].lvfit[bestnew[index].level_index].fit_diff).toFixed(2), "ac": bestnew[index].achievements })
  }
  for (let index = 0; index < 35; index++) {
    b35.value = b35.value + bestold[index].dxra;
    (b35table.value as any).push({ "no": index + 1, "name": bestold[index].title, "dxra": bestold[index].dxra, "fit": (bestold[index].lvfit[bestold[index].level_index].fit_diff).toFixed(2), "ac": bestold[index].achievements })
  }
}

onMounted(() => {
  getSongs()
})
</script>



<style>
::-webkit-scrollbar {
  width: 0;
}
</style>