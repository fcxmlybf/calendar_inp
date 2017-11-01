<template>
	<div class="calendar-input-container" :class="[isRed ? 'red-theme' : 'blue-theme']" :style="containerStyle">
		
		<label for="calendar-input">
			<input ref="mainInput" class="calendar-input" type="text" readonly="readonly" :class="{focus: isfocus}" @focus="focus" @blur="unfocus" v-model="selectValue">
		</label>
		
		<transition name="fade">
			<div id="calendar" v-show="changeShowCalendar">
				<div id="calendar-header">
					<span class="arrow" @click="subMonth">&lt;</span>
					<span id="date-box">{{trueSelectYear}}年{{trueSelectMonth}}月</span>
					<span class="arrow" @click="addMonth">&gt;</span>
				</div>
				<div class="week">
                    <span v-for="(item, index) in week" :class="{weekend: index === 0 || index === 6}">{{item}}</span>
				</div>
				<div class="days">
                    <span v-for="(item, index) in renderData"
                    :class="{weekend: index % 7 === 0 || index % 7 === 6,unselect: unselectArr.includes(index),select: index == firstDayInWeek + trueSelectDay - 1 }"
                    @click="changeSelectDay(index)">
                        {{item}}
                    </span>
				</div>
				<div class="btn-div">
					<a class="btn btn-today" @click="selectMonth=new Date().getMonth() + 1;selectDay=new Date().getDate()">今天</a>
					<a class="btn btn-yday" @click="selectYestoday">昨天</a>
					<a class="btn btn-quit" @click="showDateValue = false">取消</a>
				</div>
			</div>
		</transition>
	</div>
</template>

