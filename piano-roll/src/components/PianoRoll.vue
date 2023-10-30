<template>
  <div>
    <h1>Welcome to PianoRoll frontend coding challenge!</h1>
    <div id="buttonContainer">
      <button id="loadCSV" @click="handleClick">Load Piano Rolls!</button>
    </div>
    <div class="piano-roll-container">
      <div v-for="(item, index) in PianoRollsArray" :key="index" class="piano-roll-card" :class="{ active: index === activeItem }" @click="toggleActive(index)">
        <div class="piano-roll-svg" item>
          <div v-html="item.svgElement.outerHTML"></div>
        </div>
        <div class="description">This is a piano roll number {{ index }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import PianoRoll from '@/utils/PianoRollLogic';

export default {
  setup() {
    const pianoRollCount = 32;
    const activeItem = ref(null);
    const PianoRollsArray = ref([]);
    const csvURL = 'https://pianoroll.ai/random_notes';
    const data = ref(null);

    const toggleActive = (index) => {
      activeItem.value = index === activeItem.value ? null : index;
    };

    const loadPianoRollData = async () => {
      try {
        const response = await fetch(csvURL);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        data.value = await response.json();
      } catch (error) {
        console.error('Error loading data:', error);
      }
    };

    const handleClick = async () => {
      if (!data.value) await loadPianoRollData();
      if (!data.value) return;
      
      const buff = [];

      for (let it = 0; it < pianoRollCount; it++) {
        const start = it * 60;
        const end = start + 60;
        const partData = data.value.slice(start, end);

        const { svg } = preparePianoRollCard(it);
        const roll = new PianoRoll(svg, partData);
        buff.push(roll);
      }
      PianoRollsArray.value = buff;
      return PianoRollsArray;
    };

    const preparePianoRollCard = (rollId) => {
      const cardDiv = document.createElement('div');
      cardDiv.classList.add('piano-roll-card');

      const descriptionDiv = document.createElement('div');
      descriptionDiv.classList.add('description');
      descriptionDiv.textContent = `This is a piano roll number ${rollId}`;

      const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
      svg.classList.add('piano-roll-svg' + rollId);
      svg.setAttribute('width', '100%');
      svg.setAttribute('height', '160');

      cardDiv.appendChild(svg);
      cardDiv.appendChild(descriptionDiv);

      return { cardDiv, svg };
    };

    return {
      handleClick,
      pianoRollCount,
      toggleActive,
      PianoRollsArray,
      activeItem,
    };
  },
};
</script>
<style>
.piano-roll-container {
  display: grid;
  gap: 16px;
  grid-template-columns: repeat(1, 1fr); /* Default for screens smaller than 768px */

  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr); /* 768px and larger */
  }

  @media (min-width: 992px) {
    grid-template-columns: repeat(3, 1fr); /* 992px and larger */
  }

  @media (min-width: 1200px) {
    grid-template-columns: repeat(4, 1fr); /* 1200px and larger */
  }
}

  
h1 {
    margin-bottom: 20px;
    font-size: 42px;
}

button {
  padding: 15px 25px;
  font-size: 18px;
  color: #F0F0F0;
  background-color: #944038;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s;
  border-radius: 5px;
  border-bottom: 3px solid #381815;  /* A darker shade for 3D effect */
  position: relative;  /* Required for the top movement on hover */
  transition: all 1.1s ease;  /* Transition for all properties */
}



#buttonContainer {
  display: flex;
  justify-content: center;
}

button:hover {
  transform: scale(1.05);
}

.piano-roll-svg {
  border: 2px solid #381815;
}

.piano-roll-card {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  width: 100%;
  box-sizing: border-box;
}

.description {
  margin-top: 10px;
}
.active {
  transform: scale(1.2); 
  z-index: 1; 
  transition: transform 0.3s, z-index 0.3s;
}


div {
  transform: scale(1);
  transition: transform 0.2s; 
}

.container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

</style>