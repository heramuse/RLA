<template>
  <v-container :style="{ maxWidth: '800px' }">
      <v-row>
          <v-col cols="6">
						<div id="webcam-container"></div>
          </v-col>
          <v-col cols="6">
						<v-card v-if="resultItem" class="fill-height ">
							<v-card-text>
								<div class="text-h4 black--text font-weight-bold">{{ resultItem.name }}</div>
								<v-layout title>
									<v-spacer></v-spacer>
									<v-flex shrink>메시지</v-flex>
									<v-flex shrink primary--text pl-3>{{ resultItem.message }}</v-flex>
								</v-layout>
								<v-divider class="my-6"></v-divider>
								<v-layout title>
									<v-flex xs4 grey--text></v-flex>
									<v-flex>{{ resultItem.m1 }}</v-flex>
								</v-layout>
								<v-layout title mt-2>
									<v-flex xs4 grey--text></v-flex>
									<v-flex>{{ resultItem.m2 }}</v-flex>
								</v-layout>
								<v-layout title mt-2>
									<v-flex xs4 grey--text></v-flex>
									<v-flex>{{ resultItem.m3 }}</v-flex>
								</v-layout>
								<v-layout title mt-2>
									<v-flex xs4 grey--text></v-flex>
									<v-flex>{{ resultItem.m4 }}</v-flex>
								</v-layout>
							</v-card-text>
						</v-card>
						<v-card v-show="resultItem === null" class="fill-height">
							<v-layout fill-height justify-center align-center row wrap>
								<v-flex shrink xs12 text-center>
									<v-img :style="{ margin: '0 auto' }" :width="100" :src="require('@/assets/smail.png')"></v-img>
									<div class="grey--text text--darken-1 mt-2">자신의 모습을 카메라에 보여주세요~!</div>
								</v-flex >
							</v-layout>
						</v-card>
          </v-col>
      </v-row>
  </v-container>
</template>

<script>
// eslint-disable-next-line no-unused-vars
import * as tf from '@tensorflow/tfjs';
import * as tmImage from '@teachablemachine/image';
import data from '../assets/data.json'
export default {
	data: () => ({
		URL: 'https://teachablemachine.withgoogle.com/models/FSAzmZEcF/',		
		model: null,
		webcam: null,
		labelContainer: null,
		maxPredictions: null,
		resultItem: null,
	}),
	created () {
		this.init()
	},
	methods: {
		async init () {
			const modelURL = this.URL + "model.json";
			const metadataURL = this.URL + "metadata.json";

			// load the model and metadata
			// Refer to tmImage.loadFromFiles() in the API to support files from a file picker
			// or files from your local hard drive
			// Note: the pose library adds "tmImage" object to your window (window.tmImage)
			this.model = await tmImage.load(modelURL, metadataURL);
			this.maxPredictions = this.model.getTotalClasses();

			// Convenience function to setup a webcam
			const flip = true; // whether to flip the webcam
			const width = 376
			this.webcam = new tmImage.Webcam(width, width, flip); // width, height, flip
			await this.webcam.setup(); // request access to the webcam
			await this.webcam.play();
			window.requestAnimationFrame(this.loop);

			// append elements to the DOM
			document.getElementById("webcam-container").appendChild(this.webcam.canvas);
		},
		async loop () {
			this.webcam.update(); // update the webcam frame
			await this.predict();
			window.requestAnimationFrame(this.loop);
		},
		async predict () {
			// predict can take in an image, video or canvas html element
			const prediction = await this.model.predict(this.webcam.canvas);
			let isPrediction = false
			
			for (let i = 0; i < this.maxPredictions; i++) {
				if (prediction[i].probability.toFixed(2) === '1.00') {
					const item = data[prediction[i].className] 
					this.resultItem = item			
					isPrediction = true					
				}
			}
			if (isPrediction === false) {
				this.resultItem = null
			}					
		}		
    }
}
</script>

<style>

</style>
