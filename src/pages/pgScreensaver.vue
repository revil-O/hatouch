<template>
	<div id="wrapper" style="height: 100vh;">
		<transition name="fadePage" v-for="(photo, index) in photos">
			<div class="page photo" v-on:click="loadDashboard" v-show="currentSlide == index" :style="{ 'background-image': 'url(' + photo + ')' }">
				<div class="clock">{{currentTime.hour}}:{{currentTime.min}}<small>{{currentTime.ampm}}</small></div>
				<div class="date">{{currentTime.dayOfWeek}}, {{currentTime.monthText}} {{currentTime.day}}</div>
			</div>
		</transition>
		<transition name="fadePage">
			<div class="page weather" v-on:click="loadDashboard" v-if="currentSlide == photos.length" :style="{ 'background-image': 'url(images/weather_backgrounds/' + weatherBackground + ')' }">
				<div class="row">
					<div class="col-sm-12"><h1>Current Conditions</h1></div>
				</div>
				<div class="row">
					<div class="col-sm-6">
						<div class="row">
							<div class="col-sm-12">
								<sensor entity_id="sensor.pws_weather" color-class="bg-yellow"></sensor>
							</div>
						</div>
						<div class="row">
							<div class="col-sm-6">
								<sensor entity_id="sensor.pws_temp_f" title="Temperature" icon="fa-sun-o" color-class="bg-red"></sensor>
							</div>
							<div class="col-sm-6">
								<sensor entity_id="sensor.pws_relative_humidity" title="Humidity" icon="fa-tint" color-class="bg-aqua"></sensor>
							</div>
						</div>
						<div class="row">
							<div class="col-sm-6">
								<sensor entity_id="sensor.pws_wind_mph" title="Wind Speed" icon="fa-flag-o" color-class="bg-green"></sensor>
							</div>
							<div class="col-sm-6">
								<sensor entity_id="sensor.pws_wind_dir" title="Wind Direction" icon="fa-compass" color-class="bg-green"></sensor>
							</div>
						</div>
					</div>
					<div class="col-sm-6">
						<div class="box box-primary">
							<div class="box-header with-border">
							<h3 class="box-title">Radar Loop</h3>
							</div>
							<div class="box-body" style="text-align: center;">
								<img src="https://radar.weather.gov/Conus/Loop/centgrtlakes_loop.gif" style="width: 100%;">
							</div>
						</div>
					</div>
				</div>
			</div>
		</transition>
	</div>
</template>

