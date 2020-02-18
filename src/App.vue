<template>
	<div id="app">
		<div
			class="sidebar"
			:class="{ 'hide': !isShowSidebar}"
		>
			<button
				class="toggle-sidebar-button"
				@click="isShowSidebar = !isShowSidebar"
			>
				<span
					class="arrow"
					:class="isShowSidebar? 'arrow-right': 'arrow-left'"
				></span>
			</button>
			<div class="today">
				<h2 class="today-week">星期二</h2>
				<div class="today-info">
					<div class="today-date">2020-02-06</div>
					<div class="today-description">身分證末碼為
						<span class="today-description-high-light">2,4,6,8,0</span>
						可購買
					</div>
				</div>
			</div>
			<div class="search">
				<div class="search-input-group">
					<label for="cityName">縣市</label>
					<select
						id="cityName"
						v-model="select.cityName"
						@change="select.areaName = ''"
					>
						<option
							value=""
							disabled
						>請選擇</option>
						<option
							v-for="city in cities"
							:key="city.CityName"
							:value="city.CityName"
						>
							{{city.CityName}}
						</option>
					</select>
				</div>
				<div class="search-input-group">
					<label for="areaName">地區</label>
					<select
						id="areaName"
						v-model="select.areaName"
					>
						<option
							value=""
							disabled
						>請選擇</option>
						<option
							v-for="area in cities.find((city) => city.CityName === select.cityName).AreaList"
							:key="area.AreaName"
							:value="area.AreaName"
						>
							{{area.AreaName}}
						</option>
					</select>
				</div>
			</div>
			<div class="pharmacies">
				<template v-if="filterPharmacies.length !== 0">
					<template v-for="item in filterPharmacies">
						<div
							class="pharmacy"
							:key="item.properties.id"
						>
							<div class="pharmacy-title">{{item.properties.name}}</div>
							<div class="pharmacy-address">{{item.properties.address}}</div>
							<div class="pharmacy-phone">{{item.properties.phone}}</div>
							<div class="pharmacy-mask-group">
								<div
									class="pharmacy-mask"
									:class="getMaskCountClass(item.properties.mask_adult)"
								>
									<div class="pharmacy-mask-title">成人口罩</div>
									<div class="pharmacy-mask-count">{{item.properties.mask_adult}}</div>
								</div>
								<div
									class="pharmacy-mask"
									:class="getMaskCountClass(item.properties.mask_adult)"
								>
									<div class="pharmacy-mask-title">兒童口罩</div>
									<div class="pharmacy-mask-count">{{item.properties.mask_child}}</div>
								</div>
							</div>
						</div>
					</template>
				</template>
				<div
					v-else
					class="no-data"
				>
					沒有資料 或 請選擇地區
				</div>
			</div>
		</div>
		<div id="map">map</div>
	</div>
</template>

<script>
import cities from "./assets/cities.json";

export default {
	name: "App",
	data: () => ({
		cities,
		isShowSidebar: true,
		pharmacies: [],
		select: {
			cityName: "臺北市",
			areaName: "大安區",
		},
	}),
	async mounted() {
		const response = await this.axios.get("https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json");
		this.pharmacies = response.data.features;
		console.log(this.pharmacies);
	},
	methods: {
		getMaskCountClass(count) {
			if (count >= 50) {
				return "pharmacy-mask-more";
			}

			if (count <= 0) {
				return "pharmacy-mask-none";
			}

			return "pharmacy-mask-less";
		}
	},
	computed: {
		filterPharmacies() {
			const { cityName, areaName } = this.select;
			return this.pharmacies.filter((pharmacy) => (
				pharmacy.properties.county === cityName && pharmacy.properties.town === areaName
			));
		}
	}
};
</script>

<style>
#map {
	width: 100%;
	height: 100vh;
}

.sidebar {
	z-index: 999;
	position: absolute;
	top: 0;
	left: 0;
	height: 100vh;
	max-height: 100vh;
	width: 400px;
	background-color: #73c0d8;
	color: white;
	box-shadow: 0px 3px 6px #00000029;
	display: flex;
	flex-direction: column;
	transition: all 0.5s ease-in;
}

.sidebar.hide {
	transform: translateX(-100%);
}

.toggle-sidebar-button {
	position: absolute;
	top: 124px;
	right: -20px;
	background-color: #73c0d8;
	color: white;
	z-index: 1000;
	width: 20px;
	height: 60px;
	border: none;
	box-shadow: 0px 3px 6px #00000029;
	border-radius: 0px 5px 5px 0px;
	cursor: pointer;
	padding: 0;
}

.toggle-sidebar-button:focus,
.toggle-sidebar-button:active {
	outline: none;
}

.arrow {
	display: inline-block;
	width: 8px;
	height: 8px;
	border-color: white;
}

.arrow-left {
	border-top: 2px solid;
	border-left: 2px solid;
	transform: rotate(135deg);
	margin-right: 4px;
}

.arrow-right {
	border-top: 2px solid;
	border-right: 2px solid;
	transform: rotate(-135deg);
	margin-left: 4px;
}

.toggle-sidebar-button::before {
	top: 0;
}

.today {
	padding: 16px 16px 8px;
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.today-week {
	font-size: 36px;
	font-weight: normal;
	margin: 0;
}

.today-info {
	display: flex;
	flex-direction: column;
	align-items: flex-end;
}

.today-date {
	margin-bottom: 8px;
}

.today-description {
	font-size: 14px;
}

.today-description-high-light {
	color: #fff126;
	font-size: 16px;
	font-weight: bold;
}

.search {
	padding: 4px 0 16px;
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.search-input-group {
	flex: 1;
	display: flex;
	align-items: center;
	margin: 0 16px;
}

.search-input-group label {
	font-size: 18px;
	margin-right: 8px;
}

.search-input-group select {
	flex: 1;
	display: block;
	width: 100%;
	height: 36px;
	padding: 8px 16px;
	font-size: 1rem;
	font-weight: 400;
	line-height: 1.5;
	color: #495057;
	background-color: #fff;
	background-clip: padding-box;
	border: 1px solid #ced4da;
	border-radius: 0.25rem;
	transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.pharmacies {
	overflow: auto;
}

.pharmacy {
	cursor: pointer;
	background-color: white;
	font-size: 14px;
	color: #666666;
	padding: 16px;
	border-bottom: 2px solid #d9d9d9;
}

.pharmacy:hover {
	opacity: 0.85;
}

.pharmacy-title {
	font-size: 18px;
	font-weight: bold;
	color: #333333;
	margin-bottom: 8px;
}

.pharmacy-address,
.pharmacy-phone {
	margin-bottom: 8px;
}

.pharmacy-mask-group {
	display: flex;
	justify-content: space-between;
}

.pharmacy-mask {
	width: 140px;
	height: 25px;
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 8px 16px;
	border-radius: 20px;
	color: white;
	font-weight: bold;
	font-size: 16px;
}

.pharmacy-mask-more {
	background-color: #73c0d8;
}

.pharmacy-mask-less {
	background-color: #ffa573;
}

.pharmacy-mask-none {
	background-color: #a5a5a5;
}

.no-data {
	margin-top: 100px;
	display: flex;
	justify-content: center;
	align-items: center;
}
</style>
