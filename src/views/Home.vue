<template>
  <div class="home">
    <h3 class="title" v-if="addressInfo">
      {{addressInfo.address}}
      <span v-show="addressInfo.alias">({{addressInfo.alias}})</span>
      <i class="iconfont iconfuzhi clicks" @click="copy(addressInfo.address)"></i>
      <i class="iconfont iconerweima clicks" @click="showCode"></i>
    </h3>

    <div class="overview bg-white" v-loading="overviewLoading">
      <el-row>
        <el-col :span="24">
          <div class="title grid-content">
            {{symbol.toLocaleUpperCase()}} {{$t('tab.tab26')}}
            <span
              class="click"
              @click="toUrl('txList',addressNULSAssets)"
            >{{$t('home.home2')}}</span>
          </div>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8">
          <div class="total grid-content">
            <p>{{$t('tab.tab2')}}</p>
            <h6>
              {{addressNULSAssets.total}}
              <span
                class="font16"
                v-show="symbol.toLocaleUpperCase() ==='NULS'"
              >≈ $ {{NULSUsdt}}</span>
            </h6>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="balance grid-content">
            <p>{{$t('public.usableBalance')}}</p>
            <h6>
              <font>{{addressNULSAssets.balance}}</font>
              <el-button
                type="success"
                @click="toUrl('transfer',addressNULSAssets.account)"
              >{{$t('tab.tab31')}}</el-button>
              <el-button @click="showCode">{{$t('tab.tab27')}}</el-button>
            </h6>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="locking grid-content">
            <p>{{$t('tab.tab3')}}</p>
            <h6>
              <font>{{addressNULSAssets.locking}}</font>
              <span
                class="font14 click"
                @click="toUrl('frozenList',addressNULSAssets)"
              >{{$t('tab.tab28')}}</span>
            </h6>
          </div>
        </el-col>
      </el-row>
    </div>
    <div class="assets">
      <el-row>
        <el-col :span="24">
          <div class="grid-content">
            <el-tabs
              type="border-card"
              v-model="homeActive"
              @tab-click="handleClick"
              class="home_tabs"
            >
              <el-tab-pane :label="$t('tab.tab25')" name="homeFirst">
                <el-select v-model="assetsValue" @change="channgeAsesets" v-show="false">
                  <el-option
                    v-for="item in assetsOptions"
                    :key="item.value"
                    :label="$t('assetsType.'+item.value)"
                    :value="item.value"
                  ></el-option>
                </el-select>
                <el-table :data="addressAssetsData" stripe border v-loading="assetsListLoading">
                  <el-table-column :label="$t('nodeService.nodeService2')" align="center">
                    <template slot-scope="scope">
                      <span>{{ scope.row.account }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column :label="$t('contract.contract9')" align="center">
                    <template slot-scope="scope">
                      <span
                        class="click td"
                        @click="toUrl('contractsInfo',scope.row.contractAddress,1)"
                      >{{ scope.row.contractAddresss }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column align="center" prop="balance" :label="$t('tab.tab4')"></el-table-column>
                  <el-table-column align="center" :label="$t('tab.tab3')">
                    <template slot-scope="scope">
                      <span
                        class="click"
                        @click="toUrl('frozenList',scope.row)"
                        v-show="scope.row.locking !== '--' && scope.row.locking !==0 "
                      >{{scope.row.locking}}</span>
                      <span
                        v-show="scope.row.locking === '--' || scope.row.locking ===0"
                      >{{scope.row.locking}}</span>
                    </template>
                  </el-table-column>
                  <el-table-column align="center" prop="total" :label="$t('tab.tab2')"></el-table-column>
                  <el-table-column
                    fixed="right"
                    :label="$t('public.operation')"
                    align="center"
                    min-width="120"
                  >
                    <template slot-scope="scope">
                      <label
                        class="click tab_bn"
                        @click="toUrl('transfer',scope.row)"
                      >{{$t('nav.transfer')}}</label>
                      <span class="tab_line">|</span>
                      <label
                        class="click tab_bn"
                        @click="toUrl('txList',scope.row)"
                      >{{$t('home.home2')}}</label>
                    </template>
                  </el-table-column>
                </el-table>
                <div class="pages" v-show="false">
                  <div class="page-total">
                    {{pageNumber-1 === 0 ? 1 : (pageNumber-1) *pageSize}}-{{pageNumber*pageSize}}
                    of {{addressAssetsData.length}}
                  </div>
                  <el-pagination
                    v-show="addressAssetsData.length > pageSize"
                    class="fr"
                    background
                    @current-change="addressAssetsListPages"
                    layout=" prev, pager, next, jumper"
                    :total="addressAssetsData.length"
                  ></el-pagination>
                </div>
              </el-tab-pane>
              <el-tab-pane :label="$t('home.home1')" name="homeSecond">
                <el-table :data="crossLinkData" stripe border v-loading="crossLinkDataLoading">
                  <el-table-column prop="symbol" :label="$t('tab.tab0')" align="center"></el-table-column>
                  <!--  <el-table-column :label="$t('tab.tab1')" align="center" width="150">
              <template slot-scope="scope"><span>{{ scope.row.symbol }}</span></template>
                  </el-table-column>-->
                  <el-table-column prop="balance" :label="$t('tab.tab4')" align="center"></el-table-column>
                  <el-table-column :label="$t('tab.tab3')" align="center">
                    <template slot-scope="scope">
                      <span
                        class="click"
                        @click="toUrl('frozenList',scope.row)"
                        v-show="scope.row.locking !== '--' && scope.row.locking !==0 "
                      >{{scope.row.locking}}</span>
                      <span
                        v-show="scope.row.locking === '--' || scope.row.locking ===0"
                      >{{scope.row.locking}}</span>
                    </template>
                  </el-table-column>
                  <el-table-column prop="totalBalance" :label="$t('tab.tab2')" align="center"></el-table-column>
                  <el-table-column
                    fixed="right"
                    :label="$t('public.operation')"
                    align="center"
                    min-width="120"
                  >
                    <template slot-scope="scope">
                      <label
                        class="click tab_bn"
                        @click="toUrl('transfer',scope.row.symbol)"
                      >{{$t('nav.transfer')}}</label>
                      <span class="tab_line">|</span>
                      <label
                        class="click tab_bn"
                        @click="toUrl('txList',scope.row)"
                      >{{$t('home.home2')}}</label>
                    </template>
                  </el-table-column>
                </el-table>
                <div class="pages" v-show="addressAssetsData.length > 10">
                  <div class="page-total">
                    {{pageNumber-1 === 0 ? 1 : (pageNumber-1) *pageSize}}-{{pageNumber*pageSize}}
                    of {{crossLinkData.length}}
                  </div>
                  <el-pagination
                    v-show="addressAssetsData.length > pageSize"
                    class="fr"
                    background
                    @current-change="addressAssetsListPages"
                    layout=" prev, pager, next, jumper"
                    :total="addressAssetsData.length"
                  ></el-pagination>
                </div>
              </el-tab-pane>
            </el-tabs>
          </div>
        </el-col>
      </el-row>
    </div>

    <el-dialog title :visible.sync="qrcodeDialog" width="30%" center class="payee_dialog">
      <el-tabs v-model="activeName" @tab-click="payeeHandleClick">
        <el-tab-pane :label="$t('tips.tips12')" name="payeeInfo">
          <el-form :model="payeeForm" class="payee_form">
            <el-form-item label>
              <el-input
                v-model="payeeForm.amount"
                autocomplete="off"
                :placeholder="$t('tips.tips13')"
              ></el-input>
            </el-form-item>
            <el-form-item label>
              <el-select v-model="payeeForm.currency" :placeholder="$t('tips.tips14')">
                <el-option label="NULS" value="NULS"></el-option>
              </el-select>
            </el-form-item>
            <div class="tc">
              <el-button @click="payeeNext(0)">{{$t('tips.tips15')}}</el-button>
              <el-button type="success" @click="payeeNext(1)">{{$t('public.next')}}</el-button>
            </div>
          </el-form>
        </el-tab-pane>
        <el-tab-pane :label="$t('tips.tips16')" name="payeeScan">
          <div id="qrcode" class="qrcode"></div>
          <div class="font12 tc" style="margin: 20px 0 0 0">
            {{$t('tips.tips18')}}
            <font
              class="click td"
              @click="toUrl('nuls','http://nabox.io/',1)"
            >Nabox</font>
            /
            <font
              class="click td"
              @click="toUrl('nuls','https://www.denglu1.cn/',1)"
            >{{$t('tips.tips11')}}</font>
            {{$t('tips.tips21')}}
          </div>
        </el-tab-pane>
      </el-tabs>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
import QRCode from "qrcodejs2";
import {
  timesDecimals,
  copys,
  addressInfo,
  Times,
  superLong,
  connectToExplorer,
  Plus,
} from "@/api/util";
import { RUN_PATTERN } from "@/config";

export default {
  name: "home",
  data() {
    return {
      symbol: "TNX", //symbol
      homeActive: "homeFirst", //tab默认选中
      addressInfo: {}, //默认账户信息
      addressNULSAssets: {}, //账户NULS资产信息
      overviewLoading: true, //nuls资产加载动画
      addressAssetsData: [], //账户资产列表
      NULSUsdt: 0, //nuls美元值
      assetsListLoading: true, //账户资产列表加载动画
      //资产类型
      assetsOptions: [
        { value: "0", label: "0" },
        { value: "1", label: "1" },
        { value: "2", label: "2" },
      ],
      assetsValue: "0",
      txListDataLoading: true, //资产加载动画
      txListData: [], //交易数据
      pageNumber: 1, //页码
      pageSize: 100, //条数
      pageCount: 0, //总条数
      crossLinkData: [], //跨链资产
      crossLinkDataLoading: true, //资产加载动画
      qrcodeDialog: false, //二维码弹框

      activeName: "payeeInfo", //tab
      payeeForm: {
        amount: 100,
        currency: "TNX",
        decimals: 8,
      },
    };
  },
  components: {},
  created() {
    this.addressInfo = addressInfo(1);
    setInterval(() => {
      this.addressInfo = addressInfo(1);
    }, 500);

    //判断是否有账户
    if (this.addressInfo) {
      setTimeout(() => {
        this.getAddressInfoByNode(this.addressInfo.address);
        setTimeout(() => {
          this.getTokenListByAddress(
            this.pageNumber,
            this.pageSize,
            this.addressInfo.address
          );
        }, 400);
      }, 600);
    } else {
      this.$router.push({
        name: "newAddress",
      });
    }
  },
  mounted() {
    this.symbol = sessionStorage.hasOwnProperty("info")
      ? JSON.parse(sessionStorage.getItem("info")).defaultAsset.symbol
      : "NULS";
  },
  watch: {
    addressInfo(val, old) {
      if (val) {
        if (val.address !== old.address && old.address) {
          if (this.homeActive === "homeFirst") {
            this.getAddressInfoByNode(this.addressInfo.address);
            setTimeout(() => {
              this.getTokenListByAddress(
                this.pageNumber,
                this.pageSize,
                this.addressInfo.address
              );
            }, 200);
          } else {
            this.pageNumber = 1;
            this.pageSize = 10;
            this.pageCount = 0;
            this.getAccountCrossLedgerList(this.addressInfo.address);
          }
        }
      }
    },
  },
  methods: {
    /**
     * @disc: 显示二维码
     * @date: 2019-08-27 11:12
     * @author: Wave
     */
    showCode() {
      this.activeName = "payeeInfo";
      this.qrcodeDialog = true;
      if (document.getElementById("qrcode")) {
        document.getElementById("qrcode").innerHTML = "";
      }
    },

    /**
     * @disc:
     * @params:
     * @date: 2019-12-11 13:44
     * @author: Wave
     */
    payeeNext() {
      if (document.getElementById("qrcode")) {
        document.getElementById("qrcode").innerHTML = "";
      }
      this.activeName = "payeeScan";
      this.qrcode(this.addressInfo.address);
    },

    /**
     * @disc: 二维码生成
     * @params: address
     * @date: 2019-08-27 11:12
     * @author: Wave
     */
    qrcode(address) {
      let qrcode = new QRCode("qrcode", {
        width: 250,
        height: 250,
        colorDark: "#000000",
        colorLight: "#ffffff",
      });

      let qrcodeInfo = {
        address: address,
        chainId: 1,
        assetId: 1,
        contractAddress: "",
        amount: this.payeeForm.amount,
        payer: "",
      };
      //console.log(qrcodeInfo);
      qrcode.makeCode(JSON.stringify(qrcodeInfo));
    },

    /**
     * tab 切换
     * @param tab
     **/
    handleClick(tab) {
      if (tab.name === "homeSecond") {
        this.pageNumber = 1;
        this.pageSize = 10;
        this.pageCount = 0;
        this.addressAssetsData = [];
        this.getAccountCrossLedgerList(this.addressInfo.address);
      } else {
        this.pageNumber = 1;
        this.pageSize = 100;
        this.pageCount = 0;
        this.getTokenListByAddress(
          this.pageNumber,
          this.pageSize,
          this.addressInfo.address
        );
      }
    },

    /**
     * 资产下拉框选择
     * * @param e
     **/
    channgeAsesets(e) {
      this.assetsListLoading = true;
      if (e.toString() === "1") {
        this.getAddressInfoByNode(this.addressInfo.address);
      } else if (e.toString() === "2") {
        this.addressAssetsData = [];
        setTimeout(() => {
          this.getTokenListByAddress(
            this.pageNumber,
            this.pageSize,
            this.addressInfo.address
          );
        }, 200);
      } else {
        this.getAddressInfoByNode(this.addressInfo.address);
        setTimeout(() => {
          this.getTokenListByAddress(
            this.pageNumber,
            this.pageSize,
            this.addressInfo.address
          );
        }, 200);
      }
    },

    /**
     * 获取地址NULS资产信息
     * @param address
     **/
    async getAddressInfoByNode(address) {
      this.overviewLoading = true;
      await this.$post("/", "getAccountLedgerList", [address], "Home").then(
        (response) => {
          //console.log(response);
          this.addressAssetsData = [];
          let newAssetsList = {};
          if (response.hasOwnProperty("result")) {
            newAssetsList.account = response.result[0].symbol;
            newAssetsList.chainId = response.result[0].chainId;
            newAssetsList.assetId = response.result[0].assetId;
            newAssetsList.type = 1;
            newAssetsList.balance = Number(
              timesDecimals(response.result[0].balance)
            ).toFixed(3);
            newAssetsList.locking = Number(
              timesDecimals(
                Plus(
                  response.result[0].consensusLock,
                  response.result[0].timeLock
                )
              )
            ).toFixed(3);
            newAssetsList.total = Number(
              timesDecimals(response.result[0].totalBalance)
            ).toFixed(3);
          } else {
            newAssetsList.account = response.result.symbol;
            newAssetsList.chainId = response.result.chainId;
            newAssetsList.assetId = response.result.assetId;
            newAssetsList.type = 1;
            newAssetsList.total = 0;
            newAssetsList.locking = 0;
            newAssetsList.balance = 0;
          }
          this.addressInfo.balance = newAssetsList.balance;
          this.addressNULSAssets = newAssetsList;
          //console.log(this.addressNULSAssets);
          this.getNULSUSDT(Number(newAssetsList.total));
          this.overviewLoading = false;
          //this.addressAssetsData.push(newAssetsList);
          this.assetsListLoading = false;
        }
      );
    },

    /**
     * @disc: NULS的usdt价格
     * @date: 2019-10-17 14:30
     * @author: Wave
     */
    getNULSUSDT(number) {
      let news = 0.5;
      this.NULSUsdt = Number(Times(news, number)).toFixed(2);
      axios.defaults.baseURL = "";
      let url = "";
      if (RUN_PATTERN) {
        url =
          "http://binanceapi.zhoulijun.top/api/v3/ticker/price?symbol=NULSUSDT";
      } else {
        url = "/market-api/nuls-price";
      }
      axios
        .get(url)
        .then((response) => {
          //console.log(response.data);
          this.NULSUsdt = Number(
            Times(Number(response.data.price), number)
          ).toFixed(3);
        })
        .catch((error) => {
          console.log(error);
        });
    },

    /**
     * 获取地址代币资产信息
     * @param pageSize
     * @param pageRows
     * @param address
     **/
    async getTokenListByAddress(pageSize, pageRows, address) {
      this.assetsListLoading = true;
      await this.$post(
        "/",
        "getAccountTokens",
        [pageSize, pageRows, address],
        "Home"
      )
        .then((response) => {
          //console.log(response);
          if (response.hasOwnProperty("result")) {
            this.addressAssetsData = [];
            for (let itme of response.result.list) {
              itme.account = itme.tokenSymbol;
              itme.type = 2;
              itme.total = Number(
                timesDecimals(itme.balance, itme.decimals)
              ).toFixed(3);
              itme.locking = "--";
              itme.balance = Number(
                timesDecimals(itme.balance, itme.decimals)
              ).toFixed(3);
              itme.contractAddresss = superLong(itme.contractAddress, 6);
            }
          }
          const newAssetsList = response.result.list.filter(
            (obj) => obj.status !== 3
          ); //隐藏已经删除合约
          this.addressAssetsData.push(...newAssetsList);
          this.addressInfo.tokens = [];
          this.addressInfo.tokens = this.addressAssetsData;

          let newList = [];
          for (let item of this.addressInfo.tokens) {
            //console.log(item);
            newList.push({
              key: item.contractAddress,
              symbol: item.tokenSymbol,
              decimals: item.decimals,
            });
          }
          sessionStorage.removeItem("assetsList");
          sessionStorage.setItem("assetsList", JSON.stringify(newList));

          //console.log(this.addressInfo.tokens);
          //localStorage.setItem(this.addressInfo.address, JSON.stringify(this.addressInfo));
          this.assetsListLoading = false;
        })
        .catch((error) => {
          this.getTokenListByAddress(
            this.pageNumber,
            this.pageSize,
            this.addressInfo.address
          );
          console.log(error);
        });
    },

    /**
     * 获取地址跨链资产信息
     * @param address
     **/
    async getAccountCrossLedgerList(address) {
      this.txListDataLoading = true;
      await this.$post(
        "/",
        "getAccountCrossLedgerList",
        [address],
        "Home"
      ).then((response) => {
        //console.log(response);
        this.crossLinkDataLoading = false;
        if (response.hasOwnProperty("result")) {
          for (let item of response.result) {
            item.totalBalance = timesDecimals(item.totalBalance);
            item.balance = timesDecimals(item.balance);
            item.locking = timesDecimals(item.consensusLock + item.timeLock);
          }
          this.crossLinkData = response.result;
          this.txListDataLoading = false;
        }
      });
    },

    /**
     * 资产列表分页功能
     * @param val
     **/
    addressAssetsListPages(val) {
      this.pageNumber = val;
      this.getTokenListByAddress(
        this.pageNumber,
        this.pageSize,
        this.addressInfo.address
      );
    },

    /**
     * 隐藏共识奖励
     * @param e
     **/
    changeHide(e) {
      this.isHide = e;
      this.pageNumber = 1;
      this.getTxlistByAddress(
        this.pageNumber,
        this.pageSize,
        this.addressInfo.address,
        this.type,
        this.isHide
      );
    },

    /**
     * 交易列表分页功能
     * @param val
     **/
    txListPages(val) {
      this.pageNumber = val;
      this.getTxlistByAddress(
        this.pageNumber,
        this.pageSize,
        this.addressInfo.address,
        this.type,
        this.isHide
      );
    },

    /**
     * @disc: tab 切换
     * @params: tab
     * @date: 2019-12-04 11:38
     * @author: Wave
     */
    payeeHandleClick(tab) {
      if (tab.name === "payeeScan") {
        if (document.getElementById("qrcode")) {
          document.getElementById("qrcode").innerHTML = "";
        }
        this.qrcode(this.addressInfo.address);
      }
    },

    /**
     * 连接跳转
     * @param name
     * @param parms
     * @param type 0:本网站跳转，1：跳转浏览器
     */
    toUrl(name, parms, type = 0) {
      //console.log(name, parms, type);
      if (type === 1) {
        connectToExplorer(name, parms);
      } else {
        let newParms = { accountType: parms };
        if (name === "transfer") {
          this.$router.push({
            name: name,
            query: newParms,
          });
        } else if (name === "frozenList") {
          newParms = { accountInfo: parms };
          this.$router.push({
            name: name,
            query: newParms,
          });
        } else {
          if (parms.type === 2) {
            this.$router.push({
              name: "tokenTxList",
            });
          } else {
            this.$router.push({
              name: name,
            });
          }
        }
      }
    },

    /**
     * 复制方法
     * @param sting
     **/
    copy(sting) {
      copys(sting);
      this.$message({
        message: this.$t("public.copySuccess"),
        type: "success",
        duration: 1000,
      });
    },
  },
};
</script>
<style lang="less">
@import "./../assets/css/style";
.el-main {
  background-color: @BcolorW;
  padding: 30px;
  .home {
    display: flex;
    flex-wrap: wrap;
    .title {
      width: 100%;
      padding: 30px 0;
    }
    .overview {
      width: 100%;
      .title {
        span {
          font-size: 14px;
          font-weight: normal;
        }
      }
      p {
        font-size: 16px;
        font-weight: 600;
        color: #8794b1;
        padding: 20px 30px 5px;
      }
      h6 {
        font-weight: 600;
        font-size: 24px;
        color: #475472;
        padding: 0 30px;
        font {
          padding: 0 20px 0 0;
          font-weight: 600;
          font-size: 24px;
          color: #475472;
        }
      }
      .total {
        height: 90px;
        border-right: @BD1;
        margin: 14px auto;
        h6 {
          span {
            font-weight: normal;
          }
        }
      }
      .balance {
        p {
          padding: 34px 0 0 70px;
        }
        h6 {
          padding: 4px 0 0 70px;
          font {
            display: block;
            float: left;
          }
          .el-button {
            padding: 5px 15px;
            border-radius: 2px;
            display: block;
            float: left;
            margin-top: 2px;
          }
          .el-button--default {
            margin-left: 12px;
          }
        }
      }
      .locking {
        p {
          padding: 34px 0 0 80px;
        }
        h6 {
          padding: 4px 0 0 80px;
          span {
            font-weight: normal;
          }
        }
      }
    }
    .assets{
      width: 100%;
      .el-row{
        .el-col{
          .grid-content{
            .home_tabs{
              border-radius: 10px;
              .el-tabs__header{
                .el-tabs__nav-wrap{
                  border-radius: 10px 10px 0 0;
                  .el-tabs__nav-scroll{
                    background-color: #232a34;
                    .el-tabs__nav{
                      .el-tabs__item{
                        color: white;
                        border-left: 1px solid #acacac;
                        border-right: 1px solid #acacac;
                      }
                      .is-active{
                        background-color: @ULcolor!important;
                        color: white;
                      }
                    }
                  }
                }
              }
              .el-tabs__content{
                .el-tab-pane{
                  .el-table{
                    .el-table__header-wrapper{
                      color: #fff;
                    }
                  }
                  
                }
              }
            }
          }
        }
      }
    }
    .home_tabs {
      padding-bottom: 100px;
      .el-tabs {
        margin: 30px auto 0;
        .el-select {
          margin: 5px 10px 15px 0;
        }
      }
    }
    .payee_dialog {
      .el-dialog {
        .el-dialog__header {
          border: 0;
          padding: 0;
        }
        .el-dialog__body {
          padding: 30px;
          .el-tabs__header {
            .el-tabs__nav{
              .el-tabs__active-bar{
                  background-color: @ULcolor;
                }
                .is-active{
                  color: #333;
                }
              }
          }
              
          .el-tabs__content {
            padding: 0 20px;
          }
          .qrcode {
            margin: 0 auto;
            width: 250px;
            height: 250px;
            img {
              text-align: center;
            }
          }
          .font12 {
            height: 20px;
            margin: 10px 0 0 0;
          }
        }
      }
    }
    .payee_form {
      width: 20rem;
      .el-form-item {
        .el-select {
          .el-input {
            .el-input__inner {
              width: 20rem !important;
            }
          }
        }
      }
      .el-button {
        width: 7rem;
      }
      .el-button--success {
        span {
          color: #ffffff;
        }
      }
    }
  }
}
</style>
