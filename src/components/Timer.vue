<template>
  <div :class="[{'timer':true},{'alarm':soundAlarm}]">
    <!-- Clock Face Component -->
    <v-time-picker
      format="24hr"
      v-model="selectedTime"
      use-seconds
      @input="BackupTime()"
      ref="timepicker"
      :color="clockColor"
    ></v-time-picker>

    <!-- Timer Control Buttons -->
    <div >
     <!-- Start Timer Button -->
      <div class="control-btns">
        <v-btn
          elevation="2"      
          :color="pollingEvent === -1 ? 'success' : 'error'"
          @click="triggerTimer()"
          :disabled="selectedTime === '00:00:00'">{{(pollingEvent === -1 ? "Start" : "Stop") + " Timer"}}</v-btn>
      </div>

      <div>
        <!-- Reset Timer Button -->
        <v-btn
          class="control-btns"
          elevation="2"      
          color="warning"
          @click="RevertBackupTime()"
          :disabled="selectedTimeCpy === '00:00:00' || selectedTimeCpy === selectedTime ">Reset</v-btn>
        <!-- Clear Timer Button -->
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
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class Timer extends Vue {

  /* Declaration of $refs to allow access to the Veutify component */
  $refs!: {
    timepicker: HTMLFormElement
  }

  /* Variable Declarations */
  private selectedTime = "00:00:30";
  private selectedTimeCpy = "00:00:30";
  private pollingEvent?: number = -1;
  private pollingEventColour?: number = -1;
  private clockColor = "primary";
  private soundAlarm = false;


   /* Function to trigger the flashing of the time area to alert the user the timer has finished */
   public toggleflashColourFinish() : void {
      /* If polling is set to -1 then no interval is currently set so toggle on flashing color event */
      if(this.pollingEventColour === -1){
         /* set interval for half a second to trigger the color toggle for alert animation */
         this.pollingEventColour = setInterval(()=>{this.toggleColour()}, 500);
      }
      else{

         /* If toggle has been called whilst polling event already active we want to clear down the repeating event and initialise polling flag back to -1 */
         clearInterval(this.pollingEventColour)
         this.pollingEventColour = -1;
      }
   }
   /* Toggle event to flick between blue and red to alert the user of the timer finishing*/ 
   public toggleColour(): void {      
      if(this.clockColor == "primary")
      {
         this.clockColor = "error";
      }
      else{
            this.clockColor = "primary";
      }
   }

  /* Function to start the countdown function */
  public triggerTimer(): void {    

    try{
       /* Defend against no time being set and throw and error to prevent timer starting and looping round to 24 hours */
      if(this.selectedTime === "00:00:00")
      {
        throw "No Time Set";
      }

      /* If no timer started we can start a new one*/
      if(this.pollingEvent === -1){
        /* Show the clocker component with the second hands so we can see the hand animare round every second */
        this.$refs.timepicker.selecting = 3;
        /* Set timer event to fire every second */
        this.pollingEvent = setInterval(()=>{this.timerEvent()}, 1000);
      }
      else{
         /* Stop Timer */
         this.CancelEvent();
      }
    }
    catch(e){
       /* Print error msg to console */
      console.log(e);
    }    
  }
  
  /* Function to Stop timer and save the selected time in temporary variable in case the user chooses to revert back to it */
  public BackupTime() : void{
    this.CancelEvent();
    this.selectedTimeCpy = JSON.parse(JSON.stringify(this.selectedTime));
  }

  /* Function to reset the timer to the last set time */
  public RevertBackupTime() : void{
    this.selectedTime = JSON.parse(JSON.stringify(this.selectedTimeCpy));
    this.CancelEvent();
  }

  /*Function to clear down clock to zeros */
  public ClearSelection() : void{
    this.selectedTime = "00:00:00";
    this.selectedTimeCpy = "00:00:00";
    this.CancelEvent();
  }
  
  /* Function to terminate timer countdown event */
  public CancelEvent(): void {    
    if(this.pollingEvent !== -1)
    {
      clearInterval(this.pollingEvent);
      this.pollingEvent = -1;
    }
  }
  
  /* Fucntion to reduce clock time by one second give us a countdown */
  public timerEvent(): void{
    /* store current clock time in date object to allow easy of substracting 1 second from time */
    let selectedDateTime = new Date('1970-01-01T' + this.selectedTime);

    /*Change time to be 1 second less */
    selectedDateTime.setTime(selectedDateTime.getTime() - 1000);
    
    /*Formt time back to string that timepicker components can understand i.e. "00:30:00" with two digit numbers */
    this.selectedTime = `${selectedDateTime.getHours()<10 ? '0' + selectedDateTime.getHours() : selectedDateTime.getHours()}:` +
                        `${selectedDateTime.getMinutes()<10 ? '0' + selectedDateTime.getMinutes() : selectedDateTime.getMinutes()}:` +
                        `${selectedDateTime.getSeconds()<10 ? '0' + selectedDateTime.getSeconds() : selectedDateTime.getSeconds()}`;  

   /* if countdown has reached then end trigger alarm effects */
    if(this.selectedTime === "00:00:00")
    {
      this.soundAlarm = true;
      this.toggleflashColourFinish();
      setTimeout(()=>{
         this.soundAlarm = false; 
         this.toggleflashColourFinish();
         this.clockColor = "primary";
      },5000);
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

.alarm {
  animation: shake 0.82s cubic-bezier(.36,.07,.19,.97) both infinite;
  transform: translate3d(0, 0, 0);
  backface-visibility: hidden;
  perspective: 1000px;
}

@keyframes shake {
  10%, 90% {
    transform: translate3d(-1px, 0, 0);
  }
  
  20%, 80% {
    transform: translate3d(2px, 0, 0);
  }

  30%, 50%, 70% {
    transform: translate3d(-4px, 0, 0);
  }

  40%, 60% {
    transform: translate3d(4px, 0, 0);
  }
}

</style>
