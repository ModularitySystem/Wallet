<template>
  <div class="new_consensus bg-gray">

    <div class="bg-white">
      <div class="w1200">
        <BackBar :backTitle="$t('nav.consensus')"></BackBar>
        <h3 class="title">{{$t('newConsensus.newConsensus0')}}</h3>
      </div>
    </div>

    <div class="w1200 bg-white mt_20">
      <el-form :model="createrForm" :rules="createrRules" ref="createrForm">
        <el-form-item :label="$t('public.createAddress')">
          <el-input v-model.trim="addressInfo.address" disabled>
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('public.rewardAddress')" prop="rewardAddress">
          <el-input v-model.trim="createrForm.rewardAddress" maxlength="50">
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('public.packingAddress')" prop="blockAddress">
          <el-input v-model.trim="createrForm.blockAddress" maxlength="50">
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('public.deposit') + '(NULS)'" prop="amount">
          <span class="balance font12 fr">{{$t('consensus.consensus2')}}: {{addressInfo.balance}}</span>
          <el-input v-model.trim="createrForm.amount">
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('public.commission') + '(%)'" prop="rate">
          <el-input v-model.trim="createrForm.rate">
          </el-input>
        </el-form-item>
        <div class="font14">
          <el-tooltip placement="top">
            <div slot="content">{{$t('transfer.transfer5')}}</div>
            <i class="el-icon-warning"></i>
          </el-tooltip>
          {{$t('public.fee')}}: 0.01 <span class="fCN">NULS</span>
        </div>
        <el-form-item class="form-next">
          <el-button type="success" @click="submitForm('createrForm')">{{$t('password.password3')}}</el-button>
        </el-form-item>
      </el-form>
    </div>
    <Password ref="password" @passwordSubmit="passSubmit">
    </Password>
    <el-dialog :title="$t('newConsensus.newConsensus1')" :visible.sync="newConsensusVisible" width="40rem" class="confirm-dialog">
      <div class="bg-white">
        <div class="div-data">
          <p>{{$t('public.createAddress')}}:&nbsp;</p>
          <label>{{addressInfo.address}}</label>
        </div>
        <div class="div-data">
          <p>{{$t('public.rewardAddress')}}:&nbsp;</p>
          <label>{{createrForm.rewardAddress}}</label>
        </div>
        <div class="div-data">
          <p>{{$t('public.packingAddress')}}:&nbsp;</p>
          <label>{{createrForm.blockAddress}}</label>
        </div>
        <div class="div-data">
          <p>{{$t('public.commission')}}:&nbsp;</p>
          <label class="yellow">{{createrForm.rate}}% <span class="fCN">NULS</span></label>
        </div>
        <div class="div-data">
          <p>{{$t('public.fee')}}:&nbsp;</p>
          <label>0.001 <span class="fCN">NULS</span></label>
        </div>
        <div class="div-data">
          <p>{{$t('public.deposit')}}:&nbsp;</p>
          <label class="yellow">{{createrForm.amount}} <span class="fCN">NULS</span></label>
        </div>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newConsensusVisible = false">{{$t('password.password2')}}</el-button>
        <el-button type="success" @click="confiremSubmit">{{$t('transfer.transfer8')}}</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import nuls from 'nuls-sdk-js'
  import {getNulsBalance, inputsOrOutputs, validateAndBroadcast} from '@/api/requestData'
  import {Times} from '@/api/util'
  import Password from '@/components/PasswordBar'
  import BackBar from '@/components/BackBar'

  export default {
    data() {
      let checkRewardAddress = (rule, value, callback) => {
        if (!value) {
          return callback(new Error(this.$t('newConsensus.newConsensus2')));
        } else {
          callback();
        }
      };
      let checkBlockAddress = (rule, value, callback) => {
        if (!value) {
          return callback(new Error(this.$t('newConsensus.newConsensus3')));
        } else if (value === this.addressInfo.address) {
          return callback(new Error(this.$t('newConsensus.newConsensus4')));
        } else {
          callback();
        }
      };
      let checkAmount = (rule, value, callback) => {
        let re = /^\d+(?=\.{0,1}\d+$|$)/;
        let res = /^\d{1,8}(\.\d{1,8})?$/;
        let balance = this.balanceInfo.balance - value * 100000000;
        if (!value) {
          return callback(new Error(this.$t('newConsensus.newConsensus5')));
        } else if (!re.exec(value) || !res.exec(value)) {
          callback(new Error(this.$t('newConsensus.newConsensus6')));
        } else if (balance < 0.001) {
          callback(new Error(this.$t('newConsensus.newConsensus7')));
        } else if (value < 20000 || value > 500000) {
          callback(new Error(this.$t('newConsensus.newConsensus8')));
        } else {
          callback();
        }
      };
      let checkRate = (rule, value, callback) => {
        let re = /^\d+(?=\.{0,1}\d+$|$)/;
        let res = /^\d{1,8}(\.\d{1,8})?$/;
        if (!value) {
          return callback(new Error(this.$t('newConsensus.newConsensus9')));
        } else if (!re.exec(value) || !res.exec(value)) {
          callback(new Error(this.$t('newConsensus.newConsensus10')));
        } else if (value < 10 || value > 100) {
          callback(new Error(this.$t('newConsensus.newConsensus11')));
        } else {
          callback();
        }
      };
      return {
        addressInfo: {},//账户信息
        balanceInfo: {},//账户余额信息
        //创建节点表单
        createrForm: {
          rewardAddress: '',
          blockAddress: '',
          amount: '',
          rate: '',
        },
        createrRules: {
          rewardAddress: [
            {validator: checkRewardAddress, trigger: ['blur', 'change']},
          ],
          blockAddress: [
            {validator: checkBlockAddress, trigger: ['blur', 'change']},
          ],
          amount: [
            {validator: checkAmount, trigger: ['blur', 'change']}
          ],
          rate: [
            {validator: checkRate, trigger: ['blur', 'change']}
          ],
        },
        newConsensusVisible: false,//创建节点确认弹框
      };
    },
    created() {
      this.addressInfo = JSON.parse(sessionStorage.getItem(sessionStorage.key(0)));
      setInterval(() => {
        this.addressInfo = JSON.parse(sessionStorage.getItem(sessionStorage.key(0)));
      }, 500);
    },
    mounted() {
      this.getBalanceByAddress(this.addressInfo.address)
    },
    watch: {
      addressInfo(val, old) {
        if (val) {
          if (val.address !== old.address && old.address) {
            this.getBalanceByAddress(this.addressInfo.address)
          }
        }
      }
    },
    components: {
      Password,
      BackBar,
    },
    methods: {

      /**
       * 创建节点提交
       * @param formName
       **/
      async submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.newConsensusVisible = true;
          } else {
            return false;
          }
        });
      },

      /**
       * 获取账户余额
       * @param address
       **/
      getBalanceByAddress(address) {
        getNulsBalance(address).then((response) => {
          if (response.success) {
            this.balanceInfo = response.data;
          } else {
            this.$message({message: this.$t('public.err2') + response, type: 'error', duration: 1000});
          }
        }).catch((error) => {
          this.$message({message: this.$t('public.err3') + error, type: 'error', duration: 1000});
        });
      },

      /**
       *  确定框确定提交
       **/
      confiremSubmit() {
        this.$refs.password.showPassword(true)
      },

      /**
       *  获取密码框的密码
       * @param password
       **/
      async passSubmit(password) {
        let transferInfo = {
          fromAddress: this.addressInfo.address,
          assetsChainId: 2,
          assetsId: 1,
          amount: Number(Times(this.createrForm.amount, 100000000).toString()),
          fee: 100000
        };
        let inOrOutputs = await inputsOrOutputs(transferInfo, this.balanceInfo, 4);
        let txhex = '';
        if (inOrOutputs.success) {
          let agent = {
            agentAddress: this.addressInfo.address,
            packingAddress: this.createrForm.blockAddress,
            rewardAddress: this.createrForm.rewardAddress,
            commissionRate: Number(this.createrForm.rate),
            deposit: Number(Times(this.createrForm.amount, 100000000).toString())
          };
          let tAssemble = await nuls.transactionAssemble(inOrOutputs.data.inputs, inOrOutputs.data.outputs, '', 4, agent);
          txhex = await nuls.transactionSerialize(nuls.decrypteOfAES(this.addressInfo.aesPri, password), this.addressInfo.pub, tAssemble);
        } else {
          this.$message({message: this.$t('public.err1') + inOrOutputs.data, type: 'error', duration: 1000});
        }
        //console.log(txhex);
        //验证并广播交易
        await validateAndBroadcast(txhex).then((response) => {
          if (response.success) {
            //console.log(response.hash);
            this.$router.push({
              name: "txList"
            })
          } else {
            this.$message({message: this.$t('public.err') + response.data, type: 'error', duration: 1000});
          }
        }).catch((err) => {
          this.$message({message: this.$t('public.err0') + err, type: 'error', duration: 1000});
        });
      }
    }
  }
</script>

<style lang="less">
  @import "./../../assets/css/style";

  .new_consensus {
    .mt_20 {
      border: @BD1;
      .el-form {
        width: 600px;
        margin: 50px auto;
        .el-form-item {
          .el-form-item__content {
            .el-select {
              width: 100%;
              .el-input__inner {
                width: 100%;
              }
            }
          }
        }
      }
    }

  }
</style>
