<template>
  <div class="agencyBackstage">
    <div class="assetsDetailCls">
      <div class="commonNavBar positionFixed">
        <div class="backPassTitle"><p>代理后台</p></div>
        <div class="loginIcon arrowLeft flt "   @click="$router.go(-1);"><a href="javascript:void(0)"></a></div>
        <div class="loginIcon arrowRight xieyi"  @click="goContract"><a href="javascript:void(0)"></a></div>
      </div>
      <div id="assetsDeList" class="agencyAssetsDeList">
        <div>
          <div id="agencySubnav">
            <span @click="type = 0" :class="{on: type == 0}">佣金收益</span>
            <span @click="type = 1" :class="{on: type == 1}">下级列表</span>
          </div>
          <div class="agentsTotal clearfix" v-if="type == 0">
            <p>下级总数：<span>{{PeopleAlways}}</span>名</p>
            <p>今日新增下级：<span>{{newAdd}}</span>名</p>
          </div>
          <div class="agentsTotal clearfix" v-if="type == 1">
            <p style="border-right: none !important" size="large" @click="initDates(true)">选择年份
              <input type="text" :value="  value2 ? value2 : this.isYear" disabled>
            </p>
            <p @click="initDates()">选择月份
              <input type="text" :value="  value3 ? value3 : '全部'" disabled>
            </p>
          </div>
        </div>
        <div class="agencyBodyList">
          <div v-if="type == 0" class="agencyBodyListDiv">
            <ul class="masterUl agencyBackstageOne">
              <li>
                <p>结算时间段</p>
                <p>会员数</p>
                <p>盈亏额</p>
                <p>佣金</p>
              </li>
            </ul>
            <ul class="masterUl"
                v-infinite-scroll="loadMore"
                infinite-scroll-disabled="busy"
                infinite-scroll-distance="50"
                infinite-scroll-immediate-check="false">
              <li v-for="(master,index) in resultList">
                <p><span>{{master.name}}</span></p>
                <p><span>{{master.total_member_count}}</span></p>
                <p><span>￥{{master.total_member_profit}}</span></p>
                <p><span>￥{{master.commission}}</span></p>
              </li>
            </ul>
          </div>
          <div v-if="type == 1" class="agencyBodyListDiv agencyBodyJuniorList">
            <ul class="masterUl agencyBackstageOne">
              <li>
                <p style="width: 45%;">用户名</p>
                <p>盈亏额</p>
              </li>
            </ul>
            <ul class="masterUl"
                v-infinite-scroll="loadMore"
                infinite-scroll-disabled="busy"
                infinite-scroll-distance="50"
                infinite-scroll-immediate-check="false">
              <li class="listTit " v-for="(master,index) in resultLists">
                <p><span>{{master.username}}</span></p>
                <p><span>{{master.profit}}</span></p>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <BankChange :show=" years || month " :items="dateItems"></BankChange>
    </div>
  </div>
</template>

<script>
//	import '../../assets/scss/personal.scss';
	import BankChange from '../../components/bankChange.vue';

	export default {
		data() {
			return {
				result: [],
				dateItems: [],
				value2: 2018,
				value3: '全部',
				pickerValue: true,
				title: "代理后台",
				resultList: [],
				resultLists: [],
				showType: false,
				numIndex: 0,
				showTypeItems: [],
				delImg: require('../../assets/images/iconbtn2x.png'),
				busy: false,
				busys: false,
				params: {
					page: 1,
					year: '',
					month: ''
				},
				type: 0,
				moreMsg: '',
				betIndex: '',
				masterNum: '',
				masterNum01: '',
				masteramout: '',
				masterMarry: '',
				current_page: 0,
				last_page: 1,
				years: false,
				month: false,
				isYear: 2018,
				PeopleAlways: 0,
				newAdd: 0,
			}
		},
		mounted() {
      let that = this;
			that.init();
			that._Util.needClickFc('agencyAssetsDeList');
			let date = new Date;
			that.isYear = date.getFullYear();
			that.value2 =  that.isYear;
			that.$on('pickerCancel', () => {
				that.years = false;
				that.month = false;
				that.dateItems = [];
			});
			that.$on('pickerOk', (data) => {
				var numS = parseInt(data.province.name);
				if (numS <= 12) {
					that.value3 = numS
				} else {
					that.value2 = numS
				}
				that.params.year = this.value2;
				that.params.month = this.value3;
				that.params.page = 1;
				that.busy = false;
				if (that.params.month == '全部') {
					that.params.month = ''
				}
				that._Util.post(that, that._Api.POST_MASTER_ENDS, that.params, (data) => {
					that.resultLists = [];
					that.resultLists = data.data || [];
				})
				that.years = false;
				that.month = false;
				that.dateItems = [];
			})
		},
		methods: {
			initDates(type) {
				let that = this;
				if (type) {
					for (let i = that.isYear - 10; i < that.isYear + 2; i++) {
						that.dateItems.push({id: i, name: i});
					}
					that.years = true;
					that.month = false;
				} else {
					that.dateItems = [{id: 0, name: '全部'}];
					for (let i = 1; i < 13; i++) {
						that.dateItems.push({id: i, name: i});
					}
					that.years = false;
					that.month = true;
				}
			},
			init() {
				let that = this;
				if (!that.busy) {
					that.params.year = this.value2;
					that.params.month = this.value3;
					if (that.params.month == '全部') {
						that.params.month = ''
					}
					if (that.type == 0) {
						that.busy = true;
						that._Util.post(that, that._Api.POST_MASTER_END, that.params, (data) => {
							that.PeopleAlways = data.member_count;
							that.newAdd = data.member_daily_count;
							that.resultList = that.resultList.concat(data.data || []);
							that.masterNum = data.total || 0;
							if (data.data.length !== 0) {
								that.busy = false;
								that.params.page++;
							} else {
								that._Util.showAlert(that, {content: '已经没有更多数据了'});
								that.busy = true;
							}
							that.current_page = parseInt(data.current_page);
							that.last_page = parseInt(data.last_page);
						});
					}
					if (that.type == 1) {
						that.busy = true;
						that._Util.post(that, that._Api.POST_MASTER_ENDS, that.params, (data) => {
							that.resultLists = that.resultLists.concat(data.data || []);
							that.masterNum = data.total || 0;
							if (that.params.page <= data.last_page) {
								that.params.page++;
								that.busy = false;
							} else {
								that._Util.showAlert(that, {content: '已经没有更多数据了'});
								that.busy = true;
							}
							that.current_page = parseInt(data.current_page);
							that.last_page = parseInt(data.last_page);
						});
					}
				}
			},
			goContract() {
				let that = this;
				that.$router.push({name: 'agencyContract',query: {terminal: 1}});
			},
			changeType(v, index) {
				let that = this;
				that.numIndex = index;
				that.params.page = 1;
				that.params.type = v.type;
				that.resultList = [];
				that.showType = false;
				that.params.start_date = "";
				that.params.end_data = "";
				that.init();
			},
			loadMore() {
				let that = this;
				that.init();
			},
		},
		watch: {
			type() {
				let that = this;
				that.params.page = 1;
				that.busy = false;
				that.resultLists = [];
				that.resultList = [];
				this.init();
			}
		},
		components: {
			BankChange
		}

	}
</script>