<template>

<h1> Welcome to PianoRoll frontend coding challenge!</h1>

  <div id="buttonContainer">
    <button id="loadCSV" @click="handleClick">Load Piano Rolls!</button>
  </div>

  <div id="pianoRollContainer" >
    <div v-for="it in pianoRollCount" :key="it" class="piano-roll-card">
        <div class="piano-roll-svg">
          <svg :id="'pianoRollSvg' + it" :ref="'pianoRollRef' + it"></svg>
        </div>
        <div class="description">This is a piano roll number {{ it }}</div>
      </div>
  </div>

</template>
<script>
import PianoRollDisplay from '@/utils/PianoRollDisplay.js'; // Using the "@" alias for the "src" directory


export default {
  setup() {
    const pianoRollCount = 30;

    const handleClick = async () => {
      const csvToSVG = new PianoRollDisplay('https://pianoroll.ai/random_notes');
      const generatedData = await csvToSVG.generateSVGs();
      if (generatedData){
        console.log(generatedData);
      }
    };

    return {
      handleClick,
      pianoRollCount
    };
  },
};

</script>
<style>
#pianoRollContainer {
  display: grid;
  grid-template-columns: repeat(5, 10fr);
  gap: 16px;
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

</style>