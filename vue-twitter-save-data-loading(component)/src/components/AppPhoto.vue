<template>
  <div class="adaptive-media" :style="style().mediaStyle">
    <div v-for="(medium, i) in media" :key="i" class="adaptive-media-photo-container" :style="style(medium.sizes, index).containStyle">
      <div v-if="isLight" class="adaptive-photo">
        <img :src="photoPath" alt="photos" :style="style(medium.sizes, index).photoStyle">
        <button class="load-image" @click="loadImageHandler">
          Load Image
        </button>
      </div>
      <div v-else="" class="adaptive-photo">
        <img :src="photoPath" alt="photos" :style="style(medium.sizes, index).photoStyle">
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
import { IMAGE_TYPE } from "../config";

export default {
  props: {
    media: {
      type: Array,
      default() {
        return [{ type: "", media_url: "", sizes: {} }]
      }
    }
  },
  data() {
    return {
      photoPath: ""
    }
  },
  computed: {
    mediaUrl() {
      return this.media[0].media_url
    },
    isLight() {
      return this.photoPath.includes(IMAGE_TYPE.LIGHT)
    },
  },
  watch: {
    mediaUrl: {
      handler(mediaUrl) {
        this.photoPath = mediaUrl
      },
      immediate: true
    }
  },
  methods: {
    loadImageHandler() {
      this.photoPath = this.media[0].media_url.replace(IMAGE_TYPE.LIGHT, IMAGE_TYPE.HEAVY)
    },
    style(sizes, index) {
      const mediaStyle = {
        'display': 'inline-block',
        'maxHeight': '508px',
        'maxWidth': '508px',
        'margin': '10px 0 0 0',
        'overflow': 'hidden',
        'border': '1px solid rgba(0,0,0,0.1)',
        'borderRadius': '5px',
        'verticalAlign': 'top',
        'textAlign': 'center',
        'width': '100%'
      };
      if (this.media.length === 2) mediaStyle.height = '253px';
      if (this.media.length === 3) mediaStyle.height = '337px';
      if (this.media.length === 4) mediaStyle.height = '380px';

      if (sizes !== null && typeof sizes !== "undefined") {
        // set initial sizes / styles
        const containStyle = {'width': '100%', 'position': 'relative', 'overflow': 'hidden'};
        const photoStyle = {'width': '100%', 'position': 'relative', 'verticalAlign': 'bottom'};
        let mediaHeight = sizes.large.h, mediaWidth = sizes.large.w;

        /*
        * format single photo
        */
        if (this.media.length === 1) {
          // 508 is the width of a tweet media wrapper
          // if image is wider than this, it's height will be reduced
          // proportionally, so we adjust our calculation
          if (mediaWidth > 508) {
            const ratio = (100 / mediaWidth) * 508;
            mediaHeight = mediaHeight * (ratio / 100);
          }

          // check if image is taller than maxHeight, if so
          // center it with a negative top value
          const maxHeight = parseInt(mediaStyle.maxHeight.replace('px', ''));
          if (mediaHeight > maxHeight) {
            photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
          }
        }

        /*
        * format two photos
        */
        if (this.media.length === 2) {
          const maxHeight = 253;
          photoStyle.width = 'auto';
          photoStyle.height = '100%';
          containStyle.display = 'inline-block';
          containStyle.height = '100%';
          // give first image 1px margin right and calc width to adjust
          if (index === 0) containStyle.marginRight = '1px'
          containStyle.width = 'calc(50% - .5px)';

          const ratio = (100 / mediaWidth) * (508 /2);
          mediaHeight = mediaHeight * (ratio / 100);
          if (mediaHeight > maxHeight) {
            photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
            photoStyle.width = '100%';
            photoStyle.height = 'auto';
          } else if (mediaWidth > (508 / 2)) {
            const ratio = (100 / sizes.large.h) * 253;
            mediaWidth = mediaWidth * (ratio / 100);
            photoStyle.left = `${((508 / 2) - mediaWidth) / 2}px`;
          }
        }

        /*
        * format three photos
        */
        if (this.media.length === 3)  {
          if (index === 0) {
            const maxHeight = 337;
            containStyle.width = `${100 * (2/3)}%`;
            containStyle.marginRight = '1px';
            containStyle.height = '337px';
            containStyle.float = 'left';
            const firstWrapWidth = 508 * (2 / 3);
            const ratio = (100 / mediaHeight) * 337;
            mediaWidth = mediaWidth * (ratio / 100);
            const newRatio = (100 / sizes.medium.w) * firstWrapWidth;
            mediaHeight = mediaHeight * (newRatio / 100);

            if (mediaHeight > maxHeight) {
              photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
            }

            if (mediaWidth > firstWrapWidth) {
              photoStyle.width = 'auto';
              photoStyle.height = '100%';
              photoStyle.left = `${((508 * (2/3)) - mediaWidth) / 2}px`;
            }
          }
          if (index !== 0) {
            mediaHeight = sizes.medium.h;
            mediaWidth = sizes.medium.w;
            const maxHeight = 337 / 2;
            const maxWidth = 508 * 1/3;
            const ratio = (100 / mediaWidth) * maxWidth;
            mediaHeight = mediaHeight * (ratio / 100);

            if (mediaHeight > maxHeight) {
              photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
            } else if (mediaWidth > maxWidth) {
              photoStyle.width = 'auto';
              photoStyle.height = '100%';
              const newRatio = (100 / sizes.medium.h) * maxWidth;
              mediaWidth = mediaWidth * (newRatio / 100);
              photoStyle.left = `${(maxWidth - mediaWidth) / 2}px`;
            }

            containStyle.float = 'left';
            containStyle.marginBottom = '1px';
            containStyle.height = `calc(100% / 2 - 1px/2)`;
            containStyle.width = `calc(100% / 3 - 1px)`;
          }
        }

        /*
        * format four photos
        */
        if (this.media.length === 4) {
          if (index === 0) {
            containStyle.width = '75%';
            containStyle.marginRight = '1px';
            containStyle.height = '380px';
            containStyle.float = 'left';
            const firstWrapWidth = 508 * 0.75;
            const maxHeight = 380;
            const ratio = (100 / mediaHeight) * 380;
            mediaWidth = mediaWidth * (ratio / 100);
            const newRatio = (100 / sizes.medium.w) * firstWrapWidth;
            mediaHeight = mediaHeight * (newRatio / 100);

            if (mediaHeight > maxHeight) {
              photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
            }

            if (mediaWidth > firstWrapWidth) {
              photoStyle.width = 'auto';
              photoStyle.height = '100%';
              photoStyle.left = `${((508 * 0.75) - mediaWidth) / 2}px`;
            }
          }
          if (index !== 0) {
            mediaHeight = sizes.medium.h;
            mediaWidth = sizes.medium.w;
            const maxHeight = 380 / 3;
            const maxWidth = 508 * 1/4;

            const ratio = (100 / mediaWidth) * maxWidth;
            mediaHeight = mediaHeight * (ratio / 100);

            if (mediaHeight > maxHeight) {
              photoStyle.top = `${(maxHeight - mediaHeight) / 2}px`;
            } else if (mediaWidth > maxWidth) {
              photoStyle.width = 'auto';
              photoStyle.height = '100%';
              const newRatio = (100 / sizes.medium.h) * maxWidth;
              mediaWidth = mediaWidth * (newRatio / 100);
              photoStyle.left = `${(maxWidth - mediaWidth) / 2}px`;
            }

            containStyle.height = 'calc(100% / 3 - 2px/3)';
            containStyle.marginBottom = '1px';
            containStyle.float = 'left';
            containStyle.width = 'calc(25% - 1px)';
          }
        }

        return {
          mediaStyle,
          containStyle,
          photoStyle
        }
      }

      return {
        mediaStyle,
        containStyle: {},
        photoStyle: {}
      }
    }
  }
}
</script>

<style>
/*
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the 'License');
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an 'AS IS' BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/* MEMO: tweak */
.adaptive-photo .load-image {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #555;
  color: white;
  font-size: .75em;
  padding: 8px 16px;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  text-align: center;
}

.adaptive-photo .load-image:hover {
  background-color: black;
}
</style>