<template>
  <div class="container container-margin-top">
    <list-title :name="$t('ontIdDetail.nickname')"></list-title>
    <detail-title :name="$t('ontIdDetail.name')" :val="$route.params.ontid"></detail-title>

    <!-- Owners info -->
    <div class="row" v-if="haveData && Ddo.Owners">
      <div class="col">
        <div class="detail-col">
          <p class="font-blod">{{ $t('ontIdDetail.owner') }}</p>
          <div class="row" v-for="owner in Ddo.Owners">
            <div class="col">
              <div class="font-size14 font-Regular normal_color txt-overflow">
                <p><span class="font-blod">Type: </span>{{owner.Type}}</p>
              </div>
              <div class="font-size14 font-Regular normal_color txt-overflow">
                <p><span class="font-blod">Curve: </span>{{owner.Curve}}</p>
              </div>
              <div class="font-size14 font-Regular normal_color txt-overflow">
                <p><span class="font-blod">Value: </span>{{owner.Value}}</p>
              </div>
              <div class="font-size14 font-Regular normal_color txt-overflow">
                <p><span class="font-blod">PublicKeyId: </span>{{owner.PubKeyId}}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="row" v-if="haveData && JSON.stringify(Ddo.Attributes) !== '[]'">
      <div class="col">
        <div class="detail-col">
          <p>Claim Metadata</p>
          <div class="row" v-for="(claim, key) in Ddo.Attributes">
            <div class="col" v-if="claim.Claim">
              <div class="font-size14 font-Regular normal_color"><p>Claim Hash: {{claim.Claim.ClaimId}}</p></div>
              <div class="font-size14 font-Regular normal_color"><p>Claim Context: {{claim.Claim.ClaimContext}}</p></div>
              <div class="font-size14 font-Regular normal_color"><p>Context Desc: {{claim.Claim.ContextDesc}}</p></div>
              <div class="font-size14 font-Regular normal_color" @click="toOntIdDetailPage(claim.Claim.IssuerOntId)">
                <p>Issuer: <span class="important_color pointer">{{claim.Claim.IssuerOntId}}</span></p>
              </div>
              <div v-if="claim.SelfDefined">
                <p class="font-size14 font-Regular normal_color">Claim SelfDefined: {{claim.SelfDefined}}</p>
              </div>
            </div>

            <div class="col" v-if="claim.SelfDefined">
              <p class="font-size14 font-Regular normal_color">{{ claim.SelfDefined }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="row" v-show="!haveData">
      <div class="col">
        <div class="detail-col">{{ $t('ontIdDetail.failed') }}</div>
      </div>
    </div>

    <!-- Events on this ONT ID -->
    <div class="row" v-if="TxnTotal">
      <div class="col">
        <div class="detail-col">
          <span class="font-blod">{{ $t('ontIdDetail.events') }}</span>
          <div class="table-responsive">
            <table class="table ">
              <thead>
              <tr>
                <th class="td-tx-head font-size18 normal_color font-Blod">{{ $t('all.hash') }}</th>
                <th class="td-tx-head font-size18 normal_color font-Blod">{{ $t('all.content') }}</th>
                <th class="td-tx-head font-size18 normal_color font-Blod">{{ $t('all.fee') }}</th>
                <th class="td-tx-head font-size18 normal_color font-Blod">{{ $t('all.time') }}</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="tx in TxnList">
                <td class="font-size14 important_color font-Regular pointer" @click="toTransactionDetailPage(tx.TxnHash)">
                  {{tx.TxnHash.substr(0,4) + '...' + tx.TxnHash.substr(60)}}
                </td>
                <td class="font-size14 font-Regular normal_color ">
                  {{getOntIDEvent(tx.Description)}}
                </td>
                <td class="font-size14 font-Regular normal_color">
                  {{tx.Fee}}
                </td>
                <td class="font-size14 font-Regular normal_color">
                  {{$HelperTools.getTransDate(tx.TxnTime)}}
                </td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {mapState} from 'vuex'

  export default {
    data() {
      return {
        Ddo: {},
        claimflag: true,
        TxnList: {},
        TxnTotal: '',
        haveData: true
      }
    },
    created() {
      this.getOntIdDetail()
    },
    watch: {
      '$route': 'getOntIdDetail',
      'OntIdDetail': function () {
        console.log(this.OntIdDetail)
        if (this.OntIdDetail.info === false) {
          this.haveData = false
        } else {
          this.Ddo = this.OntIdDetail.Ddo
          this.TxnList = this.OntIdDetail.TxnList
          this.TxnTotal = this.OntIdDetail.TxnTotal
        }
      }
    },
    computed: {
      ...mapState({
        OntIdDetail: state => state.OntIDs.Detail,
      })
    },
    methods: {
      getOntIdDetail() {
        this.$store.dispatch('GetOntIdDetail', this.$route.params).then()
      },
      toReturn() {
        this.$router.go(-1)
      },
      toTransactionDetailPage($TxnId) {
        if (this.$route.params.net == undefined) {
          this.$router.push({name: 'TransactionDetail', params: {txnHash: $TxnId}})
        } else {
          this.$router.push({name: 'TransactionDetailTest', params: {txnHash: $TxnId, net: "testnet"}})
        }
      },
      toOntIdDetailPage($OntId) {
        if (this.$route.params.net == undefined) {
          this.$router.push({name: 'OntIdDetail', params: {ontid: $OntId}})
        } else {
          this.$router.push({name: 'OntIdDetailTest', params: {ontid: $OntId, net: "testnet"}})
        }
      },
      getOntIDEvent: function ($event) {
        switch ($event.substr(0, 12)) {
          case "register Ont":
            return "Register ONT ID"
          case "add publicKe":
            return "Add publickey"
          case "remove publi":
            return "Remove publickey"
          case "add attribut":
            return "Add identity attribute"
          case "update attri":
            return "Update identity attribute"
          case "delete attri":
            return "Delete identity attribute"
          case "change recov":
            return "Change recovery"
          case "add recovery":
            return "Add recovery"
          case "remove attri":
            return "Remove attribute"
        }
      }
    }
  }
</script>

<style scoped>
</style>
