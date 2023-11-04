<template>
  <div>
    <h1>Welcome to PianoRoll frontend coding challenge!</h1>
    <div id="buttonContainer">
      <button id="loadCSV" @click="handleClick">Load Piano Rolls!</button>
    </div>
    <div class="piano-roll-container" :class="{ pianoRollSelected: activeList.length > 0}">
      <div v-if="activeList.length == 0" class="gridContainerWrapper">
        <div v-for="(item, index) in PianoRollsArray" :key="index" class="piano-roll-card"
            @click="toggleActive(item, index)">
          <div v-html="item.svgElement.outerHTML"></div>
        </div>
      </div>

        <div v-if="activeIndex !== null" class="piano-roll-card  active">
          <div class="piano-roll-svg active">
          <div class="activeElement"
            @mousedown="startSelection(activeIndex, $event)"
            @mousemove="extendSelection(activeIndex, $event)"
            @mouseup="endSelection(activeIndex, $event)" 
            @mouseleave="endSelection(activeIndex, $event)">
            <div v-html="PianoRollsArray[activeIndex].svgElement.outerHTML"></div>     
            <div class="selection-line" v-if="isSelecting " :style="selectionLineStyle"></div>
            <div class="painted-area" v-if="isSelecting "  :style="paintedAreaStyle"></div>
          </div>
        </div>
        <div class="description">{{ activeIndex }}</div>  
        <div v-if="null !== activeIndex">there are {{notes.length}} selected</div>
        <div v-if="null !== activeIndex && notes.length > 0">startNoteIndex is {{startNoteIndex}} selected</div>
        <div v-if="null !== activeIndex && notes.length > 0">endNoteIndex is {{Math.round(endNoteIndex)}} selected</div>
        </div>
        <div class="ListOfInactives">
          <div v-for="(item ,index) in inactiveList" :key="index" class="piano-roll-card inactive"
              @click="toggleActive(item, index)"
              >                
              <div v-html="item.svgElement.outerHTML"></div>
                <div>{{ index }}</div>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import PianoRoll from '@/utils/PianoRollLogic';

