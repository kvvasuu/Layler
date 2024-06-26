<template>
  <div class="title">Layler</div>
  <div class="description">
    Your tool for optimal pallet placement on truck trailers. Enter the
    dimensions and quantity of pallets, and our app will automatically plan the
    most efficient layout, saving you time and space.<br />
    The perfect solution for logistics and transportation!
  </div>
  <svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 512 512"
    fill="currentColor"
    class="help-button"
    @click="toggleHelpModal"
  >
    <path
      d="M256 512A256 256 0 1 0 256 0a256 256 0 1 0 0 512zM216 336h24V272H216c-13.3 0-24-10.7-24-24s10.7-24 24-24h48c13.3 0 24 10.7 24 24v88h8c13.3 0 24 10.7 24 24s-10.7 24-24 24H216c-13.3 0-24-10.7-24-24s10.7-24 24-24zm40-208a32 32 0 1 1 0 64 32 32 0 1 1 0-64z"
    />
  </svg>
  <div class="form-container">
    <div class="form-inputs">
      <div class="pallet-input-group">
        <input
          v-model="palletLength"
          id="palletLength"
          ref="palletLength"
          type="number"
          step="0.1"
          min="0.4"
          max="13.6"
          class="pallet-input"
          @blur="checkPalletLength"
        /><label class="pallet-input-label" for="palletLength">Length:</label>
        <div
          class="pallet-input-arrow-up"
          @mousedown="mouseDown($refs.palletLength, 'increase')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-up arrow"></div>
        </div>
        <div
          class="pallet-input-arrow-down"
          @mousedown="mouseDown($refs.palletLength, 'decrease')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-down arrow"></div>
        </div>
      </div>
      <div class="pallet-input-group">
        <input
          v-model="palletWidth"
          id="palletWidth"
          ref="palletWidth"
          type="number"
          step="0.1"
          min="0.4"
          max="2.4"
          class="pallet-input"
          @blur="checkPalletWidth"
        /><label class="pallet-input-label" for="palletWidth">Width:</label>
        <div
          class="pallet-input-arrow-up"
          @mousedown="mouseDown($refs.palletWidth, 'increase')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-up arrow"></div>
        </div>
        <div
          class="pallet-input-arrow-down"
          @mousedown="mouseDown($refs.palletWidth, 'decrease')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-down arrow"></div>
        </div>
      </div>
      <div class="pallet-input-group">
        <input
          v-model="numberOfPallets"
          id="numberOfPallets"
          ref="numberOfPallets"
          type="number"
          step="1"
          min="0"
          max="204"
          class="pallet-input"
        /><label class="pallet-input-label" for="numberOfPallets"
          >Number:</label
        >
        <div
          class="pallet-input-arrow-up"
          @mousedown="mouseDown($refs.numberOfPallets, 'increase')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-up arrow"></div>
        </div>
        <div
          class="pallet-input-arrow-down"
          @mousedown="mouseDown($refs.numberOfPallets, 'decrease')"
          @mouseout="mouseUp"
          @mouseup="mouseUp"
        >
          <div class="arrow-down arrow"></div>
        </div>
      </div>
    </div>
    <div class="form-button">
      <button
        class="button-sort"
        :class="{ 'button-sort-active': sort }"
        @click="sort = !sort"
      >
        Sort
      </button>
    </div>
  </div>
  <div class="trailer-container">
    <trailer
      :trailerLength="trailerLength"
      :trailerWidth="trailerWidth"
      :pallets="passPallets"
      :sort="sort"
      :rotate="mobile"
      @unloading="
        (bool, length) => (
          (unloading = bool),
          ((numberOfPallets = length), (currentPalletNumber = length))
        )
      "
      @update-all-pallets="(pallet) => updateInputs(pallet)"
    ></trailer>
    <input
      class="trailer-length trailer-size"
      v-model="trailerLength"
      id="trailerLength"
      type="number"
      min="1"
      max="20"
      step="0.01"
    />
    <input
      class="trailer-width trailer-size"
      v-model="trailerWidth"
      id="trailerWidth"
      type="number"
      min="1"
      max="4"
      step="0.01"
    />
    <Transition name="doorOpen">
      <div
        :class="{ 'door-left-mobile': mobile, 'door-left': !mobile }"
        :style="doorStyle"
        v-if="unloading"
      ></div>
    </Transition>
    <Transition name="doorOpen">
      <div
        :class="{ 'door-right-mobile': mobile, 'door-right': !mobile }"
        :style="doorStyle"
        v-if="unloading"
      ></div>
    </Transition>
  </div>
  <div class="state-buttons">
    <button class="button-sort state-button" @click="resetState">Reset</button>
    <button
      class="button-sort state-button"
      @click="(event) => loadState(event)"
    >
      Load
    </button>
    <button class="button-sort state-button" @click="saveState">Save</button>
  </div>
  <Transition name="bounce"
    ><div v-if="saveStateModal" class="saved-state-modal">
      Layout saved!
    </div></Transition
  >
  <Transition name="bounce"
    ><div v-if="loadStateModal" class="saved-state-modal">
      Layout loaded!
    </div></Transition
  >
  <Transition name="fade">
    <HelpModal v-if="helpModal" @toggle-modal="toggleHelpModal"></HelpModal>
  </Transition>
