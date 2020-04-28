<template>
	<view class="content">
		
		<!-- 地址 -->
		<view class="address">
			<text class="iconfont icondizhi1 blue"></text>
			<text>{{address}}></text>
		</view>
		
		<!-- 搜索 -->
		<search-box></search-box>
		
		<!-- 轮播图 -->
		<swiper class="high" indicator-dots="true" autoplay="true">
			<block v-for="item in bannerPic">
				<swiper-item>
					<image :src="item" mode="widthFix"></image>
				</swiper-item>
			</block>
		</swiper>
		
		<!-- 种类 -->
		<swiper indicator-dots="true" class="high2">
				<swiper-item>
					<block v-for="(item,index) in typePic[0].entries" v-if="index<10">
						<view class="img">
							<image class="img1" :src="item.image_hash"  mode="widthFix"></image>
							<text>{{item.name}}</text>
						</view>
					</block>
				</swiper-item>
				<swiper-item>
					<block v-for="(item,index) in typePic[0].entries" v-if="index>=10">
							<view class="img">
								<image class="img1" :src="item.image_hash"  mode="widthFix"></image>
								<text>{{item.name}}</text>
							</view>
						</block>
				</swiper-item>
		</swiper>
		
		<!-- 会员 -->
		<view class="vip">
			<image :src="typePic[1].entries[0].image_hash" mode="widthFix"></image>
			<view class="vipword">{{typePic[1].entries[0].name}}</view>
		</view>
		
		<!-- 商家 -->
		<view class="intro">附近推荐</view>
		<view v-for="item in restaurants">
			<view class="r_item">
				<view class="r_left">
					<image :src="item.restaurant.image_path" mode="widthFix"></image>
				</view>
				<view class="r_right">
					<view class="title">{{item.restaurant.name}}</view>
					<view class="grade">
						<text class="rating">★{{item.restaurant.rating}} </text>
						<text>{{item.restaurant.business_info.recent_order_num_display}}</text>
					</view>
					<view class="delivery">
						<text>起送 ￥ {{item.restaurant.float_minimum_order_amount}}  配送 ￥ {{item.restaurant.float_delivery_fee}} </text>
						<text class="time">{{item.restaurant.order_lead_time}}分钟 {{item.restaurant.distance}}</text>
					</view>
					<view class="comment" v-if="item.restaurant.promotion_info">
						<text>"{{item.restaurant.promotion_info}}"</text>
					</view>
					<view class="tag">
						<block v-for="item in item.restaurant.business_info.lemon_support_tags">
							<text class="discount">{{item.text}}</text>
						</block>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import search from '../../components/search/search.vue'
	export default {
		components:{
			searchBox:search
		},
		data() {
			return {
				address:'',
				bannerPic:[],
				typePic:[],
				restaurants:[],
				color:'ccc'
			}
		},
		onLoad() {
			this.location();	// 获取地址
			this.banner(); // 获取轮播图
			this.entrie(); // 获取种类图
			this.restaurant(); //获取商家信息
		},
		methods: {
			location()
			{
				uni.request({
					url:"http://192.168.137.1:3000/restapi/member/v2/users/14547420/location",
					method:"GET",
					sslVerify:true,
					success:(res)=>
					{
						this.address = res.data.addresses[0].address;
					},
					fail(err)
					{
						console.log('err:',err);
					}
				})
			},
			banner()
			{
				uni.request({
					url:"http://192.168.137.1:3000/restapi/shopping/v2/banners",
					method:"GET",
					sslVerify:true,
					success:(res)=>
					{
						for(let i in res.data)
						{
							this.bannerPic.push(this.urlFormatter(res.data[i].image_hash));
						}
					},
					fail(err)
					{
						console.log('err:',err);
					}
				})
			},
			entrie()
			{
				uni.request({
					url:"http://192.168.137.1:3000/restapi/shopping/v2/entries",
					method:"GET",
					sslVerify:true,
					success:(res)=>
					{
						for(let i in res.data)
						{
							for(let j in res.data[i].entries)
							{
								res.data[i].entries[j].image_hash = this.urlFormatter(res.data[i].entries[j].image_hash);
							}
						}
						this.typePic = res.data;
					},
					fail(err)
					{
						console.log('err:',err);
					}
				})
			},
			restaurant()
			{
				uni.request({
					url:"http://192.168.137.1:3000/restapi/shopping/v3/restaurants",
					method:"GET",
					sslVerify:true,
					success:(res)=>
					{
						for(let i in res.data.items)
						{
							res.data.items[i].restaurant.image_path = this.urlFormatter(res.data.items[i].restaurant.image_path);
							res.data.items[i].restaurant.business_info = JSON.parse(res.data.items[i].restaurant.business_info);
							if(res.data.items[i].restaurant.distance > 1000)
							{
								res.data.items[i].restaurant.distance = parseInt(res.data.items[i].restaurant.distance/1000*10)/10 + ' km';
							}
							else
							{
								res.data.items[i].restaurant.distance = res.data.items[i].restaurant.distance + ' m';
							}
						}
						this.restaurants = res.data.items;
						console.log('res:',this.restaurants);
					}
				})
			},
			 urlFormatter(oldValue) { //改变获取的图片格式
			   const first = "https://fuss10.elemecdn.com/";
			   const second = oldValue.slice(0, 1);
			   const third = oldValue.slice(1, 3);
			   const forth = oldValue.slice(3);
			   const type = forth.indexOf('png');
			   let fifth = '';
			   if(type == -1)
			   {
				   fifth =
				     ".jpeg?imageMogr/thumbnail/!90x90r/gravity/Center/crop/90x90/";
			   }
			   else
			   {
				   fifth =
				     ".png?imageMogr/thumbnail/!90x90r/gravity/Center/crop/90x90/";
			   }
			   return `${first}${second}/${third}/${forth}${fifth}`;
			 }
		}
	}
</script>

<style>
@import url("../../static/style/iconfont.css");
@import url("./index.css");
</style>

