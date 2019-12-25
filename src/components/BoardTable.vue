<template>
  <div class="container">
    <div v-if="this.isLoading" class="progress" id="loader">
      <div class="indeterminate"></div>
    </div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <td
              colspan="4"
              style="textAlign: left,
                    backgroundColor: rgb(20, 24, 32)"
            >Table</td>
          </tr>
        </thead>
        <tbody>
          <tr>
            <th scope="col">サイズ</th>
            <th scope="col">価格</th>
            <th scope="col">サイズ</th>
            <th scope="col">%</th>
          </tr>
          <tr v-for="(val, index) in askData" :key="'ask' + index">
            <td
              scope="row"
              style="paddingLeft: 0; paddingRight: 0"
            >
              <div
                class="bar"
                :style="{
                          width: Math.min(val / -20000, 100) + '%',
                          backgroundColor: '#1C9FA0'}"
              ></div>
            </td>
            <td
              :class="{increase: isPrevState && askPriceChange[index] > 0, decrease: isPrevState && askPriceChange[index] < 0}"
            >{{askDataLabel[index].toFixed(2)}}</td>
            <td
              :class="{increase: isPrevState && askSizeChange[index] > 0, decrease: isPrevState && askSizeChange[index] < 0}"
            >{{getDigitsP(-val)}}</td>
            <td
              :class="{increase: isPrevState && askPercentageChange[index] > 0, decrease: isPrevState && askPercentageChange[index] < 0}"
            >{{getPecentage(askDataLabel[index]) + '%'}}</td>
          </tr>
          <tr>
            <td
              colspan="4"
              :style="{
                    textAlign: 'center', 
                    color: tickDirection > 0 ? '#1C9FA0' : '#E54646'}"
            >
              {{middle_price}}
              <i :class="'fa ' + gettickDirectionState()"></i>
            </td>
          </tr>
          <tr v-for="(val, index) in bidData" :key="'bid' + index">
            <td scope="row" style="paddingLeft: 0; paddingRight: 0">
              <div
                class="bar"
                :style="{
                          width: Math.min(val / -20000, 100) + '%',
                          backgroundColor: '#E54646'}"
              ></div>
            </td>
            <td
              :class="{increase: isPrevState && bidPriceChange[index] > 0, decrease: isPrevState && bidPriceChange[index] < 0}"
            >{{bidDataLabel[index].toFixed(2)}}</td>
            <td
              :class="{increase: isPrevState && bidSizeChange[index] > 0, decrease: isPrevState && bidSizeChange[index] < 0}"
            >{{getDigitsP(-val)}}</td>
            <td
              :class="{increase: isPrevState && bidPercentageChange[index] > 0, decrease: isPrevState && bidPercentageChange[index] < 0}"
            >{{getPecentage(bidDataLabel[index]) + '%'}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: "BoardTable",
  data() {
    return {
      middle_price: "",
      tickDirection: 0,
      askData: [],
      askDataLabel: [],
      askPriceChange: [],
      askSizeChange: [],
      askPercentageChange: [],
      bidData: [],
      bidDataLabel: [],
      bidPriceChange: [],
      bidSizeChange: [],
      bidPercentageChange: [],
      isLoading: true,
      isPrevState: false
    };
  },
  mounted() {
    let that = this;
    let wsBitMEXTrade = new WebSocket("wss://www.bitmex.com/realtime");
    let wsBitMEX = new WebSocket("wss://www.bitmex.com/realtime");
    wsBitMEXTrade.onopen = function() {
      wsBitMEXTrade.send(
        JSON.stringify({ op: "subscribe", args: ["trade:XBTUSD"] })
      );
    };

    wsBitMEXTrade.onmessage = function(msg) {
      let response = JSON.parse(msg.data);
      if (response.data) {
        let middle_price = response.data[0].price.toFixed(0);
        let tickDirection;
        if (!that.isLoading) {
          switch (response.data[0].tickDirection) {
            case "PlusTick":
              tickDirection = 2;
              break;
            case "ZeroPlusTick":
              tickDirection = 1;
              break;
            case "MinusTick":
              tickDirection = -1;
              break;
            case "ZeroMinusTick":
              tickDirection = -2;
              break;
            default:
              tickDirection = 0;
              break;
          }
        }
        that.middle_price = middle_price;
        that.tickDirection = tickDirection;
      }
    };

    wsBitMEX.onopen = function() {
      wsBitMEX.send(
        JSON.stringify({ op: "subscribe", args: ["orderBook10:XBTUSD"] })
      );
    };

    wsBitMEX.onmessage = function(msg) {
      var response = JSON.parse(msg.data);
      if (response.data) {
        let askDataLabel = response.data[0].asks.map(x => x[0]);
        let askData = response.data[0].asks.map(x => -x[1]);
        askDataLabel.reverse();
        askData.reverse();
        let bidDataLabel = response.data[0].bids.map(x => x[0]);
        let bidData = response.data[0].bids.map(x => -x[1]);
        if (!this.isLoading) {
          that.isPrevState = true;
          for (let i = 0; i < askData.length; i++) {
            that.askPriceChange[i] = askDataLabel[i] - that.askDataLabel[i];
            that.askSizeChange[i] = -(askData[i] - that.askData[i]);
            that.askPercentageChange[i] =
              that.getPecentage(askDataLabel[i]) -
              that.getPecentage(that.askDataLabel[i]);
            that.bidPriceChange[i] = bidDataLabel[i] - that.bidDataLabel[i];
            that.bidSizeChange[i] = -(bidData[i] - that.bidData[i]);
            that.bidPercentageChange[i] =
              that.getPecentage(bidDataLabel[i]) -
              that.getPecentage(that.bidDataLabel[i]);
          }
        }
        if (that.isLoading) {
          that.isLoading = false;
        }
        that.askData = askData;
        that.askDataLabel = askDataLabel;
        that.bidData = bidData;
        that.bidDataLabel = bidDataLabel;
      }
    };
  },
  methods: {
    getDigitsP: function(digits) {
      let digLen = digits.toString().length;
      switch (digLen) {
        case 5:
          return `${digits}          `;
        case 6:
          return `${digits}        `;
        case 7:
          return `${digits}      `;
        case 8:
          return `${digits}    `;
        case 9:
          return `${digits}  `;
        case 10:
          return `${digits}`;
        default:
          return `${digits}`;
      }
    },
    gettickDirectionState: function() {
      switch (this.tickDirection) {
        case 2:
          return "fa-caret-up";
        case 1:
          return "fa-arrow-up";
        case -1:
          return "fa-arrow-down";
        case -2:
          return "fa-caret-down";
        default:
          return "";
      }
    },
    getPecentage: function(price) {
      return (((price - this.middle_price) / this.middle_price) * 100).toFixed(
        2
      );
    }
  }
};
</script>

<style>
table.striped {
  border: 1px dashed rgb(49, 49, 52);
}

table.striped > thead > tr > th,
table.striped > tbody > tr > td,
table.striped > tbody > tr > th {
  border: 1px dashed rgb(49, 49, 52);
  padding: 0.3em;
  width: 40px;
  text-align: right;
}
table.striped > tbody > tr:nth-child(odd) > td,
table.striped > tbody > tr:nth-child(odd) > th {
  background-color: rgb(38, 38, 38);
}
table.striped > tbody > tr:nth-child(even) > td,
table.striped > tbody > tr:nth-child(even) > th {
  background-color: rgb(20, 24, 34);
}
table {
  font-size: 1.5vh !important;
  table-layout: fixed;
  margin: 0 auto !important;
  color: white;
  max-width: 400px;
}
.bar {
  height: 1em;
  float: right;
}
#loader {
  z-index: 1;
}
@keyframes increase {
  from {
    background-color: #1c9fa0;
  }
}
@keyframes decrease {
  from {
    background-color: #e54646;
  }
}
.increase {
  animation: increase 0.5s ease-out;
}
.decrease {
  animation: decrease 0.5s ease-out;
}
</style>