</template>

<script>
import HelpModal from "./components/HelpModal.vue";
import Trailer from "./components/Trailer.vue";

export default {
  components: {
    Trailer,
    HelpModal,
  },
  data() {
    return {
      trailerLength: 0,
      trailerWidth: 0,
      palletLength: 0,
      palletWidth: 0,
      numberOfPallets: 0,
      currentPalletNumber: 0,
      pallets: [],
      step: 0.1,
      sort: false,
      interval: null,
      isMouseDown: false,
      mobile: false,
      unloading: false,
      saveStateModal: false,
      loadStateModal: false,
      helpModal: false,
    };
  },
  methods: {
    arrangePallets() {
      if (this.numberOfPallets > 250) {
        this.numberOfPallets = 250;
      }
      let temp = this.pallets.length;

      if (temp < this.numberOfPallets) {
        while (temp < this.numberOfPallets) {
          this.pallets.push({
            length: this.palletLength,
            width: this.palletWidth,
            number: this.currentPalletNumber,
            name: "",
            color: "#dfa36c",
          });
          temp++;
          this.currentPalletNumber++;
        }
      } else if (this.numberOfPallets < temp) {
        while (temp > this.numberOfPallets) {
          this.pallets.pop();
          temp--;
          this.currentPalletNumber--;
        }
      }
    },
    increaseInput(ref) {
      let step = 0.1;
      if (ref.id == "numberOfPallets") step = 1;
      this[ref.id] = Math.round(Number(this[ref.id] + step) * 100) / 100;
    },
    decreaseInput(ref) {
      let step = 0.1;
      if (ref.id == "numberOfPallets") step = 1;
      if (this[ref.id] > 0)
        this[ref.id] = Math.round(Number(this[ref.id] - step) * 100) / 100;
    },
    async mouseDown(ref, type) {
      this.isMouseDown = true;
      switch (type) {
        case "increase":
          this.increaseInput(ref);
          this.interval = setInterval(() => {
            this.increaseInput(ref);
            this.checkPalletLength();
            this.checkPalletWidth();
          }, 300);
          setTimeout(() => {
            clearInterval(this.interval);
            if (this.isMouseDown) {
              this.interval = setInterval(() => {
                this.increaseInput(ref);
                this.checkPalletLength();
                this.checkPalletWidth();
              }, 100);
            }
          }, 1000);
          break;
        case "decrease":
          this.decreaseInput(ref);
          this.interval = setInterval(() => {
            this.decreaseInput(ref);
            this.checkPalletLength();
            this.checkPalletWidth();
          }, 300);
          setTimeout(() => {
            clearInterval(this.interval);
            if (this.isMouseDown) {
              this.interval = setInterval(() => {
                this.decreaseInput(ref);
                this.checkPalletLength();
                this.checkPalletWidth();
              }, 100);
            }
          }, 1000);
          break;
      }
    },
    async saveState() {
      localStorage.setItem("pallets", JSON.stringify(this.pallets));
      localStorage.setItem("trailerLength", JSON.stringify(this.trailerLength));
      localStorage.setItem("trailerWidth", JSON.stringify(this.trailerWidth));
      localStorage.setItem("palletLength", JSON.stringify(this.palletLength));
      localStorage.setItem("palletWidth", JSON.stringify(this.palletWidth));
      localStorage.setItem(
        "numberOfPallets",
        JSON.stringify(this.numberOfPallets)
      );
      localStorage.setItem("sort", JSON.stringify(this.sort));
      if (!this.saveStateModal) {
        this.saveStateModal = true;
      }
      setTimeout(() => {
        if (this.saveStateModal) {
          this.saveStateModal = false;
        }
      }, 1300);
    },
    async loadState(event) {
      if (localStorage.getItem("pallets") !== null) {
        this.trailerLength = await JSON.parse(
          localStorage.getItem("trailerLength")
        );
        this.trailerWidth = await JSON.parse(
          localStorage.getItem("trailerWidth")
        );
        this.palletLength = await JSON.parse(
          localStorage.getItem("palletLength")
        );
        this.palletWidth = await JSON.parse(
          localStorage.getItem("palletWidth")
        );
        this.numberOfPallets = await JSON.parse(
          localStorage.getItem("numberOfPallets")
        );
        this.sort = await JSON.parse(localStorage.getItem("sort"));
        this.pallets = await JSON.parse(localStorage.getItem("pallets"));
        if (event) {
          this.loadStateModal = true;
          setTimeout(() => {
            if (this.loadStateModal) {
              this.loadStateModal = false;
            }
          }, 1300);
        }
      } else {
        this.resetState();
      }
    },
    resetState() {
      (this.trailerLength = 13.6),
        (this.trailerWidth = 2.45),
        (this.palletLength = 1.2),
        (this.palletWidth = 0.8),
        (this.numberOfPallets = 7),
        (this.sort = false);
      this.pallets.map((el, index) => {
        el.length = this.palletLength;
        el.width = this.palletWidth;
        el.name = "";
        el.number = index;
        el.color = "#dfa36c";
      });
      this.arrangePallets();
      this.checkPalletLength();
      this.checkPalletWidth();
    },
    mouseUp() {
      this.isMouseDown = false;
      clearInterval(this.interval);
      this.checkPalletLength();
      this.checkPalletWidth();
    },
    isMobile() {
      if (
        /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
          navigator.userAgent
        ) ||
        window.innerWidth <= 1000 ||
        window.screenWidth <= 1000
      ) {
        this.mobile = true;
      } else {
        this.mobile = false;
      }
    },
    checkPalletLength() {
      if (this.pallets.length > 0) {
        if (this.palletLength > this.trailerLength) {
          this.palletLength = this.trailerLength;
        }
        if (this.palletLength < 0.4) {
          this.palletLength = 0.4;
        }
        if (
          this.pallets.every(
            (el) =>
              el.length === this.pallets[0].length &&
              this.pallets[0].length !== this.palletLength
          )
        ) {
          this.pallets.map((el) => {
            el.length = this.palletLength;
          });
        }
      }
    },
    checkPalletWidth() {
      if (this.pallets.length > 0) {
        if (this.palletWidth > this.trailerWidth) {
          this.palletWidth = this.trailerWidth;
        }
        if (this.palletWidth < 0.4) {
          this.palletWidth = 0.4;
        }
        if (
          this.pallets.every((el) => el.width === this.pallets[0].width) &&
          this.pallets[0].width !== this.palletWidth
        ) {
          this.pallets.map((el) => {
            el.width = this.palletWidth;
          });
        }
      }
    },
    updateInputs(pallet) {
      console.log(pallet);
      this.palletLength = pallet.length;
      this.palletWidth = pallet.width;
    },
    toggleHelpModal() {
      this.helpModal = !this.helpModal;
    },
  },
  watch: {
    numberOfPallets() {
      this.arrangePallets();
    },
    trailerLength() {
      if (this.trailerLength > 15) {
        this.trailerLength = 15;
      }
      if (this.trailerLength < 1) {
        this.trailerLength = 1;
      }
      this.arrangePallets();
    },
    trailerWidth() {
      if (this.trailerWidth > 4) {
        this.trailerWidth = 4;
      }
      if (this.trailerWidth < 1) {
        this.trailerWidth = 1;
      }
      this.arrangePallets();
    },
  },
  computed: {
    passPallets() {
      return this.pallets;
    },
    doorStyle() {
      if (!this.mobile) {
        return {
          width: "0.1rem",
          height: this.trailerWidth / 2 + 0.2 + "rem",
        };
      } else {
        return {
          height: "0.1rem",
          width: this.trailerWidth / 2 + 0.2 + "rem",
        };
      }
    },
  },
  created() {
    this.isMobile();
    window.addEventListener("resize", this.isMobile);
    this.loadState();
  },
  destroyed() {
    window.removeEventListener("resize", this.isMobile);
  },
};
</script>

