<template>
  <div class="ant-modal-sun1">
    <div class="ant-modal-mask closeModalMask"></div>
    <div class="ant-modal-wrap closeModalWrap">
      <div class="ant-modal">
        <div class="ant-modal-content">
          <button type="button" class="ant-modal-close" @click="closeModalStake">
            <div class="ant-modal-close-x">
              <img src="../assets/antModalClose.png" alt="">
            </div>
          </button>
          <div class="ant-modal-header">
            <div class="ant-tabs-tab" :class="{activeTab: isShow}" @click="activeTab">Stake</div>
            <div class="ant-tabs-tab claimWithDraw" :class="{activeTab: !isShow}"  @click="activeTab">Claim</div>
          </div>
          <div class="ant-modal-tabs" v-if="isShow">
            <div>
              <p class="sub-text">
                <span>Stake Token</span>
                <span @click="getBalanceOfStake">Get</span>
              </p>
              <div class="input readToken">
                <input type="text" v-model="balanceOfStake" placeholder="0" readonly>
                <span>USDT</span>
              </div>
            </div>
            <div>
              <p class="sub-text">
                <span>Stake USDT</span>
                <span @click="getMAXToken">MAX</span>
              </p>
              <div class="input">
                <input type="text" placeholder="0" v-model="amountToken" id="amountStakeUSDT">
                 <span>USDT</span>
              </div>
              <p>Maximum Stake Token: <span>0</span> SUN</p>
            </div>
            <button class="stakeToken-btn" @click="stakeToken">Stake Token</button>
          </div>
          <div class="ant-modal-tabs" v-else>
            <div>
              <p class="sub-text">
                <span>Pending Claim</span>
                <span>Claim</span>
              </p>
              <div class="input readToken">
                <input type="text" placeholder="0" v-model="amountWithDraw" readonly>
                <span>USDT</span>
              </div>
            </div>
            <ul>
              <li v-for="balanceOfStake in balanceOfEachStake" v-bind:key="balanceOfStake.id">
                <p class="sub-text">
                  <span>Amount Stake: {{ balanceOfStake.amount }} USDT</span>
                  <span>MAX</span>
                </p>
                <div class="unStake-section">
                  <div>
                    <input type="text" placeholder="0" :data-value=balanceOfStake.id>
                    <span>USDT</span>
                  </div>
                  <button class="unStake-btn" @click="Unstaking(balanceOfStake.id)">Unstake</button>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
    data(){
      return{
        addressUSDT: 'TUtvDtkoRGcqvVJcYGntox3KT2xy2QaRM5',
        addressBank: 'TGKpYkw4D4TNK3YyXx1z9FWjp9kho7dv6n',
        addressTRX: 'TVu3H7drteoaEDDpfBGyh3ZBtbUPhzbA96',
        balanceOfStake : 0,
        isShow: true,
        amountToken: 0,
        amountTRX: 0,
        amountWithDraw: 0,
        countNumOfStake: 0,
        balanceOfEachStake: []
      }
    },
    methods: {
      activeTab: function(el){
        if(el.target.classList.contains('activeTab')){
          return
        }
        else this.isShow = !this.isShow
        if(el.target.classList.contains('claimWithDraw')){
            this.getWithDraw()
            this.getbalanceOfEachStake()
          }
      },
      closeModalStake: function(){
        let antModalMask = document.querySelector('.ant-modal-sun1 .ant-modal-mask')
        let antModalWrap = document.querySelector('.ant-modal-sun1 .ant-modal-wrap')
        antModalMask.classList.add("closeModalMask")
        antModalWrap.classList.add("closeModalWrap")
      },
      stakeToken: async function() {
        if(this.amountToken === 0)  {
          alert("Invalid value amount token")
        }
        else{
          const trc20ContractAddress = this.addressBank;
          let amountStake = parseInt(document.getElementById('amountStakeUSDT').value) * Math.pow(10,2)
          try {
            let contract = await window.tronWeb.contract().at(trc20ContractAddress)
            await contract.stake(this.addressUSDT,amountStake).send({
                feeLimit: 50000000
            }).then(output => output)
          } catch(error) {
            console.error("trigger smart contract error",error)
          }
        }
      }, 
      Unstaking: async function(id){
        const trc20ContractAddress = this.addressBank;
        let amountUnstake = parseFloat(document.querySelector("[data-value='" + id + "']").value) * Math.pow(10,2)
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress)
          await contract.unstaking(this.addressUSDT,amountUnstake,id-1).send({
              feeLimit: 50000000
          }).then(output => output)
        }catch(error) {
          console.error("trigger smart contract error",error)
        }
      },
      getBalanceOfStake: async function() {
        const trc20ContractAddress = this.addressBank;
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress);
          var result = await contract.methods.balanceOfStake(this.addressUSDT).call()
          this.balanceOfStake = parseInt(result._hex) / Math.pow(10,2)
        } catch(error) {
          console.error("trigger smart contract error",error)
        }
      },
      getMAXToken: async function() {
        const trc20ContractAddress = this.addressUSDT
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress);
          let result = await contract.methods.balanceOf(this.addressTRX).call()
          this.amountToken = parseInt(result.balance._hex) / Math.pow(10,2)
        } catch(error) {
            console.error("trigger smart contract error",error)
        }
      },
      getWithDraw: async function(){
        const trc20ContractAddress = this.addressBank;
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress);
          var result = await contract.methods.interestBalance(this.addressUSDT).call()
          this.amountWithDraw = parseInt(result._hex) / Math.pow(10,2)
        } catch(error) {
          console.error("trigger smart contract error",error)
        }
      },
      getNumOfStake: async function(){
        const trc20ContractAddress = this.addressBank
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress)
          let result = await contract.methods.countNumOfStake(this.addressUSDT).call()
          return parseInt(result._hex)
        } catch(error) {
          console.error("trigger smart contract error",error)
        }
      },
      getbalanceOfEachStake: async function(){
        this.balanceOfEachStake = []
        const trc20ContractAddress = this.addressBank
        try {
          let contract = await window.tronWeb.contract().at(trc20ContractAddress)
          let countNumOfStake = await this.getNumOfStake()
          for(let i = 0; i < countNumOfStake; i++){
            let result = await contract.methods.balanceOfEachStake(this.addressUSDT,i).call()
            let newItem = {
              id: i+1,
              amount: parseInt(result._hex)
            }
            if(newItem.amount !== 0){
              this.balanceOfEachStake.push(newItem)
            }
          }
        } catch(error) {
          console.error("trigger smart contract error",error)
        }
      },
    }
}
</script>
<style lang="scss" scoped>
  .ant-modal-sun1{
    height: 100%;
    .ant-modal-mask{
      position: fixed;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      z-index: 1000;
      width: 100%;
      height: 100%;
      background-color: rgba(0,0,0,.45);
      transition:.5s;
      &.closeModalMask{
        opacity: 0;
        visibility: hidden;
      }
    }
    .ant-modal-wrap{
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: 1000;
      overflow: auto;
      transform: scale(1);
      transition: .35s;
      transform-origin: (50%) (60%);
      &:before{
        background: black;
      }
      &.closeModalWrap{
        transform: scale(0);
        visibility: hidden;
      }
      .ant-modal{
        width: 400px;
        transform-origin: 661px -27px;
        text-align: center;
        top: calc(50vh - 225px);
        background: #fff;
        box-shadow: inset -7px -8px 10px 0 rgba(51,1,138,.21);
        border-radius: 30px;
        box-sizing: border-box;
        color: rgba(0,0,0,.65);
        font-size: 14px;
        font-variant: tabular-nums;
        line-height: 1.5715;
        list-style: none;
        font-feature-settings: "tnum","tnum";
        position: relative;
        top: 100px;
        margin: 0 auto;
        padding: 0 0 24px;
        .ant-modal-content{
          border-radius: 30px 30px 0 0;
          box-shadow: none;
          .ant-modal-close{
            padding-top: 15px;
            position: absolute;
            top: 0;
            right: 0;
            z-index: 10;
            color: rgba(0,0,0,.45);
            font-weight: 700;
            line-height: 1;
            border: 0;
            outline: 0;
            background: transparent;
            .ant-modal-close-x{
              width: 56px;
              height: 56px;
              margin-top: -5px;
              text-align: center;
              cursor: pointer;
              img{
                width: 40px;
              }
            }
          }
          .ant-modal-header{
            display: flex;
            align-items: center;
            justify-content: center;
            padding-top: 20px;
            margin-bottom: 40px;
            .ant-tabs-tab{
              width: 90px;
              padding: 15px 0;
              cursor: pointer;
              margin: 0 10px;
              &.activeTab{
                border-bottom: 2px solid #6726eb;
              }
            }
          }
          .ant-modal-tabs{
            width: 85%;
            margin: 0 auto;
            text-align: left;
            &>div{
              margin-top: 10px;
              p{
                font-size: 11px;
              }
              p:first-child{
                color: #333;
                font-size: 12px;
                font-weight: 600;
              }
              p.sub-text{
                display: flex;
                justify-content: space-between;
                span:last-child{
                  color: #fff;
                  font-size: 8px;
                  font-weight: 500;
                  padding: 4px 10px;
                  background-color: #6726eb;
                  border-radius: 20px;
                  cursor: pointer;
                }
              }
              .input{
                position: relative;
                 input{
                  height: 50px;
                  width: 100%;
                  border-radius: 8px;
                  padding-left: 15px;
                  background-color: #fff;
                  outline: none;
                  border: 1px solid rgb(224, 224, 224);
                  margin-top: 10px;
                  margin-bottom: 10px;
                  &:focus{
                    border: 1px solid #6726eb;
                  }
                }
                &.readToken{
                  input{
                    background-color: rgb(233, 233, 233);
                    border: none;
                    cursor: auto;
                    &:focus{
                      border: none;
                      outline: none;
                    }
                  }
                }
                span{
                  position: absolute;
                  top: 25px;
                  right: 10px;
                }
              }
            }
            ul{
              li{
                list-style-type: none;
                p.sub-text{
                  margin-top: 10px;
                  font-size: 13px;
                  color: #333;
                  display: flex;
                  justify-content: space-between;
                  span:last-child{
                    color: #fff;
                    font-size: 8px;
                    font-weight: 500;
                    padding: 4px 10px;
                    background-color: #6726eb;
                    border-radius: 20px;
                    cursor: pointer;
                    margin-right: 70px;
                  }
                }
                div.unStake-section{
                  display: flex;
                  div{
                    position: relative;
                    width: 80%;
                    input{
                      height: 30px;
                      width: 100%;
                      border-radius: 8px;
                      padding-left: 15px;
                      background-color: #fff;
                      outline: none;
                      border: 1px solid rgb(224, 224, 224);
                      margin-top: 10px;
                      margin-bottom: 10px;
                      &:focus{
                        border: 1px solid #6726eb;
                      }
                    }
                    span{
                      position: absolute;
                      right: 5px;
                      top: 15px;
                      font-size: 13px;
                    }
                  }
                  button{
                    margin-left: 10px;
                    height: 20px;
                    margin-top: 15px;
                    font-size: 12px;
                    padding: 2px 5px;
                    border-radius: 5px;
                    color: #fff;
                    background: #6726eb;
                    cursor: pointer;
                  }
                }
              }
            }
          }
          .stakeToken-btn{
            display: block;
            width: 100%;
            height: 40px;
            margin: 70px 0 10px 0;
            border-radius: 10px;
            border: 0;
            background: #6726eb;
            cursor: pointer;
            color: #fff;
            outline: none;
          }
        }
      }
    }
  }
</style>