<script>
	export default {
		name: 'calendar-input',
		props: {
//			show: {   //是否显示组件
//				type: Boolean,
//				default: true
//			},
			isRed: {  //红蓝两种主题可选
				type: Boolean,
				default: false
			},
			limit: {  //日期可选范围
				type: Object,
				default() {
					return {
						minYear: 1900,
						minMonth: 1,
						minDay: 1,
						maxYear: 2030,
						maxMonth: 3,
						maxDay: 20
					}
				}
			},
			select:{
				type:Object,
				default(){
					return{
						selectYear: new Date().getFullYear(),
						selectMonth: new Date().getMonth() + 1,
						selectDay: new Date().getDate()
					}
				}
			},
			showCalendar: {  //是否显示日历
				type: Boolean,
				default: false
			},
			containerStyle: { //组件容器样式
				type: Object
			}
		},
		data () {
			return {
				isfocus: false,
				changeShowCalendar: this.showCalendar,
				week: ["日", "一", "二", "三", "四", "五", "六"],
				date: new Date(),
//				selectYear: new Date().getFullYear(),
//				selectMonth: new Date().getMonth() + 1,
//				selectDay: new Date().getDate()
				selectYear: this.select.selectYear,
				selectMonth:  this.select.selectMonth,
				selectDay:  this.select.selectDay,
				showDateValue:false,
			}
		},
		methods: {
			focus() {
				this.isfocus = true;
				this.changeShowCalendar = true;
				this.showDateValue = true;
			},
			unfocus(){
				this.isfocus = false;
				this.changeShowCalendar = false;
			},
			subMonth() {
				this.changeShowCalendar = true;
				if (this.selectMonth === 1) {
					this.selectMonth = 12;
					this.selectYear -= 1;
				} else {
					this.selectMonth -= 1
				}
				if (this.selectYear < this.limit.minYear) this.selectYear = this.limit.minYear;
				if (this.selectYear === this.limit.minYear) {
					if (this.selectMonth <= this.limit.minMonth) {
						this.selectMonth = this.limit.minMonth;
					}
				}
			},
			addMonth() {
				this.changeShowCalendar = true;
				if (this.selectMonth === 12) {
					this.selectMonth = 1;
					this.selectYear += 1;
				} else {
					this.selectMonth += 1
				}
				if (this.selectYear > this.limit.maxYear) this.selectYear = this.limit.maxYear;
				if (this.selectYear === this.limit.maxYear) {
					if (this.selectMonth >= this.limit.maxMonth) {
						this.selectMonth = this.limit.maxMonth;
					}
				}
			},
			changeSelectDay(index) {//index是0到42中的一个数字
				//当index序号隶属于上个月的日期或者下个月的日期的时候不能点
				if (this.unselectArr.includes(index)) return false;
				//得到点击的时候的日期数字
				this.selectDay = index - this.firstDayInWeek + 1;
				
				this.changeShowCalendar = false;
			},
			selectYestoday(){
				if(new Date().getDate() == 1){
					this.selectMonth=new Date().getMonth();
					this.selectDay = new Date(this.trueSelectYear, this.selectMonth, 0).getDate();
				}else{
					this.selectDay = new Date().getDate()-1;
				}
			}
		},
		computed: {
			trueSelectYear: function () {//过滤了一遍选择年份
				if (this.selectYear < this.limit.minYear) return this.limit.minYear;
				if (this.selectYear > this.limit.maxYear) return this.limit.maxYear;
				return this.selectYear;
			},
			trueSelectMonth: function () {//过滤了一遍选择月份
				if (this.selectYear === this.limit.minYear && this.selectMonth < this.limit.minMonth) return this.limit.minMonth;
				if (this.selectYear === this.limit.maxYear && this.selectMonth > this.limit.maxMonth) return this.limit.maxMonth;
				return this.selectMonth;
			},
			trueSelectDay: function () {//过滤了一遍选择日期
				if (this.selectYear === this.limit.minYear && this.selectMonth === this.limit.minMonth && this.selectDay < this.limit.minDay) return this.limit.minDay;
				if (this.selectYear === this.limit.maxYear && this.selectMonth === this.limit.maxMonth && this.selectDay > this.limit.maxDay) return this.limit.maxDay;
				if (this.selectDay > this.dayCount) return this.dayCount;
				return this.selectDay;
			},
			selectValue: function () {//得到选择的年月日值
				return `${this.trueSelectYear}-${this.trueSelectMonth}-${this.trueSelectDay}`;
			},
			firstDayInWeek: function () {//这个月第一天是星期几
				return new Date(this.trueSelectYear, this.trueSelectMonth - 1, 1).getDay();
			},
			dayCount: function () {//这个月有多少天
				return new Date(this.trueSelectYear, this.trueSelectMonth, 0).getDate();
			},
			lastMonthDay: function () {
				//这个月第一天是星期几
				let lastNum  = this.firstDayInWeek;
				let lastDays = [];
				if (lastNum === 0) return lastDays;
				
				//得到上个月的天数
				let lastDayNum = new Date(this.trueSelectYear, this.trueSelectMonth - 1, 0).getDate();
				for (let i = 0; i < lastNum; i++) {
					lastDays.unshift(lastDayNum);
					lastDayNum--;
				}
				return lastDays;
			},
			nextMonthDay: function () {
				//日历中这个月最后一天之后的数目
				let num      = 42 - this.firstDayInWeek - this.dayCount;
				let nextDays = [];
				if (num === 0) return nextDays;
				
				let i   = 1;
				for (; i <= num; i++) {
					nextDays.push(i);
				}
				return nextDays;
			},
			renderData: function () {//将这个月、上个月、下个月的42个日期全展示到calendar版面上
				let nowDays = [];
				let i = 1;
				for (; i <= this.dayCount; i++) {
					nowDays.push(i);
				}
				return [...this.lastMonthDay, ...nowDays, ...this.nextMonthDay];
			},
			unselectArr: function () {//每月只能点击这个月份的日期，上个月或下个月的日期都不能点，并放在这个arr中
				
				let index = 0;
				let arr   = [];
				
				if (this.trueSelectYear === this.limit.minYear && this.trueSelectMonth === this.limit.minMonth) {//最小的年份月份的处理
					for (; index < this.firstDayInWeek + this.limit.minDay - 1; index++) {
						arr.push(index);
					}
				} else {
					for (; index < this.firstDayInWeek; index++) {
						arr.push(index)
					}
				}
				
				if (this.trueSelectYear === this.limit.maxYear && this.trueSelectMonth === this.limit.maxMonth) {//最大的年份月份的处理
					index = this.firstDayInWeek + this.limit.maxDay;
					for (; index < 42; index++) {
						arr.push(index);
					}
				} else {
					index = this.firstDayInWeek + this.dayCount;
					for (; index < 42; index++) {
						arr.push(index);
					}
				}
				
				return arr;
			}
		},
		watch: {
			selectValue:function (newVal) {
				this.$emit('getValue', newVal);
			},
			showDateValue:function(event){
				if(!event){
					this.$refs.mainInput.value = '';
					this.$emit('getValue', '');
				}
			},
			select:function(event){
				this.selectYear=parseInt(event.selectYear);
				this.selectMonth=parseInt(event.selectMonth);
				this.selectDay=parseInt(event.selectDay);
			}
			
		},
//		mounted() {
//			this.$emit('getValue', this.selectValue);
//		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
	@import "./calender";
</style>