<style scoped>
.title {
  font-size: 0.7rem;
  font-weight: bold;
  margin: 0 0 0.1rem 0;
}

.description {
  max-width: 10rem;
  text-align: center;
  margin: 0 0.6rem 0.6rem 0.6rem;
}

.help-button {
  position: absolute;
  top: 0.1rem;
  right: 0.5rem;
  height: 0.5rem;
  cursor: pointer;
  border-radius: 1rem;
  margin: 0.1rem;
  transition: all 0.3s ease;
}

.help-button:active {
  color: rgb(223, 163, 108);
  box-shadow: 0 0 4px rgb(160, 113, 70);
}

.form-container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.form-inputs {
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-button {
  display: flex;
  align-items: center;
  justify-content: center;
}

.pallet-input-group {
  position: relative;
  margin: 0.5rem;
}

.pallet-input {
  outline: none !important;
  background-color: transparent;
  border: 1px solid #bdbdbd;
  border-radius: 2rem;
  padding: 0.14rem 0.1rem 0.08rem 0.1rem;
  text-align: center;
  color: #bdbdbd;
  width: 2rem;
  height: 0.6rem;
  font-size: 0.35rem;
  font-weight: bold;
  transition: all 0.3s ease;
}

.pallet-input:focus {
  color: rgb(223, 163, 108);
  outline: none !important;
  border: 1.5px solid rgb(241, 165, 94);
  filter: drop-shadow(0 0 1px rgb(160, 113, 70));
}

.pallet-input-label {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #bdbdbd;
  font-weight: bold;
  padding: 0 5px;
  border: 1.5px solid #bdbdbd;
  border-radius: 20px;
  top: -20%;
  left: 15%;
  background-color: transparent;
  cursor: default;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.pallet-input:focus ~ .pallet-input-label {
  color: rgb(241, 165, 94);
  border: 1.5px solid rgb(241, 165, 94);
}

.pallet-input-arrow-up {
  position: absolute;
  width: 0.4rem;
  height: 0.34rem;
  top: 0;
  right: 0.1rem;
  background-color: transparent;
  border: none;
  cursor: pointer;
}
.pallet-input-arrow-down {
  position: absolute;
  width: 0.4rem;
  height: 0.34rem;
  bottom: 0;
  right: 0.1rem;
  background-color: transparent;
  cursor: pointer;
}

.arrow-up {
  position: absolute;
  width: 0.18rem;
  height: 0.18rem;
  top: 60%;
  border: none;
  border-top: 0.06rem solid #bdbdbd;
  border-left: 0.06rem solid #bdbdbd;
  rotate: 45deg;
  transition: all 0.3s ease;
}

.arrow-down {
  position: absolute;
  width: 0.18rem;
  height: 0.18rem;
  bottom: 60%;
  border: none;
  border-top: 0.06rem solid #bdbdbd;
  border-left: 0.06rem solid #bdbdbd;
  rotate: -135deg;
  transition: all 0.3s ease;
}

.arrow:hover {
  border-top: 0.06rem solid rgb(241, 165, 94);
  border-left: 0.06rem solid rgb(241, 165, 94);
  filter: drop-shadow(0 0 1px rgb(160, 113, 70));
}

.arrow:active {
  transform: scale(0.9);
}

.button-sort {
  background-color: transparent;
  cursor: pointer;
  border: 1px solid #bdbdbd;
  border-radius: 2rem;
  padding: 0.1rem;
  text-align: center;
  color: #bdbdbd;
  width: 1.4rem;
  font-size: 0.3rem;
  font-weight: bold;
  transition: color 0.3s ease, border 0.3s ease, filter 0.3s ease;
}

.button-sort:active {
  color: rgb(241, 165, 94);
  border: 1.5px solid rgb(241, 165, 94);
  transform: translate(0, 1px);
}

.button-sort-active {
  color: rgb(241, 165, 94);
  border: 1.5px solid rgb(241, 165, 94);
}

.trailer-container {
  position: relative;
  margin: 0.5rem;
}

.door-left {
  right: -0.01rem;
  top: 0;
  position: absolute;
  background-color: rgb(37, 31, 25);
  box-sizing: border-box;
  border-radius: 0.03rem;
  transform-origin: 50% 2%;
  rotate: -135deg;
}

.door-left-mobile {
  left: -0.01rem;
  bottom: 0;
  position: absolute;
  background-color: rgb(37, 31, 25);
  box-sizing: border-box;
  border-radius: 0.03rem;
  transform-origin: 2% 50%;
  rotate: 135deg;
}

.door-right {
  right: -0.01rem;
  bottom: 0;
  position: absolute;
  background-color: rgb(37, 31, 25);
  box-sizing: border-box;
  border-radius: 0.03rem;
  transform-origin: 50% 98%;
  rotate: 135deg;
}

.door-right-mobile {
  right: 0.01rem;
  bottom: 0;
  position: absolute;
  background-color: rgb(37, 31, 25);
  box-sizing: border-box;
  border-radius: 0.03rem;
  transform-origin: 98% 50%;
  rotate: -135deg;
}

.trailer-length {
  position: absolute;
  bottom: -32px;
  left: 50%;
  transform: translate(-50%, 0);
  border: none;
  width: 50px;
}

.trailer-width {
  position: absolute;
  top: 50%;
  right: -60px;
  transform: translate(0, -50%);
  width: 40px;
  border: none;
}

.trailer-size {
  background-color: transparent;
  text-align: center;
  font-size: 20px;
  color: #bdbdbd;
  transition: color 0.3s ease;
}

.trailer-size:focus {
  color: rgb(241, 165, 94);
  filter: drop-shadow(0 0 1px rgb(160, 113, 70));
  outline: none !important;
}

.trailer-size::-webkit-outer-spin-button,
.trailer-size::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.state-buttons {
  margin-top: 0.6rem;
  margin-left: auto;
  margin-right: 0.6rem;
}

.state-button {
  margin: 0 0 0 0.2rem;
}

.saved-state-modal {
  text-align: center;
  text-justify: center;
  padding: 0.3rem;
  position: fixed;
  font-size: 0.4rem;
  background: rgb(52, 52, 53);
  background: radial-gradient(
    circle,
    rgba(52, 52, 53, 1) 0%,
    rgb(44, 44, 44) 100%
  );
  border: 1px solid #929292;
  border-radius: 2rem;
  top: 1rem;
  z-index: 12;
}

@media screen and (max-width: 1000px) {
  .state-buttons {
    margin: auto;
    margin-top: 0.6rem;
  }
  .state-button {
    margin: 0.1rem;
  }
  .help-button {
    top: -0.1rem;
    right: -0.1rem;
  }
}

@media screen and (max-width: 640px) {
  .form-inputs {
    flex-direction: column;
  }
  .pallet-input-arrow-up,
  .pallet-input-arrow-down {
    display: none;
  }
  .state-buttons {
    margin: auto;
    margin-top: 0.6rem;
  }
  .pallet-input-group {
    margin: 0.3rem;
  }
}

@media (hover: hover) {
  .button-sort:hover {
    color: #f1a55e;
    border: 1.5px solid rgb(241, 165, 94);
    filter: drop-shadow(0 0 1px rgb(160, 113, 70));
  }
  .help-button:hover {
    color: rgb(223, 163, 108);
    box-shadow: 0 0 4px rgb(160, 113, 70);
  }
}

.doorOpen-enter-active,
.doorOpen-leave-active {
  transition: all 0.5s ease;
}

.doorOpen-enter-from,
.doorOpen-leave-to {
  rotate: 0deg;
  opacity: 0;
}
</style>
