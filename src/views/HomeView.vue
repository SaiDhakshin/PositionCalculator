<template>
  <div>
    <h1>Position Size Calculator</h1>
    <div class="form">
    <label for="AccountBalance"><div class="form-label"> Account Balance: </div><input v-model="form.accountBalance" @input="calculateTotal"/></label>
    <!-- <label for="Instrument"><div class="form-label">Instrument: </div><input v-model="form.instrument"/></label> -->
    <label for="Leverage"><div class="form-label">Leverage: </div><input v-model="form.leverage" @input="calculateEffectiveCapital(); calculateTotal()"/></label>
    <label for="Riskpercentage"><div class="form-label">Risk Percentage (%): </div><input v-model="form.riskPercentage" @blur="calculateRiskAmount" @input="calculateTotal"/></label>
    <label for="Riskamount"><div class="form-label">Risk Amount: </div><input disabled v-model="form.riskAmount" /></label>
    <label for="Riskamount"><div class="form-label">Risk Amount Leveraged: </div><input disabled v-model="form.riskAmountLeveraged"/></label>
    <label for="Entry"><div class="form-label">Entry Price: </div><input v-model="form.entryPrice" @input="calculateTotal"/></label>
    <label for="Exit"><div class="form-label">Exit Price: </div><input v-model="form.exitPrice" @input="calculateTotal"/></label>
    <label for="NotionalValue"><div class="form-label">Notional Value: </div><input disabled v-model="form.notionalValue"/></label>
    <label for="NotionalValue"><div class="form-label">Notional Value Leveraged: </div><input disabled v-model="form.notionalValueLeveraged"/></label>
    <label for="PositionalSize"><div class="form-label">Position Size excluding Leverage: </div><input disabled v-model="form.positionSize"/></label>
    <label for="PositionalSize"><div class="form-label">Position Size without Leverage: </div><input disabled v-model="form.positionSizeNotLeveraged"/></label>
    <label for="PositionalSize"><div class="form-label">Position Size x{{ form.leverage }}: </div><input disabled v-model="form.positionSizeLeveraged"/></label>
    </div>
    <button @click="onSubmit">Calculate</button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const form: any = ref({
  accountBalance: 10000,
  effectiveCapital: '',
  instrument: '',
  leverage: '',
  riskPercentage: '',
  riskAmount: '',
  riskAmountLeveraged: '',
  entryPrice: '',
  exitPrice: '',
  notionalValue: '',
  notionalValueLeveraged: '',
  positionSize: '',
  positionSizeNotLeveraged: '',
  positionSizeLeveraged: ''
})

onMounted(async () => {});

// Function to calculate notional value based on entryPrice and positionSize
const calculateNotionalValue: any = () => {
    if (form.value.entryPrice && form.value.positionSize) {
      form.value.notionalValue = Math.abs(form.value.entryPrice * form.value.positionSize);
      form.value.notionalValueLeveraged = Math.abs(form.value.entryPrice * form.value.positionSize * form.value.leverage);
    } 
}

const calculateRiskAmount: any = () => {
    form.value.riskAmount = (form.value.accountBalance * form.value.riskPercentage) / 100;
    form.value.riskAmountLeveraged = (form.value.effectiveCapital * form.value.riskPercentage) / 100
}

const calculateEffectiveCapital: any = () => {
  if(form.value.leverage){
    form.value.effectiveCapital = form.value.accountBalance * form.value.leverage;
  }
}

const calculatePositionSize: any = () => {
  // Calculate risk per share based on position type
  let riskPerShare = 0;
  if (form.value.exitPrice > form.value.entryPrice) {
    riskPerShare = form.value.entryPrice - form.value.exitPrice; // Long position
  } else {
    riskPerShare = form.value.exitPrice - form.value.entryPrice; // Short position
  }

  // Calculate position size
  if (Math.abs(riskPerShare) > 0) {
    form.value.positionSize = Math.abs(form.value.riskAmount / (riskPerShare * form.value.leverage));
    form.value.positionSizeNotLeveraged = Math.abs(form.value.riskAmount / riskPerShare);
    form.value.positionSizeLeveraged = Math.abs((form.value.riskAmount / riskPerShare) * form.value.leverage);
    calculateNotionalValue();
  } else {
    form.value.positionSize = 0; // No valid position size if risk per share is not positive
    form.value.positionSizeLeveraged = 0;
    form.value.notionalValue = 0;
    form.value.notionalValueLeveraged = 0;
  }
}

const calculateTotal: any = () => {
  calculateRiskAmount();
  calculatePositionSize(); 
}

const onSubmit: any = () => {
  calculateTotal();
}
</script>

<style scoped>
.form{
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 30px;
}

.form-label{
  display: inline-block;
  width: 300px;
}
</style>