export default {
  setup() {
    const pianoRollCount = 32;
    const activeIndex = ref(null);
    const inactiveList = ref([]);
    const activeList = ref([]);
    const activeElement = ref(null);
    const PianoRollsArray = ref([]);
    const csvURL = 'https://pianoroll.ai/random_notes';
    const data = ref(null);
    const notes = ref([]);

    const isSelecting = ref(false); 
    const startNoteIndex = ref(null); 
    const endNoteIndex = ref(null); 

    const toggleActive = (item ,index) => {
      if(activeElement.value === null) {
        activeIndex.value = index;
        activeElement.value = PianoRollsArray.value[index];
        activeList.value.push(activeElement);
        inactiveList.value = PianoRollsArray.value.filter((item) => item !== PianoRollsArray.value[index]);
      }
      if (activeIndex.value !== index) {
        
        activeList.value.splice(0,1,item);
        inactiveList.value.splice(activeIndex.value,1,item);
        console.log(inactiveList.value.splice(activeIndex.value,1,item))
        console.log( activeIndex.value, "  ",index);
        activeIndex.value = index;
      }
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
      if (!data.value) return; //error to throw
      
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
      svg.style.width='100%';

      cardDiv.appendChild(svg);
      cardDiv.appendChild(descriptionDiv);

      return { cardDiv, svg };
    };

    const startSelection = (index, event) => {
      if (activeIndex.value === index ) {
            isSelecting.value = true;
            startNoteIndex.value = event.layerX;
      }
    };
    const extendSelection = (index, event) => {
      if(isSelecting.value && activeIndex.value === index) {
        if(event.target.matches('.piano-roll-svg' + activeIndex.value)) {
          endNoteIndex.value = event.layerX;
          }
        else if(event.target.matches('.painted-area')) {
          endNoteIndex.value = Math.abs(event.clientX - event.currentTarget.getBoundingClientRect().left);
          }
        else if(event.target.matches('rect')) {
          endNoteIndex.value = event.clientX - event.currentTarget.getBoundingClientRect().left;
          }
        else if(event.target.matches('line')) {
          endNoteIndex.value = event.clientX - event.currentTarget.getBoundingClientRect().left;
      }
    }
    };

    const endSelection = (index) => {
      if (activeIndex.value === index && isSelecting.value) {
        isSelecting.value = false;

        const containerDiv = document.querySelector('.piano-roll-svg' + activeIndex.value);
        const noteElements = containerDiv.querySelectorAll('.note-rectangle');

        let SelectedNotes = [];
        noteElements.forEach((element) => {
            const elementRect = element.getBoundingClientRect();
            const elementX = Math.round(elementRect.left);
            if ((startNoteIndex.value <= elementX && endNoteIndex.value >= elementX) ||
                (startNoteIndex.value >= elementX && endNoteIndex.value <= elementX)) {
                SelectedNotes.push(element);
            }
        });
        console.log('Selected Notes:', SelectedNotes);
        
        return notes.value = SelectedNotes;
      }
    };
    const selectionLineStyle = computed(() => {
        if (isSelecting.value) {
        const left = Math.min(startNoteIndex.value, endNoteIndex.value) + 'px';
          return { left };
        } else {
          return { display: 'none' };
        }
        });

    const paintedAreaStyle = computed(() => {
        if (isSelecting.value) {
            const left = Math.min(startNoteIndex.value, endNoteIndex.value) + 'px';
            const width = Math.abs(startNoteIndex.value - endNoteIndex.value) + 'px';
            return { left, width };
        } else {
            return { display: 'none' };
        }
        });
    

    return {
      handleClick,
      pianoRollCount,
      toggleActive,
      PianoRollsArray,
      activeIndex,
      startSelection,
      endSelection,
      extendSelection,
      notes,
      paintedAreaStyle,
      selectionLineStyle,
      isSelecting,
      startNoteIndex,
      endNoteIndex,
      inactiveList,
      activeElement,
      activeList
    };
  },
};
</script>
<style>
.piano-roll-container .gridContainerWrapper {
  display: grid;
  gap: 3.5vw;
  margin: 1.5vh 5vw 0px 5vw;
  grid-template-columns: repeat(1, 1fr); 

  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media (min-width: 992px) {
    grid-template-columns: repeat(3, 1fr);
  }

  @media (min-width: 1200px) {
    grid-template-columns: repeat(4, 1fr); 
  }
}
.piano-roll-container.pianoRollSelected {
  display: flex;
  gap: 3.5vw;
  margin: 1.5vh 5vw 0px 5vw;
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
  border-bottom: 3px solid #381815;  
  position: relative;  
  transition: all 1.1s ease;  
}
svg {
  aspect-ratio: 4/3;
  width: 100%;
  @media (min-width: 768px) {
    aspect-ratio: 4/3;        
  }
  @media (min-width: 992px) {
    aspect-ratio: 16/9;
  }
  @media (min-width: 1800px) {
    aspect-ratio: 21/9;
  }
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
  margin-bottom: 8px;
  padding: 10px;
  width: 100%;
  box-sizing: border-box;
  border-radius: 10px; /* Rounded corners */
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); /* Shadow effect */
  background-color: #d0d9dc;
}
.piano-roll-card.active {
  position: relative;
  height: fit-content;
}
.description {
  margin-top: 10px;
}
.activeElement {
  position: relative;
}
.ListOfInactives{
  overflow-y: auto; 
  padding: 20px; 
  height: 70vh;
  scroll-behavior: smooth;
}
.selection-line {
    position: absolute;
    background-color: #00f; 
    width: 2px; 
    top: 0;
    bottom: 0;
  }

.painted-area {
    position: absolute;
    background-color: rgba(0, 0, 255, 0.2); 
    top: 0;
    bottom: 0;
  }

</style>