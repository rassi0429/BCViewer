<template lang="pug">
  v-row(justify="center" align="center" )
    v-col(cols="12" sm="8" md="6")
      v-card()
        v-card-title 現在のブロックの高さ
        v-row(align="end" )
          v-col(cols="12" align-self="end" style="text-align:end")
            p.text-h2.text-right.px-3 {{blockchain.length}}
    v-col(cols="12" sm="8" md="6")
      v-card
        v-card-title 書き込み待ちのデータ
        v-row(align="end" )
          v-col(cols="12" align-self="end" v-for="(tra,i) in currentData.split(`\n`)" )
            p.px-3 {{getTransactionDesc(tra)}}
    v-divider
    p ↓ブロックチェーン
    v-col(cols="12")
      v-expansion-panels
        v-expansion-panel(v-for="(item,i) in blockchain.slice().reverse()" :key="i")
          v-expansion-panel-header ID:{{blockchain.length - i}} {{getMineUser(item)}} {{ getTransactionText(getBlockData(item))}}
          v-expansion-panel-content(v-for="(tra,ii) in getBlockData(item)" :key="ii")
            p {{getTransactionDesc(tra)}}

        // {{getBlockData(item)}}
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      blockchain: [],
      currentData: ""
    }
  },
  methods: {
    async getBlockChain() {
      const result = await fetch("https://nftfesta.kokoa.dev/getBlockChain?json=true")
      this.blockchain = await result.json()
    },
    async getCurrentData() {
      const result = await fetch("https://nftfesta.kokoa.dev/getCurrentData")
      this.currentData = await result.text()
    },
    getMineUser(item) {
      const array = item.data.split(":")
      return array[array.length - 1]
    },
    getBlockData(item) {
      const data = item.data.split(":")
      const transactions = data[0].split("\n")
      return transactions
    },
    getTransactionText(item) {
      if (item[0] === "") return "データなし"
      let send = 0
      let msg = 0
      let withdraw = 0
      item.forEach(i => {
        const transaction = i.replace("{", "").replace("}", "").split("$")
        switch (transaction[0]) {
          case "send":
            send += 1
            break;
          case "msg":
            msg += 1
            break;
          case "withdraw":
            withdraw += 1
            break;
        }
      })
      return `送金: ${send} メッセージ: ${msg}  出金: ${withdraw} `
    },
    getTransactionDesc(tra) {
      const transaction = tra.replace("{", "").replace("}", "").split("$")
      let w = ""
      switch (transaction[0]) {
        case "send":
          w = `送金: ${transaction[1]} → ${transaction[2]} : ${transaction[3]}`
          break;
        case "msg":
          w = `メッセージ: ${transaction[1]} : ${transaction[2]}`
          break;
        case "withdraw":
          w = `出金: ${transaction[1]} : ${transaction[2]}`
          break;
      }
      return w
    }
  },
  mounted() {
    this.getBlockChain()
    this.getCurrentData()
    setInterval(() => {
      this.getBlockChain()
      this.getCurrentData()
    },2000)
  }
}
</script>
