<template>
  <section id="calculator">
    <div class="history"></div>
    <div class="result" ref="resultRef">
      <input type="text" value="0" ref="resultOutput" v-model="resultValue" />
    </div>
    <div class="panel">
      <div class="panel-key" id="percent" @click="add('%')">%</div>
      <div class="panel-key" id="root" @click="add('r')">√x</div>
      <div class="panel-key" id="power" @click="add('p')">^2</div>
      <div class="panel-key" id="C" @click="clear()">C</div>

      <div class="panel-key" id="delete" @click="deleteLastChar()">DEL</div>
      <div class="panel-key" id="div" @click="add('(')">(</div>
      <div class="panel-key" id="div" @click="add(')')">)</div>
      <div class="panel-key" id="div" @click="add('/')">/</div>

      <div class="panel-key num" id="num_7" @click="add('7')">7</div>
      <div class="panel-key num" id="num_8" @click="add('8')">8</div>
      <div class="panel-key num" id="num_9" @click="add('9')">9</div>
      <div class="panel-key" id="mul" @click="add('*')">*</div>

      <div class="panel-key num" id="num_4" @click="add('4')">4</div>
      <div class="panel-key num" id="num_5" @click="add('5')">5</div>
      <div class="panel-key num" id="num_6" @click="add('6')">6</div>
      <div class="panel-key" id="minus" @click="add('-')">-</div>

      <div class="panel-key num" id="num_1" @click="add('1')">1</div>
      <div class="panel-key num" id="num_2" @click="add('2')">2</div>
      <div class="panel-key num" id="num_3" @click="add('3')">3</div>
      <div class="panel-key" id="plus" @click="add('+')">+</div>

      <div class="panel-key" id="CE">CE</div>
      <div class="panel-key num" id="num_0" @click="add('0')">0</div>
      <div class="panel-key" id="dot" @click="add('.')">.</div>
      <div class="panel-key" id="equals" @click="solve()">=</div>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";

@Component
export default class Calculator extends Vue {
  resultValue: string = "0";
  lastValue: string = "";

  exprArray: string[] = [];

  isVerified: boolean = true;

  operationLog: string = "";

  verifyInput(): boolean {
    const numbers: RegExp = /^[\d.]+$/g;

    return numbers.test(this.resultValue);
  }

  @Watch("resultValue")
  onResultChanged(currentValue: string, oldValue: string) {
    if (currentValue.length == 0) this.resultValue = "0";

    // this.isVerified = this.verifyInput();

    // if (!this.isVerified) this.resultValue = oldValue;
  }

  add(char: string) {
    if (this.resultValue == "0") this.resultValue = char;
    else this.resultValue += char;

    if (!/^[\d.]+$/.test(char)) {
      if (this.lastValue.length != 0) this.exprArray.push(this.lastValue);
      this.exprArray.push(char);
      this.lastValue = "";
    } else {
      this.lastValue += char;
    }

    console.log(this.lastValue);
  }

  solve() {
    if (this.resultValue == "0") return;

    if (this.lastValue.length != 0) this.exprArray.push(this.lastValue);

    if (!/[\d)]/.test(this.exprArray[this.exprArray.length - 1]))
      this.exprArray.pop();

    let stack: string[] = [];
    let output: string[] = [];
    let outputStr = "";

    for (let sign of this.exprArray) {
      if (/\d/.test(sign)) output.push(sign);
      else if (!/[()]/.test(sign)) {
        let foundHigher = false;
        for (let i = stack.length - 1; i >= 0; i--) {
          if (stack[i] == "(" && foundHigher) break;

          if (stack[i] == "*" || stack[i] == "/") foundHigher = true;
        }

        if (/[+-]/.test(sign) && foundHigher) {
          for (let i = stack.length - 1; i >= 0; i--) {
            if (stack[i] == "(") break;

            output.push(stack.pop() as string);
          }
        }

        stack.push(sign);
      }

      if (sign === "(") stack.push(sign);

      if (sign === ")") {
        for (let i = stack.length - 1; i >= 0; i--) {
          if (stack[i] == "(") {
            stack.pop();
            break;
          }

          output.push(stack.pop() as string);
        }
      }

      console.log(output);
    }

    for (let i = stack.length - 1; i >= 0; i--) output.push(stack[i]);

    stack.length = 0;

    output.forEach(el => (outputStr += el));

    console.log(output);

    this.solveRPN(output);
  }

  solveRPN(rpnArray: string[]) {
    let stack: string[] = [];

    for (let sign of rpnArray) {
      if (/\d/.test(sign)) {
        stack.push(sign);
      } else {
        const arg1: number = parseFloat(stack[stack.length - 1]);
        const arg2: number = parseFloat(stack[stack.length - 2]);

        stack.splice(-2);

        switch (sign) {
          case "+":
            stack.push((arg2 + arg1).toString());
            break;

          case "-":
            stack.push((arg2 - arg1).toString());
            break;

          case "*":
            stack.push((arg2 * arg1).toString());
            break;

          case "/":
            stack.push((arg2 / arg1).toString());
            break;

          default:
            break;
        }
      }
    }

    if (parseFloat(this.resultValue) != NaN) {
      this.resultValue = stack[0];

      this.lastValue = "";
      this.exprArray.length = 0;
      this.exprArray.push(stack[0]);

      console.log(this.exprArray);
    }
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