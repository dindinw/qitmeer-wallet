<template>
  <el-container>
    <el-header class="cheader">
      <el-row type="flex" justify="space-between">
        <el-col :span="4">
          <h2>交易记录</h2>
        </el-col>
        <el-col :span="20">
          <el-row>
            <el-col :span="2">
              <span style="color:#303133;font-weight:600;">账户：</span>
            </el-col>
            <el-col :span="4">
              <el-select v-model="currentAccount" placeholder="账号" size="small">
                <el-option
                  v-for="item in accounts"
                  :key="item.account"
                  :label="item.account"
                  :value="item.account"
                ></el-option>
              </el-select>
            </el-col>
            <el-col :span="2">
              <span style="color:#303133;font-weight:600;">地址：</span>
            </el-col>
            <el-col :span="14">
              <el-select v-model="currentAddress" style="width:100%">
                <el-option
                  v-for="item in addresses"
                  :key="item.addr"
                  :label="item.addr"
                  :value="item.addr"
                ></el-option>
              </el-select>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </el-header>

    <el-main class="cmain">
      <el-table :data="txList">
        <el-table-column prop="txId" label="txId" width="470"></el-table-column>
        <el-table-column prop="type" label="in/out" width="60"></el-table-column>
        <el-table-column prop="amount" label="金额"></el-table-column>
      </el-table>
    </el-main>
  </el-container>
</template>
<style>
</style>

<script>
export default {
  data() {
    // var txlist = [
    //   {
    //     type: "in",
    //     from: "asdfasdfasdfasdf",
    //     to: "asdfasdfasdfasdf",
    //     amount: 34234,
    //     date: "2019-07-23 23:32:23"
    //   }
    // ];

    return {
      txList: [],
      accounts: [],
      addresses: ["*"],
      currentAccount: "",
      currentAddress: "*"
    };
  },
  mounted() {
    if (this.$store.state.Accounts.length == 0) {
      this.$router.push("/account");
      return;
    }
    this.accounts = this.$store.state.Accounts;
    this.currentAccount = this.$store.state.Accounts[0].account;
  },
  methods: {
    txlist2table(txlist) {
      let _this = this;
      let tmpTable = [];
      for (let i = 0; i < txlist.length; i++) {
        let inOut = "in";

        let amount = 0;

        let findAddr = false;
        for (let j = 0; j < txlist[i].vout.length; j++) {
          if (txlist[i].vout[j].scriptPubKey.addresses) {
            let addrs = txlist[i].vout[j].scriptPubKey.addresses;
            for (let k = 0; k < addrs.length; k++) {
              if (addrs[k] == this.currentAddress) {
                amount = amount + txlist[i].vout[j].amount;
                findAddr = true;
                break;
              }
            }
            if (findAddr) {
              break;
            }
          }
        }
        console.log(findAddr);

        tmpTable.push({
          date: "",
          type: inOut,
          txId: txlist[i].txid,
          amount: amount
        });
      }
      console.log(tmpTable);
      _this.txList = tmpTable;
    },
    getTxList(addr) {
      if (addr == "*") {
        //todo all account addresses
        return;
      }

      let _this = this;

      _this.$emit("setLoading", true, "");

      _this
        .$axios({
          method: "post",
          data: JSON.stringify({
            id: new Date().getTime(),
            method: "wallet_getTxListByAddr",
            params: [addr, 2, -1, 100]
          })
        })
        .then(response => {
          _this.$emit("setLoading", false, "");
          if (typeof response.data.error != "undefined") {
            _this.$emit("alertResError", response.data.error, () => {});
            return;
          }

          // let tmpTable = [];
          // for (let i = 0; i < response.data.result.length; i++) {
          //   tmpTable.push({ addr: response.data.result[i] });
          // }
          // _this.txList = tmpTable;

          _this.txlist2table(response.data.result.transactions);
        })
        .catch(() => {
          _this.$emit("setLoading", false, "");
        });
    },
    getAddressList(account) {
      let _this = this;
      _this
        .$axios({
          method: "post",
          data: JSON.stringify({
            id: new Date().getTime(),
            method: "wallet_getAddressesByAccount",
            params: [account]
          })
        })
        .then(response => {
          if (typeof response.data.error != "undefined") {
            _this.$emit("alertResError", response.data.error, () => {});
            return;
          }

          let tmpTable = [];
          for (let i = 0; i < response.data.result.length; i++) {
            tmpTable.push({ addr: response.data.result[i] });
          }
          _this.addresses = tmpTable;
          _this.currentAddress = _this.addresses[0].addr;
        });
    }
  },
  watch: {
    currentAccount() {
      this.getAddressList(this.currentAccount);
    },
    currentAddress() {
      this.txList = [];
      this.getTxList(this.currentAddress);
    }
  }
};
</script>