<template>
  <section id="calculator">
    <div class="history"></div>
    <div class="result" ref="resultRef">
      <div class="log">{{ operationLog }}</div>
      <input type="text" value="0" ref="resultOutput" v-model="resultValue" />
    </div>
    <div class="panel">
      <div class="panel-key" id="percent" @click="calculate('percent')">%</div>
      <div class="panel-key" id="root" @click="calculate('root')">√x</div>
      <div class="panel-key" id="power" @click="calculate('power')">^2</div>
      <div class="panel-key" id="fraction" @click="calculate('fraction')">1/x</div>

      <div class="panel-key" id="CE" @click="clearEntry()">CE</div>
      <div class="panel-key" id="C" @click="clear()">C</div>
      <div class="panel-key" id="delete" @click="deleteLastChar()">DEL</div>
      <div class="panel-key" id="div" @click="addOperand('div')">/</div>

      <div class="panel-key num" id="num_7" @click="dial('7')">7</div>
      <div class="panel-key num" id="num_8" @click="dial('8')">8</div>
      <div class="panel-key num" id="num_9" @click="dial('9')">9</div>
      <div class="panel-key" id="mul" @click="addOperand('mul')">*</div>

      <div class="panel-key num" id="num_4" @click="dial('4')">4</div>
      <div class="panel-key num" id="num_5" @click="dial('5')">5</div>
      <div class="panel-key num" id="num_6" @click="dial('6')">6</div>
      <div class="panel-key" id="minus" @click="addOperand('minus')">-</div>

      <div class="panel-key num" id="num_1" @click="dial('1')">1</div>
      <div class="panel-key num" id="num_2" @click="dial('2')">2</div>
      <div class="panel-key num" id="num_3" @click="dial('3')">3</div>
      <div class="panel-key" id="plus" @click="addOperand('plus')">+</div>

      <div class="panel-key" id="sign">+/-</div>
      <div class="panel-key num" id="num_0" @click="dial('0')">0</div>
      <div class="panel-key" id="dot" @click="dot()">.</div>
      <div class="panel-key" id="equals" @click="equals()">=</div>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";

@Component
export default class Calculator extends Vue {
  operationList: Array<string> = [];
  operationActive: boolean = false;

  resultValue: string = "0";
  lastValue: string = "";

  isVerified: boolean = true;

  operationLog: string = "";

  entryCleared: boolean = false;

  verifyInput(): boolean {
    const numbers: RegExp = /^[\d.]+$/g;

    return numbers.test(this.resultValue);
  }

  @Watch("resultValue")
  onResultChanged(currentValue: string, oldValue: string) {
    if (currentValue.length == 0) this.resultValue = "0";

    this.isVerified = this.verifyInput();

    if (!this.isVerified) this.resultValue = oldValue;
  }

  resetState() {
    this.operationList.length = 0;
  }

  dot() {
    if (this.resultValue.includes(".") || this.resultValue.length == 0) return;

    this.resultValue += ".";
  }

  clear() {
    this.resultValue = "0";

    this.resetState();
  }

  clearEntry() {
    console.log("CE");

    if (this.operationList.length == 0) return;

    if (isNaN(parseFloat(this.operationList[this.operationList.length - 1]))) {
      this.operationList.splice(-1);
      console.log(this.operationList);

      this.entryCleared = true;
    }
  }

  deleteLastChar() {
    if (this.resultValue.length == 0) return;

    this.resultValue = this.resultValue.substring(
      0,
      this.resultValue.length - 1
    );
  }

  resolveOperation() {
    if (!this.isVerified) return;

    if (this.operationList.length < 3) return;
    if (isNaN(parseFloat(this.operationList[0]))) return;
    if (isNaN(parseFloat(this.operationList[2]))) return;

    const number1: number = parseFloat(this.operationList[0]);
    const number2: number = parseFloat(this.operationList[2]);

    const operator: string = this.operationList[1];

    let result: number = 0;

    this.operationLog = number1.toString();

    switch (operator) {
      case "plus":
        result = number1 + number2;
        this.operationLog += "+";
        break;
      case "minus":
        result = number1 - number2;
        this.operationLog += "-";
        break;
      case "mul":
        result = number1 * number2;
        this.operationLog += "*";
        break;
      case "div":
        result = number1 / number2;
        this.operationLog += "/";
        break;

      default:
        break;
    }

    this.operationLog += number2;

    result = parseFloat(result.toFixed(5));

    this.operationList.splice(0, 3);
    this.operationList.unshift(result.toString());

    this.resultValue = result.toString();
  }

  addOperand(operand: string) {
    if (this.operationList.length == 1) {
      this.operationList.push(operand);
    } else {
      this.operationList.push(this.resultValue.toString(), operand);
    }

    this.operationActive = true;
    this.resolveOperation();
  }

  calculate(type: string) {
    if (!this.isVerified) return;

    this.operationList.push(this.resultValue.toString());
    this.resolveOperation();

    let result: number = parseFloat(this.resultValue);

    this.operationLog = this.resultValue;

    switch (type) {
      case "percent":
        result *= 0.01;
        this.operationLog = `(${this.operationLog})%`;
        break;
      case "root":
        result = Math.sqrt(result);
        this.operationLog = `sqrt(${this.operationLog})`;
        break;
      case "power":
        result *= result;
        this.operationLog = `(${this.operationLog})^2`;
        break;
      case "fraction":
        result = 1 / result;
        this.operationLog = `1/(${this.operationLog})`;
        break;

      default:
        break;
    }

    result = parseFloat(result.toFixed(5));

    this.resultValue = result.toString();
    this.operationList[0] = result.toString();
  }

  equals() {
    this.operationList.push(this.resultValue.toString());

    this.resolveOperation();
  }

  dial(num: number): void {
    if (this.resultValue.length > 18) return;

    if (this.resultValue == "0" || this.operationActive) {
      this.resultValue = num.toString();
      this.operationActive = false;
    } else this.resultValue += num.toString();
  }
}
</script>

<style lang="scss" scoped>
#calculator {
  min-height: 100vh;

  display: grid;
  grid-template-rows: minmax(100px, 2fr) minmax(50px, 0.5fr) minmax(200px, 3fr);
}

.history {
  background-color: #0062be;
}

.result {
  display: flex;
  justify-content: center;
  flex-direction: column;

  padding: 0.6rem;

  background-color: #0091d4;
  overflow: hidden;

  & > input {
    border: none;
    background: none;
    color: white;
    font-size: 2.3rem;
  }

  & > .log {
    color: rgb(218, 218, 218);
    font-size: 1rem;

    min-height: 1rem;
  }
}

.panel {
  background-color: #3d3d3d;

  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 5px;
  padding: 5px;

  &-key {
    display: flex;
    justify-content: center;
    align-items: center;

    font-size: 1.4rem;

    background-color: #5f5f5f;
    color: white;

    -moz-user-select: none;
    -webkit-user-select: none;

    cursor: pointer;

    transition: background-color 0.3s;

    &.num {
      background-color: #202020;
    }

    &:hover {
      background-color: #adadad;
    }
  }
}
</style>