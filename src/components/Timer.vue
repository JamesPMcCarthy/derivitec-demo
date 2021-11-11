<template>
  <div class="timer">

    <v-time-picker
      format="24hr"
      v-model="selectedTime"
      use-seconds
      @input="BackupTime()"
      ref="timepicker"
    ></v-time-picker>

    <div >

      <div class="control-btns">
        <v-btn
          elevation="2"      
          :color="pollingEvent === -1 ? 'success' : 'error'"
          @click="triggerTimer()"
          :disabled="selectedTime === '00:00:00'">{{(pollingEvent === -1 ? "Start" : "Stop") + " Timer"}}</v-btn>
      </div>

      <div>
        <v-btn
          class="control-btns"
          elevation="2"      
          color="warning"
          @click="RevertBackupTime()"
          :disabled="selectedTimeCpy === '00:00:00' || selectedTimeCpy === selectedTime ">reset</v-btn>
        <v-btn
          class="control-btns"
          elevation="2"      
          color="error"
          @click="ClearSelection()"
          :disabled="selectedTime === '00:00:00'">Clear</v-btn>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Timer extends Vue {

  $refs!: {
    timepicker: HTMLFormElement
  }

  private selectedTime = "00:00:30";
  private selectedTimeCpy = "00:00:30";
  private pollingEvent?: number = -1;

  @Prop() private msg!: string;

  public triggerTimer(): void {    

    try{
      if(this.selectedTime === "00:00:00")
      {
        throw "No Time Set";
      }

      if(this.pollingEvent == -1){
        this.$refs.timepicker.selecting = 3
        this.pollingEvent = setInterval(()=>{this.timerEvent()}, 1000);
      }
      else{
        clearInterval(this.pollingEvent);
        this.pollingEvent = -1;
      }
    }
    catch(e){
      console.log(e)
    }    
  }
  
  public BackupTime() :void{

    this.selectedTimeCpy = JSON.parse(JSON.stringify(this.selectedTime));
  }

  public RevertBackupTime() :void{

    this.selectedTime = JSON.parse(JSON.stringify(this.selectedTimeCpy));
    this.CancelEvent();
  }

  public ClearSelection() :void{

    this.selectedTime = "00:00:00";
    this.selectedTimeCpy = "00:00:00";
    this.CancelEvent();
  }

  public CancelEvent(): void {
    
    if(this.pollingEvent !== -1)
    {
      clearInterval(this.pollingEvent);
      this.pollingEvent = -1
    }
  }

  public timerEvent(): void{

    let selectedDateTime = new Date('1970-01-01T' + this.selectedTime);
    selectedDateTime.setTime(selectedDateTime.getTime() - 1000)
    
    this.selectedTime = `${selectedDateTime.getHours()<10 ? '0' + selectedDateTime.getHours() : selectedDateTime.getHours()}:` +
                        `${selectedDateTime.getMinutes()<10 ? '0' + selectedDateTime.getMinutes() : selectedDateTime.getMinutes()}:` +
                        `${selectedDateTime.getSeconds()<10 ? '0' + selectedDateTime.getSeconds() : selectedDateTime.getSeconds()}`;  

    if(this.selectedTime === "00:00:00")
    {
      this.CancelEvent();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.timer{
  display:block;
  padding-top:50px;
}
.control-btns{
  margin:10px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