<script>
module.exports = {
	computed: {
		entities() {
			return this.$store.state.entities;
		},
		currentTime() {
			return this.$parent.timeVars;
		},
		weatherBackground() {
			//Map wunderground icon filename to background filenames
			var mapping = {
				'chanceflurries' : 'snow.jpg',
				'chancerain' : 'rain.jpg',
				'chancesleet' : 'rain.jpg',
				'chancesnow' : 'snow.jpg',
				'chancetstorms' : 'thunderstorm.jpg',
				'clear' : 'sunny.jpg',
				'cloudy' : 'cloudy.jpg',
				'flurries' : 'snow.jpg',
				'fog' : 'fog.jpg',
				'hazy' : 'fog.jpg',
				'mostlycloudy' : 'partlycloudy.jpg',
				'mostlysunny' : 'partlycloudy.jpg',
				'partlycloudy' : 'partlycloudy.jpg',
				'partlysunny' : 'partlycloudy.jpg',
				'sleet' : 'rain.jpg',
				'rain' : 'rain.jpg',
				'snow' : 'snow.jpg',
				'sunny' : 'sunny.jpg',
				'tstorms' : 'thunderstorm.jpg',
				'nt_chanceflurries' : 'nt_snow.jpg',
				'nt_chancerain' : 'nt_rain.jpg',
				'nt_chancesleet' : 'nt_rain.jpg',
				'nt_chancesnow' : 'nt_snow.jpg',
				'nt_chancetstorms' : 'nt_thunderstorm.jpg',
				'nt_clear' : 'nt_sunny.jpg',
				'nt_cloudy' : 'nt_cloudy.jpg',
				'nt_flurries' : 'nt_snow.jpg',
				'nt_fog' : 'nt_fog.jpg',
				'nt_hazy' : 'nt_fog.jpg',
				'nt_mostlycloudy' : 'nt_cloudy.jpg',
				'nt_mostlysunny' : 'nt_sunny.jpg',
				'nt_partlycloudy' : 'nt_cloudy.jpg',
				'nt_partlysunny' : 'nt_cloudy.jpg',
				'nt_sleet' : 'nt_rain.jpg',
				'nt_rain' : 'nt_rain.jpg',
				'nt_snow' : 'nt_snow.jpg',
				'nt_sunny' : 'nt_sunny.jpg',
				'nt_tstorms' : 'nt_thunderstorm.jpg',
			}
			//Make sure weather entity appears
			if (this.$store.state.entities['sensor.pws_weather'] !== undefined) {
				//Grab the wundergorund icon name
				var string = this.$store.state.entities['sensor.pws_weather']['attributes']['entity_picture'].split('/');
				var icon = string[string.length-1].split('.');
				if (mapping[icon[0]] !== undefined) {
					return mapping[icon[0]];
				} else {
					//Default background
					return 'partlycloudy.jpg';
				}
			} else {
				//Default background
				return 'partlycloudy.jpg';
			}
		}
	},
	data: function() {
		return {
			currentSlide : 11,
			maxSlides : 1,
			timerMultiply : 0,
			photos : []
		}
	},
	mounted: function() {
		var self = this;
		this.timer = setInterval(self.handleSlideTimer, 10000);
		
		//Get slides
		var xhr = new XMLHttpRequest();
		xhr.open('GET', 'get_pictures.php');
		xhr.onload = function(e) {
			if (JSON.parse(this.response) != null) {
				self.photos = JSON.parse(this.response);
				self.maxSlides = self.photos.length;
			}
		}
		xhr.send();
	},
	methods: {
		advanceSlide: function() {
			if (this.currentSlide < this.maxSlides) {
				this.currentSlide++;
			} else {
				this.currentSlide = 0;
			}
		},
		handleSlideTimer: function() {
			//Non photo slides stay up twice as long
			if (this.currentSlide > this.photos.length - 1) {
				if (this.timerMultiply < 1) {
					this.timerMultiply++;
				} else {
					this.timerMultiply = 0;
					this.advanceSlide();
				}
			} else {
				this.advanceSlide();
			}
		},
		loadDashboard: function() {
			this.$router.replace('/');
		}
	}
}
</script>
<style>
	.fadePage-enter-active, .fadePage-leave-active {
		transition: opacity .75s
	}
	.fadePage-enter, .fadePage-leave-to {
		opacity: 0
	}
	
	#wrapper {
		background-color: black;
	}
	
	.page {
		height: 100vh;
		width: 100vw;
		background-repeat: no-repeat;
		background-size: cover;
		background-position: center;
		padding: 48px;
		position: absolute;
		top: 0;
		left: 0;
	}
	
	.clock {
		position: absolute;
		color: white;
		font-size: 120px;
		font-weight: bold;
		right: 48px;
		bottom: 70px;
		text-shadow: 2px 2px 2px rgba(0, 0, 0, 1);
	}
	.clock small {
		font-size: 72px;
	}
	.date {
		position: absolute;
		bottom: 32px;
		right: 48px;
		text-shadow: 2px 2px 2px rgba(0, 0, 0, 1);
		color: white;
		font-size: 60px;
	}

	.weather h1 {
		color: white;
		margin-bottom: 48px;
		font-weight: bold;
		text-shadow: 2px 2px 2px rgba(0, 0, 0, 1);
	}
</style>