<template>
  <section id="calculator">
    <div class="history"></div>
    <div class="result" ref="resultRef">0</div>
    <div class="panel">
      <div class="panel-key" id="percent" @click="calculate('percent')">%</div>
      <div class="panel-key" id="root" @click="calculate('root')">√x</div>
      <div class="panel-key" id="power" @click="calculate('power')">^2</div>
      <div class="panel-key" id="fraction" @click="calculate('fraction')">1/x</div>

      <div class="panel-key" id="CE">CE</div>
      <div class="panel-key" id="C" @click="clear()">C</div>
      <div class="panel-key" id="delete">DEL</div>
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
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Calculator extends Vue {
  operationList: Array<string> = [];
  operationActive: boolean = false;

  mounted() {}

  resetState() {
    this.operationList.length = 0;
    this.operationActive = true;
  }

  clear() {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;

    this.resetState();
    resultRef.innerText = "0";
  }

  resolveOperation() {
    if (this.operationList.length < 3) return;
    if (isNaN(parseFloat(this.operationList[0]))) return;
    if (isNaN(parseFloat(this.operationList[2]))) return;

    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;

    const number1: number = parseFloat(this.operationList[0]);
    const number2: number = parseFloat(this.operationList[2]);

    const operator: string = this.operationList[1];

    let result: number = 0;

    switch (operator) {
      case "plus":
        result = number1 + number2;
        break;
      case "minus":
        result = number1 - number2;
        break;
      case "mul":
        result = number1 * number2;
        break;
      case "div":
        result = number1 / number2;
        break;

      default:
        break;
    }

    result = parseFloat(result.toFixed(5));

    this.operationList.splice(0, 3);
    this.operationList.unshift(result.toString());

    resultRef.innerText = result.toString();

    console.log(this.operationList);
  }

  addOperand(operand: string) {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;

    this.operationList.push(resultRef.innerText);
    this.operationList.push(operand);

    this.operationActive = true;

    this.resolveOperation();
  }

  calculate(type: string) {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;

    this.operationList.push(resultRef.innerText);
    this.resolveOperation();

    let result: number = parseFloat(
      this.operationList.length == 0
        ? resultRef.innerText
        : this.operationList[0]
    );

    switch (type) {
      case "percent":
        result *= 0.01;
        break;
      case "root":
        result = Math.sqrt(result);
        break;
      case "power":
        result *= result;
        break;
      case "fraction":
        result = 1 / result;
        break;

      default:
        break;
    }
    result = parseFloat(result.toFixed(5));

    resultRef.innerText = result.toString();
    this.operationList[0] = result.toString();
    this.resetState();
  }

  equals() {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;
    this.operationList.push(resultRef.innerText);

    this.resolveOperation();
    this.resetState();
  }

  dot() {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;

    if (resultRef.innerText.includes(".") || resultRef.innerText.length == 0)
      return;

    resultRef.innerText += ".";
  }

  dial(num: number): void {
    const resultRef: HTMLElement = this.$refs.resultRef as HTMLElement;
    const resultValue: number = parseFloat(resultRef.innerText);

    if (resultRef.innerText == "0" || this.operationActive) {
      resultRef.innerText = num.toString();
      this.operationActive = false;
    } else resultRef.innerText += num.toString();
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
  align-items: center;

  font-size: 2.5rem;
  padding: 0.5rem;

  background-color: #0091d4;
  color: white;

  overflow: hidden;
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