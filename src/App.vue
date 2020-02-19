<template>
	<div id="app">
		<div class="sidebar">
			<div class="today">
				<h2 class="today-week">{{today.getDay() | convertToChineseDay}}</h2>
				<div class="today-info">
					<div class="today-date">{{today | convertToDateString}}</div>
					<div class="today-description">身分證末碼為
						<span class="today-description-high-light">
							{{today.getDay() % 2 === 0 ? "2,4,6,8,0" : "1,3,5,7,9"}}
						</span>
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
						@change="select.areaName = ''; updateMap()"
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
						@change="updateMap"
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
					<template v-for="pharmacy in filterPharmacies">
						<div
							class="pharmacy"
							:key="pharmacy.properties.id"
							@click="focus(pharmacy)"
						>
							<div class="pharmacy-title">{{pharmacy.properties.name}}</div>
							<div class="pharmacy-address">{{pharmacy.properties.address}}</div>
							<div class="pharmacy-phone">{{pharmacy.properties.phone}}</div>
							<div class="pharmacy-mask-group">
								<div
									class="pharmacy-mask"
									:class="getMaskCountClass(pharmacy.properties.mask_adult)"
								>
									<div class="pharmacy-mask-title">成人口罩</div>
									<div class="pharmacy-mask-count">{{pharmacy.properties.mask_adult}}</div>
								</div>
								<div
									class="pharmacy-mask"
									:class="getMaskCountClass(pharmacy.properties.mask_child)"
								>
									<div class="pharmacy-mask-title">兒童口罩</div>
									<div class="pharmacy-mask-count">{{pharmacy.properties.mask_child}}</div>
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
import OpenStreetMap, { markerIcons } from "./classes/openStreetMap";

const getMaskCountClass = (count) => {
	if (count >= 50) {
		return "pharmacy-mask-more";
	}

	if (count <= 0) {
		return "pharmacy-mask-none";
	}

	return "pharmacy-mask-less";
};

const createPopUp = (pharmacy) => (`
<div class="pharmacy-title">${pharmacy.properties.name}</div>
  <div class="pharmacy-address">${pharmacy.properties.address}</div>
  <div class="pharmacy-phone">${pharmacy.properties.phone}</div>
  <div class="pharmacy-mask-group">
    <div
      class="pharmacy-mask ${getMaskCountClass(pharmacy.properties.mask_adult)}"
    >
      <div class="pharmacy-mask-title">成人口罩</div>
      <div class="pharmacy-mask-count">${pharmacy.properties.mask_adult}</div>
    </div>
    <div
      class="pharmacy-mask ${getMaskCountClass(pharmacy.properties.mask_child)}"
    >
      <div class="pharmacy-mask-title">兒童口罩</div>
      <div class="pharmacy-mask-count">${pharmacy.properties.mask_child}</div>
    </div>
  </div>
`);

export default {
	name: "App",
	data: () => ({
		cities,
		pharmacies: [],
		select: {
			cityName: "臺北市",
			areaName: "大安區",
		},
		today: new Date(),
	}),
	async mounted() {
		this.openStreetMap = new OpenStreetMap("map", 25.033671, 121.564427);

		const response = await this.axios.get("https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json");
		this.pharmacies = response.data.features;

		this.updateMap();
	},
	methods: {
		getMaskCountClass,
		setMarkers() {
			this.filterPharmacies.forEach((pharmacy) => {
				const [lng, lat] = pharmacy.geometry.coordinates;
				this.openStreetMap.addMarker(lat, lng, markerIcons.green, createPopUp(pharmacy));
			});
		},
		resetMarkers() {
			this.openStreetMap.removeAllMarkers();
		},
		moveTo(pharmacy) {
			const [lng, lat] = pharmacy.geometry.coordinates;
			this.openStreetMap.moveTo(lat, lng);
		},
		openPopUp(pharmacy) {
			const [lng, lat] = pharmacy.geometry.coordinates;
			this.openStreetMap.addMarker(lat, lng, markerIcons.green, createPopUp(pharmacy), true);
		},
		updateMap() {
			this.resetMarkers();
			this.setMarkers();

			if (this.filterPharmacies.length !== 0) {
				this.focus(this.filterPharmacies[0]);
			}
		},
		focus(pharmacy) {
			this.moveTo(pharmacy);
			this.openPopUp(pharmacy);
		}
	},
	computed: {
		filterPharmacies() {
			const { cityName, areaName } = this.select;
			return this.pharmacies.filter((pharmacy) => (
				pharmacy.properties.county === cityName && pharmacy.properties.town === areaName
			));
		}
	},
	filters: {
		convertToChineseDay(day) {
			const chineseDay = ["日", "一", "二", "三", "四", "五", "六"];
			return `星期${chineseDay[day]}`;
		},
		convertToDateString(date) {
			const pad = (n) => {
				return n < 10 ? `0${n}` : n;
			};
			return `${date.getFullYear()}-${pad(date.getMonth() + 1)}-${pad(date.getDate())}`;
		},

	}
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";

#map {
	margin-left: 400px;
	width: calc(100% - 400px);
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
	transition: all 0.3s;
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

.pharmacy-mask:not(:last-child) {
	margin-right: 16px;
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
