<template>
  <v-container>
    <v-row class="text-center">
    <v-col cols="12">
      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          AD Risk Predict Model 
        </h1>
      </v-col>

      <v-row 
        align="center"
        justify="space-around"
      >
        <v-col cols="4">
        </v-col>
        <v-col cols="4" class="text-left pl-16 pd-6">
          <h3> Please answer </h3>
        </v-col>
        <v-col cols="4">
          <h3> Importance </h3>
        </v-col>
      </v-row>

      <v-card 
        v-for="f in features"
        :key="f.name"
        outlined
        class="pa-0 pl-2 ma-0"
      >
      <v-row 
        align="center"
        justify="space-around"
        color="red"
      >
        <v-col cols="4" class="text-right">
          <p> {{ f.question }} </p>
        </v-col>

        <v-col cols="4" class="pl-16">
          <v-row justify="start" v-if="f.select">
          <v-radio-group row v-model="f.ans">
            <!--eslint-disable-next-line-->
            <div v-for="n in f.answers.length" >
            <v-radio :label="f.answers[n-1]" :value="n-1"></v-radio>
            </div>
          </v-radio-group>
          </v-row>

          <v-col cols="3" v-if="!f.select" class="ma-0 pa-0">
            <v-text-field class="pa-0 ma-0" v-model="f.ans">
            </v-text-field>
          </v-col>
        </v-col>
        
        <v-col cols="4">
<!--           <h3> {{ f.ans }} </h3> -->
          <v-rating
            v-if="sex != null"
            background-color="indigo lighten-3"
            color="indigo"
            medium
            readonly
            :value="f.imp[sex]"
          ></v-rating>
        </v-col>
      </v-row>

      </v-card>
      <v-row justify="space-around" class="ma-6">
              <v-btn 
                color="pink"
                dark
                @click="reset"
              > reset </v-btn> 
              <v-btn 
                color="green"
                dark
                @click="submit"
              > submit </v-btn> 
      </v-row>
  
    </v-col>
    </v-row>
    <v-dialog v-model="dialog" persistent max-width="330">
      <v-card>
        <v-card-title class="headline">Results</v-card-title>
        <v-card-text >
          <div v-if="label != null">
            <p style="color:black;">Your baby's probability of getting AD by the age of 6 months is <b style="color:red;">{{ risk }}</b>.
            <br><br>
              The AD risk is <b style="color:red;">{{ level }}</b>.</p>
            <br>
            For further information, please seek medical advice by your family pediatrician.
          </div>
          <div v-if="label == null">
            <h1> error </h1>
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="dialog = false">Agree</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from 'axios'

  export default {
    name: 'HelloWorld',

    data: () => ({
      host: '',
      dialog: false,
      risk: '',
      level: '',
      label: null,

      features: [
        {
          name: 'sex',
          question: 'Child sex?',
          answers: ['boy', 'girl'],
          ans: null,
          select: true,
          imp: [5, 5],
        },  
        {
          name: 'MAE',
          question: 'Maternal had atopic dermatitis history?',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [4, 3],
        },
        {
          name: 'FAE',
          question: 'Paternal had atopic dermatitis history?',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [3, 3],
        },
        {
          name: 'MAR',
          question: 'Maternal allergic rhinitis',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [4, 4],
        },
        {
          name: 'FAR',
          question: 'Paternal allergic rhinitis',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [3, 5],
        },
        {
          name: 'MAz',
          question: 'Maternal asthma',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [1, 1],
        },
        {
          name: 'FAz',
          question: 'Paternal asthma',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [0, 1],
        },
        {
          name: 'Aa28',
          question: 'Natural labor or Caesarean section?',
          answers: ['Natural', 'Caesarean'],
          ans: null,
          select: true,
          imp: [2, 1],
        },
        {
          name: 'Preg Ren',
          question: 'House renovation during pregnancy',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [0, 2],
        },
        {
          name: 'Preg Paint2',
          question: 'House furniture painting during pregnancy',
          answers: ['no', 'yes'],
          ans: null,
          select: true,
          imp: [1, 2],
        },
        {
          name: 'Aa1',
          question: 'What number is your baby in the order?',
          ans: null,
          select: false,
          imp: [1, 2],
        },
        {
          name: 'B Wt',
          question: 'Birthweight',
          ans: null,
          select: false,
          imp: [0, 0],
        },
        {
          name: 'Breastmilk Dur',
          question: 'Duration of breastfeeding',
          ans: null,
          select: false,
          imp: [1, 0],
        },
        {
          name: 'MAge',
          question: 'Maternal age at childbirth',
          ans: null,
          select: false,
          imp: [2, 4],
        },
        {
          name: 'FAge',
          question: 'Paternal age at childbirth',
          ans: null,
          select: false,
          imp: [3, 1],
        },
        {
          name: 'MEdu',
          question: 'Maternal education',
          ans: null,
          select: false,
          imp: [5, 5],
        },
        {
          name: 'FEdu',
          question: 'Paternal education',
          ans: null,
          select: false,
          imp: [5, 3],
        },
        {
          name: 'Gestation',
          question: 'Gestational week',
          ans: null,
          select: false,
          imp: [1, 0],
        },
        {
          name: 'K6',
          question: 'How many fungus walls at home?',
          ans: null,
          select: false,
          imp: [2, 2],
        },
        {
          name: 'Menarche',
          question: 'Mother meanarche',
          ans: null,
          select: false,
          imp: [2, 1],
        },
      ],
    }),
    
    computed: {
      sex() {
        return this.features[0].ans;
      }
    },
    
    watch: {
      sex: function(val) {
        if (val == 0) {
          console.log("boy");
          this.features = this.features.sort(function (a, b) {
            //return a.imp[0] <= b.imp[0] ? 1 : -1;
            return b.imp[0] - a.imp[0];
          });
        
        } else if (val == 1) {
          console.log("girl");
          this.features = this.features.sort(function (a, b) {
            //return a.imp[1] <= b.imp[1] ? 1 : -1;
            return b.imp[1] - a.imp[1];
          });
        }
      }
    },

    methods: {
      submit() {
        if (this.sex != null) {
        var data = {};
        for(let i=0; i<this.features.length; ++i) {
          data[this.features[i].name] = this.features[i].ans;
          // Aa28 在後端 +1 
        }
          
        console.log(data);
        const path = this.host + '/submit';
        axios.post(path, data)
          .then((res) => {
            this.label = res.data.split(' ')[0];
            console.log(res.data.split(' '));
            if (this.label == 0) {
              this.level = 'low';
              if (this.sex == 0) this.risk = 'lower than 4%';
              else this.risk = 'lower than 3%';
              
            } else if (this.label == 1) {
              this.level = 'low-moderate';
              if (this.sex == 0) this.risk = 'between 4% ~ 8%';
              else this.risk = 'between 3% ~ 6%';
              
            } else if (this.label ==2) {
              this.level = 'high-moderate';
              if (this.sex == 0) this.risk = 'between 8% ~ 16%';
              else this.risk = 'between 6% ~ 12%';
              
            } else if (this.label ==3) {
              this.level = 'high';
              if (this.sex == 0) this.risk = 'higher than 16%';
              else this.risk = 'higher than 12%';
            }
            this.dialog = true;
            console.log(this.label);
          })
          .catch((error) => {
            console.error(error);
          });
        } else {
          console.log('error');
        }
        
      },

      reset() {
        for(let i=0; i<this.features.length; ++i) {
          this.features[i].ans = null;
        }
      },

    },

    created() {
      this.host = 'http://' + document.location.hostname + ':1234';
    },
  }
</script>
