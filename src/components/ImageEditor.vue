<script setup>
import {computed, onMounted, ref} from 'vue';

let fileInput, previewImg, filterSlider;
let brightness = 100, saturation = 100, inversion = 0, grayscale = 0;
let rotate = 0, flipHorizontal = 1, flipVertical = 1;

onMounted(() => {
  fileInput = document.querySelector('.file-input');
  previewImg = document.querySelector('.preview-img img');
  filterSlider = document.querySelector('.slider input');
});

const applyFilters = () => {
  previewImg.style.transform = `rotate(${rotate}deg) scale(${flipHorizontal}, ${flipVertical})`;
  previewImg.style.filter = `brightness(${brightness}%) saturate(${saturation}%) invert(${inversion}%) grayscale(${grayscale}%)`;
};

const chooseImage = () => fileInput.click();
const loadImage = () => {
  let file = fileInput.files[0];
  if (!file) return;
  previewImg.src = URL.createObjectURL(file);
  previewImg.addEventListener('load', () => {
    resetFilters();
    document.querySelector('.editor-container').classList.remove('disable');
  });
};

const selectedFilter = ref('Brightness');
const sliderValue = ref(brightness);

const filter = (filter) => {
  selectedFilter.value = filter;

  if (selectedFilter.value == 'Brightness') {
    filterSlider.max = 200;
    sliderValue.value = brightness;
  } else if (selectedFilter.value == 'Saturation') {
    filterSlider.max = 200;
    sliderValue.value = saturation;
  } else if (selectedFilter.value == 'Inversion') {
    filterSlider.max = 100;
    sliderValue.value = inversion;
  } else {
    filterSlider.max = 100;
    sliderValue.value = grayscale;
  }
};

const updateSlider = () => {
  sliderValue.value = filterSlider.value;

  if (selectedFilter.value == 'Brightness') {
    brightness = filterSlider.value;
  } else if (selectedFilter.value == 'Saturation') {
    saturation = filterSlider.value;
  } else if (selectedFilter.value == 'Inversion') {
    inversion = filterSlider.value;
  } else {
    grayscale = filterSlider.value;
  }

  applyFilters();
};

const rotateImg = (deg) => {
  rotate += deg;
  applyFilters();
};

const flipImgHorizontal = () => {
  flipHorizontal = flipHorizontal === 1 ? -1 : 1;
  applyFilters();
};

const flipImgVertical = () => {
  flipVertical = flipVertical === 1 ? -1 : 1;
  applyFilters();
};

const resetFilters = () => {
  brightness = 100, saturation = 100, inversion = 0, grayscale = 0;
  rotate = 0, flipHorizontal = 1, flipVertical = 1;

  filter('Brightness');
  applyFilters();
};

const saveImage = () => {
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');
  canvas.width = previewImg.naturalWidth;
  canvas.height = previewImg.naturalHeight;

  ctx.filter = `brightness(${brightness}%) saturate(${saturation}%) invert(${inversion}%) grayscale(${grayscale}%)`;
  ctx.translate(canvas.width / 2, canvas.height / 2);
  if (rotate != 0) {
    ctx.rotate(rotate * Math.PI / 180);
  }
  ctx.scale(flipHorizontal, flipVertical);
  ctx.drawImage(previewImg, -canvas.width / 2, -canvas.height / 2, canvas.width, canvas.height);

  const link = document.createElement('a');
  link.download = 'image.jpg';
  link.href = canvas.toDataURL();
  link.click();
};
</script>

<template>
  <div class="editor-container disable">
    <h2>Easy Image Editor</h2>

    <div class="wrapper">
      <!-- Editor Panel -->
      <div class="editor-panel">
        <!-- Filters -->
        <div class="filters">
          <label class="title">Filters</label>

          <!-- Options -->
          <div class="options">
            <button @click="filter('Brightness')" :class="{'active': selectedFilter == 'Brightness'}">Brightness</button>
            <button @click="filter('Saturation')" :class="{'active': selectedFilter == 'Saturation'}">Saturation</button>
            <button @click="filter('Inversion')" :class="{'active': selectedFilter == 'Inversion'}">Inversion</button>
            <button @click="filter('Grayscale')" :class="{'active': selectedFilter == 'Grayscale'}">Grayscale</button>
          </div>

          <!-- Slider -->
          <div class="slider">
            <div class="slider-info">
              <p>{{ selectedFilter }}</p>
              <p>{{ sliderValue }}%</p>
            </div>
            <input @input="updateSlider()" type="range" :value="sliderValue" min="0" max="200">
          </div>
        </div>

        <!-- Rotate -->
        <div class="rotate">
          <label>Rotate & Flip</label>

          <!-- Options -->
          <div class="options">
            <button @click="rotateImg(-90)">
              <img src="../assets/restore.svg" class="w-5 h-5" alt="Rotate left">
            </button>
            <button @click="rotateImg(90)">
              <img src="../assets/reload.svg" class="w-5 h-5" alt="Rotate right">
            </button>
            <button @click="flipImgHorizontal()">
              <img src="../assets/flip-horizontal.svg" class="w-5 h-5" alt="Flip vertical">
            </button>
            <button @click="flipImgVertical()">
              <img src="../assets/flip-vertical.svg" class="w-5 h-5" alt="Flip horizontal">
            </button>
          </div>
        </div>
      </div>

      <!-- Preview Image -->
      <div class="preview-img">
        <img ref="previewImage" src="../assets/image-placeholder.svg" alt="Preview image">
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button @click="resetFilters()" class="reset-filters">Reset Filters</button>
      <div class="controls-right">
        <input @change="loadImage()" type="file" class="file-input" accept="image/*" hidden>
        <button @click="chooseImage()" class="choose-img">Choose Image</button>
        <button @click="saveImage()" class="save-img">Save Image</button>
      </div>
    </div>
  </div>
</template>
