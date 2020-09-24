
<template>
  <!-- <v-container ref="conversationRef" class="fill-height" style="overflow:scroll"> -->
  <div>
    <div :style="{'margin-bottom':marginBottom}">
      <v-row
        no-gutters
        v-for="(message,index) in message"
        class="ma-2"
        :ref="'message'+index"
        :class="{'justify-end':message.type==='request'}"
        :key="index"
      >
        <v-col cols="1" v-if="message.type ==='response'" class="d-flex justify-end mr-2">
          <v-avatar color="blue darken-3" size="36">
            <span class="white--text">BD</span>
          </v-avatar>
        </v-col>
        <v-col cols="10" v-if="message.type==='response'" ref="response">
          <v-card
            flat
            class="px-5 py-2"
            style="border-radius:4px;position:relative;display:inline-flex"
            :class="{'grey lighten-4 triangle left-top':message.type==='response'}"
          >{{message.text}}</v-card>
        </v-col>
        <v-col cols="10" class="d-flex justify-end" v-if="message.type==='request'">
          <v-card
            flat
            class="px-5 py-2 mr-5 blue darken-3"
            style="border-radius:4px;position:relative;display:inline-flex"
            :class="{'white--text triangle right-top':message.type==='request'}"
          >{{message.text}}</v-card>
        </v-col>
        <v-col cols="1" v-if="message.type ==='request'" class="d-flex justify-start">
          <v-avatar class="blue darken-3 white--text" size="36">
            <span>VV</span>
          </v-avatar>
        </v-col>
      </v-row>
    </div>
    <v-row
      ref="inputRow"
      style="margin-top:10px;position:absolute;"
      v-if="currentMessage.optionStatus"
      class="d-flex justify-end bottom mr-4"
    >
      <div v-if="currentMessage.sessionAttributes == 'check'">
        <v-card
          md="3"
          v-for="(buttonValue, index) in currentMessage.responseCard"
          v-bind:key="index"
          color="white"
          class="px-3 py-1 mx-1 my-2 blue--text text--darken-3"

          outlined
          col-md-8
        >
          <label>
            <input
              type="checkbox"
              :value="buttonValue.value"
              :id="index"
              v-model="currentMessage.options"
              :disabled="currentMessage.buttonStatus"
            />
            {{ buttonValue.value }}
          </label>
        </v-card>
        <button
          @click="pushChat(currentMessage.options.join())"
          :disabled="currentMessage.buttonStatus"
          class="white--text blue darken-3 px-3 py-1 mx-1 my-2"
        >Send</button>
      </div>
      <div v-else class="text-right">
        <div class="justify-end d-flex" v-for="(buttonValue, index) in currentMessage.responseCard" :key="index">
          <v-btn
            :value="buttonValue.value"
            :id="index"
            outlined
            color="blue darken-3"
            :disabled="currentMessage.buttonStatus"
            @click.prevent="callMethod(buttonValue.value)"
            class="d-flex px-3 py-1 mr-0 my-2 justify-self-end"
          >{{ buttonValue.value }}</v-btn>
        </div>
      </div>
    </v-row>
    <input
      v-show="formStatus"
      v-model="inputValue"
      class="inputMessage"
      type="text"
      placeholder="Type here..."
      @keyup.enter="pushChat(inputValue)"
      :disabled="inputStatus"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      lexruntime: '',
      lexUserId: '',
      marginBottom: '100px',
      sessionAttributes: {},
      message: [],
      currentMessage: {},
      finalOutput: [],
      inputStatus: true,
      inputValue: '',

      fullFilled: false,
      onlineStatus: 'online',
      formStatus: false,
      finalOutput: {},
      offsetTop: 0
    }
  },
  computed: {
    target() {
      // const value = this.chatbot
      // if (!isNaN(value)) return Number(value)
    }
  },
  mounted() {
    // Initialize the Amazon Cognito credentials provider

    AWS.config.region = 'us-east-1' // Region
    AWS.config.credentials = new AWS.CognitoIdentityCredentials({
      // Provide your Pool Id here
      IdentityPoolId: 'us-east-1:72ce6932-3555-4a7e-b412-51ebce92e460'
    })
    this.lexruntime = new AWS.LexRuntime()
    this.lexUserId = 'chatbot-demo' + Date.now()
    this.$nextTick(() => {
      this.welcome('ent')
    })
  },

  // updated() {
  //   console.log(this.$refs.conversationRef)
  //   this.$refs.conversationRef.scrollTop = this.$refs.conversationRef.scrollHeight
  //   console.log(
  //     this.$refs.conversationRef.scrollTop,
  //     this.$refs.conversationRef.scrollHeight
  //   )
  // },
  methods: {
    callMethod(buttonValue) {
      this.pushChat(buttonValue)
      // this.$vuetify.goTo(this.$refs['message'+this.message.length-1][this.$refs['message'+this.message.length-1].length-1], {})
    },
    welcome(val) {
      let params = {
        botAlias: '$LATEST',
        botName: 'PreConsultation',
        inputText: val,
        userId: this.lexUserId,
        sessionAttributes: this.sessionAttributes
      }

      let vm = this
      this.lexruntime.postText(params, function(err, data) {
        if (err) {
          console.log(err, err.stack)
          //showError('Error:  ' + err.message + ' (see console for details)')
          vm.showError()
        }
        if (data) {
          console.log(data)
          // capture the sessionAttributes for the next cycle
          this.sessionAttributes = data.sessionAttributes
          // show response and/or error/dialog status

          vm.showResponseMsg(data)
        }
      })
    },
    pushChat(val) {
      if (val) {
        this.message[this.message.length - 1].buttonStatus = true
        this.message[this.message.length - 1].optionStatus = false
        console.log('optionStatus' + this.optionStatus)
        //to block the buttons the previously rendered array element.
        var params = {
          botAlias: '$LATEST',
          botName: 'PreConsultation',
          inputText: val,
          userId: this.lexUserId,
          sessionAttributes: this.sessionAttributes
        }

        this.showRequest(val)
        let vm = this
        this.lexruntime.postText(params, async function(err, data) {
          if (err) {
            console.log(err, err.stack)
            //showError('Error:  ' + err.message + ' (see console for details)')
            vm.showError()
          }
          if (data) {
            console.log(data + ' part2 data')
            // capture the sessionAttributes for the next cycle
            vm.sessionAttributes = data.sessionAttributes
            // show response and/or error/dialog status

            vm.showResponseMsg(data)
            await vm.$refs
            // vm.$vuetify.goTo(900,{})
            vm.marginBottom = vm.$refs.inputRow
              ? vm.$refs.inputRow.clientHeight + 'px'
              : '100px'
            vm.$vuetify.goTo(
              vm.$refs['message' + (vm.message.length - 1)][0],
              {}
            )
          }
        })
      } else {
        alert('Enter a valid input...')
      }
      this.inputValue = ''
    },
    showError() {
      this.message.push({ text: 'Try again later...', type: 'response' })
      this.onlineStatus = 'offline'
    },
    showRequest(daText) {
      this.message.push({ text: daText, type: 'request' })
    },
    showResponseMsg: function(data) {
      if (data.responseCard) {
        this.inputStatus = true
        this.formStatus = false
        this.message.push({
          text: data.message,
          type: 'response',
          sessionAttributes: data.sessionAttributes.type,
          responseCard: data.responseCard.genericAttachments[0].buttons,
          options: [],
          buttonStatus: false,
          optionStatus: true
        })
        this.currentMessage = this.message[this.message.length - 1]
      } else {
        this.formStatus = true
        this.inputStatus = false
        this.message.push({ text: data.message, type: 'response' })
      }
      if (data.dialogState === 'Fulfilled') {
        this.inputStatus = true
        this.formStatus = false

        this.finalOutput = data.sessionAttributes

        for (var a in this.finalOutput) {
          console.log(a.slice(1, a.length) + '---' + this.finalOutput[a])
        }
      }
    }
  }
}
</script>
<style scoped>
.borders {
  border: 1px solid green;
}
.inputMessage {
  border: 10px solid #1565c0;
  bottom: 0;
  height: 60px;
  left: 0;
  outline: none;
  padding-left: 10px;
  position: absolute;
  right: 0;
  width: 100%;
  margin-top: 5px;
}
.bottom {
  position: absolute;
  right: 0;
  bottom: 0;
}
.request:after {
  position: absolute;
  content: '';
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0px 10px 10px 0;
  border-color: transparent #fff transparent transparent;
  top: 0;
  left: -10px;
}

.triangle.left-top:after {
  content: ' ';
  position: absolute;
  width: 0;
  height: 0;
  left: -10px;
  right: auto;
  top: 0px;
  bottom: auto;
  border: 10px solid;
  border-color: #f5f5f5 transparent transparent transparent;
}

.triangle.right-top:before {
  content: ' ';
  position: absolute;
  width: 0;
  height: 0;
  left: auto;
  right: -10px;
  top: 0;
  border: 10px solid;
  bottom: auto;
  border-color: #1565c0 transparent transparent transparent;
}
</style>