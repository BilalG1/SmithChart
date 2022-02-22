<template>
<div @click="checkPageClick" class="smith-chart-page">
  <Transition mode="out-in" name="pane-slide">
    <div class="options" id="options-pane" v-if="options.showPane">

      <div class="options-top-bar">
        <button class="show-add-items-button" id="show-add-items" @click="options.showAddItems = !options.showAddItems" :style="options.showAddItems ? {background: '#d0d0d0'}:{}">+</button>
        <button class="show-add-items-button" id="show-settings" @click="options.showSettings = !options.showSettings" :style="options.showSettings ? 'font-size: 22px; background: #d0d0d0;' : 'font-size: 20px'"> &#9881; </button>
        <button class="show-add-items-button" id="show-carrots" @click="toggleOptionsPane(false)" style="font-size: 16px; font-weight: 600; margin-right: 10px"> &lt;&lt; </button>
        <Transition name="add-list">
          <div class="add-lines-wrapper add-items-stay-open" v-if="options.showAddItems">
            <button @click="setActiveAdd('Point')" class="add-items-stay-open" :style="options.activeAdder=='Point' ? 'opacity: 1; background: #e0e0e0;' : ''">Point</button>
            <button @click="setActiveAdd('VSWR Circle')" class="add-items-stay-open" :style="options.activeAdder=='VSWR Circle' ? 'opacity: 1; background: #e0e0e0;' : ''">VSWR Circle ( r + jx )</button>
            <button @click="setActiveAdd('VSWR Circle Gamma')" class="add-items-stay-open" :style="options.activeAdder=='VSWR Circle Gamma' ? 'opacity: 1; background: #e0e0e0;' : ''">VSWR Circle ( | &Gamma; | )</button>
            <button @click="setActiveAdd('Resistance Circle')" class="add-items-stay-open" :style="options.activeAdder=='Resistance Circle' ? 'opacity: 1; background: #e0e0e0;' : ''">Resistance Circle</button>
            <button @click="setActiveAdd('Reactance Curve')" class="add-items-stay-open" :style="options.activeAdder=='Reactance Curve' ? 'opacity: 1; background: #e0e0e0;' : ''">Reactance Curve</button>
            <button @click="setActiveAdd('Wavelength')" class="add-items-stay-open" :style="options.activeAdder=='Wavelength' ? 'opacity: 1; background: #e0e0e0;' : ''">Wavelength Line</button>
          </div>
          <div class="add-lines-wrapper settings-region-el" v-else-if="options.showSettings">
            <div class="settings-row settings-region-el" v-for="(set, index) in Object.keys(options.settings)" :key="set" @click="options.settings[set] = !options.settings[set]">
              <input type="checkbox" v-model="options.settings[set]" class="settings-region-el"/>
              <span class="settings-region-el">{{formatSettingLabel(index)}}</span>
            </div>
            <div @click="options.showMoreBox = !options.showMoreBox" class="settings-row settings-show-more-box">
              More
            </div>
          </div>
          <div v-else-if="options.showMoreBox" class="add-lines-wrapper more-info-box">
            <span>
              <b>Quick Tools: </b>
              <br>
              <br>
              <li>Click colored icon to the left of mark to toggle showing it on chart</li>
              <br>
              <li>Double click any mark from list to transfer to admittance (reflect point across origin, add 0.25 wavelength, etc)</li>
              <br>
              <li>Hold 's' key and double click any mark to switch between types (point -> vswr point -> ... -> point)</li>
              <br>
              <li>Hold 'd' key and double click any mark to duplicate the mark</li>
              <br>
              <br>
              <br>
              <b>For Proffessor Robert Candler, ECE 101A, Winter '22</b>
            </span>
          </div>
        </Transition>
      </div>

      <div class="points-list-wrapper">
        <TransitionGroup name="plist" tag="div">
          <div v-for="(m, index) in marks" :key="m.r+m.color" :class="{'points-list': true, 'points-list-select': m.select == indexSelect}">
            <div class="point-index-tracker" @click="toggleShowMark(index)" :style="m.select == indexSelect ? 'background: #6399CB' : ''">
              <div class="index-label" :style="m.select == indexSelect ? 'color: white': ''">
                {{index+1}}
              </div>
              <svg width="24px" height="24px">
                <circle v-if="m.type == 'Point'" r="12" :fill="m.hide ? 'lightgray' : m.color" cx="12" cy="12"/>
                <circle v-else r="12" fill="white" cx="12" cy="12"/>

                <circle v-if="m.type == 'Point'" r="4" fill="white" cx="12" cy="12"/>
                <line v-else-if="m.type == 'Wavelength'" x1="19" y1="24" x2="5" y2="0" :stroke="m.hide ? 'lightgray' : m.color" stroke-width="3" />
                <circle v-else-if="m.type != 'Reactance Curve'" r="10" stroke-width="4" :stroke="m.hide ? 'lightgray' : m.color" fill="white" cx="12" cy="12"/>

                <line v-if="m.type == 'Wavelength'" x1="12" y1="12" x2="5" y2="24" :stroke="m.hide ? 'lightgray' : m.color" stroke-width="3" />
                <circle v-if="m.type == 'Reactance Curve'" :stroke="m.hide ? 'lightgray' : m.color" r="24" stroke-width="4" fill="transparent" cx="24" cy="-12"/>
                <circle v-if="m.type == 'Reactance Curve'" :stroke="m.hide ? 'lightgray' : m.color" r="24" stroke-width="4" fill="transparent" cx="24" cy="36"/>
              </svg>
            </div>
            <div class="point-formatted" @click="editMark(index)" @dblclick="specialClickMark(index)" v-html="formatMark(m)">
            </div>
            <button class="remove-point" @click="removeMark(m)">X</button>
          </div>
        </TransitionGroup>
      </div>

    </div>

    <div class="pane-else" v-else>
      <button class="show-add-items-button pane-show-button" @click="toggleOptionsPane(true)"> &gt;&gt; </button>
    </div>
  </Transition>


  <div class="smith-chart-holder" id="smith-chart-holder" >
    <div class="chart-coords" v-html="currentChartCoords ? currentChartCoords : 'Hover over the chart'">
    </div>

    <smith-chart v-if="smithChartRadius" ref="chart" @mousedown.prevent="" @mousemove="updateChartLines" @mouseup="confirmChartMark" :label-rings="options.settings.labelRings" :resistance-labels="options.settings.resistanceLabels" :reactance-labels="options.settings.reactanceLabels" :radius="smithChartRadius">
      <sm-point v-for="p in marks.filter(m => m.type == 'Point' && !m.hide)" :key="p.r + p.color" :res="p.r" :react="p.x" :fill="p.color" :r="(p.select == indexSelect ?  7 : 5)"/>
      
      <sm-vswr-circle v-for="c in marks.filter(m => (m.type == 'VSWR Circle' || m.type == 'VSWR Circle Gamma') && !m.hide)" :key="c.r + c.color" :res="c.r" :react="c.x" :stroke="c.color" :stroke-width="(c.select == indexSelect ? 4 : 2)" :fill="colorToFillColor(c.color)" :showPoint="c.type=='VSWR Circle'"/>
      
      <sm-res-circle v-for="rc in marks.filter(m => m.type == 'Resistance Circle' && !m.hide)" :key="rc.r + rc.color" :res="rc.r" :fill="colorToFillColor(rc.color)" :stroke="rc.color" :stroke-width="(rc.select == indexSelect ? 4 : 2)"/>

      <sm-react-curve v-for="xc in marks.filter(m => m.type == 'Reactance Curve' && !m.hide)" :key="xc.x + xc.color" :react="xc.x" :fill="colorToFillColor(xc.color)" :double="options.settings.mirrorReactances" :stroke="xc.color" :stroke-width="(xc.select == indexSelect ? 4 : 2)"/>
    </smith-chart>
    <canvas id="chart-canvas"/>
  </div>
</div>
</template>

<script>
import SmPoint from './components/sm-point.vue'
import SmVswrCircle from './components/sm-vswr-circle.vue'
import SmithChart from './components/smith-chart.vue'
import SmResCircle from'./components/sm-res-circle.vue'
import SmReactCurve from './components/sm-react-arc.vue'

export default {
  components: {
    SmithChart,
    SmPoint,
    SmVswrCircle,
    SmResCircle,
    SmReactCurve,
  },
  data() {
    return {
      smithChartRadius: 400,
      colors: ['blue', 'green', 'purple', 'black', 'red'],
      curColor: 0,
      indexSelect: -1,

      currentChartCoords: '',
      marks: [],
      
      options: {
        showPane: true,
        showAddItems: false,
        showSettings: false,
        showMoreBox: false,

        activeAdder: 'Point',
        settings: {
          labelRings: true,
          resistanceLabels: true,
          reactanceLabels: true,
          fillCircles: false,
          mirrorReactances: false,
        }
      },
      updatingChart: false,
      updatingChartText: false,
      curKeyDown: {},
    }
  },
  mounted(){
    window.addEventListener('resize', this.windowSizeChange);
    this.windowSizeChange('bypass');

    window.addEventListener('keydown', (e) => this.curKeyDown[e.key] = true)
    window.addEventListener('keyup', (e) => {
      if(this.curKeyDown[e.key])
        this.curKeyDown[e.key] = false;
    });
  },
  methods: {
    roundValues(r0,x0){
      let r = Math.round(r0 * 100) / 100;
      let x = Math.round(x0 * 100) / 100;

      if(Math.abs(Math.round(r*10)/10 - r) <= .02)
        r = Math.round(r*10)/10;
      if(Math.abs(Math.round(x*10)/10 - x) <= .02)
        x = Math.round(x*10)/10

      return [r,x]
    },
    updateChartLines(e){
      let [r, x] = this.cartesianToResistReact(e.clientX, e.clientY);
      this.currentChartCoords = r >= 0 ? this.formatMark({r: r, x: x, type: this.options.activeAdder}) : '';

      if(r < 0)
        return;
      if(this.options.activeAdder != 'Wavelength')
        [r,x] = this.roundValues(r,x);

      if((this.options.activeAdder == 'VSWR Circle' || this.options.activeAdder == 'VSWR Circle Gamma'))
        this.updateCircle(e);
      else if(this.options.activeAdder == 'Resistance Circle')
        this.updateResCircle(r);
      else if(this.options.activeAdder == 'Reactance Curve')
        this.updateReactCurve(x);
      else if(this.options.activeAdder == 'Wavelength')
        this.updateWavelengthLine(e,r,x);
    },
    updateCircle(e){
      let canv = document.getElementById('chart-canvas')
      let ctx = canv.getContext('2d');
      let chartRadius = this.$refs.chart.radius;

      let p = document.getElementById('center-chart-ref').getBoundingClientRect();
      let r = Math.sqrt(Math.pow(e.clientX - p.left - 3, 2) + Math.pow(e.clientY - p.top - 3, 2));

      this.clearCanvasNewLines();
      ctx.beginPath();
      ctx.arc(chartRadius, chartRadius, r, 0, 2*Math.PI);
      ctx.stroke();
    },
    updateResCircle(res){
      let ctx = document.getElementById('chart-canvas').getContext('2d');
      let chartRadius = this.$refs.chart.radius;
      let r = chartRadius / (res + 1);

      this.clearCanvasNewLines();
      ctx.beginPath();
      ctx.arc(2*chartRadius - r, chartRadius, r, 0, 2*Math.PI);
      ctx.stroke();
    },
    updateReactCurve(x){
      let canv = document.getElementById('chart-canvas')
      let ctx = canv.getContext('2d');
      let chartRadius = this.$refs.chart.radius;

      this.clearCanvasNewLines();
      ctx.beginPath();
      ctx.arc(2*chartRadius, -chartRadius / x + chartRadius, chartRadius / Math.abs(x), 0, 2*Math.PI);
      ctx.stroke();
    },
    updateWavelengthLine(e, r, x){
      let canv = document.getElementById('chart-canvas')
      let ctx = canv.getContext('2d');
      let chartRadius = this.$refs.chart.radius;

      let a = ((Math.pow(r, 2) - 1 + Math.pow(x, 2)) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * chartRadius;
      let b = ((2*x) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * chartRadius;
      a = (a == 0 ? .01 : a);
      let theta = Math.atan(b / a);
      theta = (a < 0 && b > 0 ? theta + Math.PI : theta);
      theta = (a < 0 && b <= 0 ? theta - Math.PI : theta);

      this.clearCanvasNewLines();
      ctx.beginPath();
      ctx.moveTo(chartRadius, chartRadius);
      ctx.lineTo(chartRadius + chartRadius*Math.cos(theta), chartRadius - chartRadius*Math.sin(theta));
      ctx.stroke();
    },
    confirmChartMark(e){
      let [r, x] = this.cartesianToResistReact(e.clientX, e.clientY);

      if(r >= 0){
        if(this.options.activeAdder != 'Wavelength')
          [r,x] = this.roundValues(r,x);
        if(this.options.activeAdder == 'Reactance Curve' && x == 0)
          x = .0004;
        if(this.curColor == 3 && (this.options.activeAdder == 'Resistance Circle' || this.options.activeAdder == 'Reactance Curve'))
          this.curColor = 4;
        if(this.options.activeAdder == 'Resistance Circle')
          x = 0;
        if(this.options.activeAdder == 'Reactance Curve')
          r = 0;
        this.marks.push({r: r, x: x, type: this.options.activeAdder, color: this.colors[this.curColor]});
        this.curColor = (this.curColor + 1) % this.colors.length;
      }

      this.clearCanvasNewLines();
    },
    cartesianToResistReact(a, b) {
      const rect = document.getElementById('origin-ref-circle').getBoundingClientRect()
      a = a - rect.left;
      b = b - rect.top;

      a = (a - this.$refs.chart.radius) / this.$refs.chart.radius;
      b = -1* (b - this.$refs.chart.radius) / this.$refs.chart.radius;

      let r = (1 - Math.pow(a,2) - Math.pow(b,2)) / (Math.pow(b,2) + Math.pow((a - 1),2));
      let x = (-b - b*r) / (a - 1);

      return [r, x];
    },
    removeMark(p){
      for(let i in this.marks){
        if(this.marks[i].r == p.r && this.marks[i].x == p.x && this.marks[i].color == p.color && this.marks[i].type == p.type){
          this.marks.splice(i, 1);
          return;
        }
      }
    },
    formatMark(p){
      let [r,x] = this.roundValues(p.r, p.x);
      x = Math.abs(x);

      if(p.type == 'Point')
        return p.x > 0 ? (r + ' + j' + x) : (r + ' - j' + x)

      else if(p.type == 'Resistance Circle')
        return p.type + ' (r = ' + r + ')';

      else if(p.type == 'VSWR Circle Gamma'){
        let radius = this.$refs.chart.radius;
        let a = ((Math.pow(r, 2) - 1 + Math.pow(x, 2)) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * radius;
        let b = -((2*x) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * radius;
        let Gamma = (Math.sqrt(Math.pow(a, 2) + Math.pow(b, 2))) / radius;
        Gamma = Math.round(Gamma * 100) / 100;
        return 'VSWR Circle ' + '(' + '| &Gamma; |' + ' = ' + Gamma + ')';
      }

      else if(p.type == 'Reactance Curve'){
        if(this.options.settings.mirrorReactances)
          return p.type + ' (x = &#177;' + x + ')';
        return p.type + ' (x = ' + (p.x > 0 ? x : -x) + ')';
      }

      else if(p.type == 'Wavelength'){
        let radius = this.$refs.chart.radius;
        let a = ((Math.pow(r, 2) - 1 + Math.pow(x, 2)) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * radius;
        let b = ((2*p.x) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * radius;
        a = (a == 0 ? .01 : a);
        let theta = Math.atan(b / a);
        theta = (a < 0 && b > 0 ? theta + Math.PI : theta);
        theta = (a < 0 && b <= 0 ? theta - Math.PI : theta);
        let lambda = Math.round(1000 * ((-theta / Math.PI * .25) + .25)) / 1000;
        return '&lambda; = ' + lambda;
      }

      else 
        return p.type + ' (' + r + (p.x > 0 ? ' + j' : ' - j') + x + ')';
    },
    setActiveAdd(str){
      this.options.activeAdder = str;
    },
    editMark(ind){
      this.indexSelect++;
      this.marks[ind].select = this.indexSelect;
      console.log(ind);
    },
    checkPageClick(e){
      let className = e.target.className;
      let id = e.target.id
      if(!className || className != 'point-formatted')
        this.indexSelect++;
      if(this.options.showAddItems && id != 'show-add-items' && (!className.includes || !className.includes('add-items-stay-open')))
        this.options.showAddItems = false;
      if(this.options.showSettings && id != 'show-settings' && (!className.includes || !className.includes('settings-region-el')))
        this.options.showSettings = false;
      if(this.options.showMoreBox && (!className.includes || !className.includes('settings-show-more-box')))
        this.options.showMoreBox = false;

      this.clearCanvasNewLines();
    },
    toggleShowMark(ind){
      this.marks[ind].hide = this.marks[ind].hide ? false : true;
    },
    formatSettingLabel(ind){
      let arr = ['Outer labels', 'Resistance labels', 'Reactance labels', 'Fill regions', 'Mirror Reactances']
      return arr[ind];
    },
    colorToFillColor(col){
      if(!this.options.settings.fillCircles)
        return 'none';
      let map = {
        red: 'rgba(255,0,0,.2)',
        green: 'rgba(0,255,0,.2)',
        blue: 'rgba(0,0,255,.2)',
        purple: 'rgba(200,0,200,.2)',
        black: 'rgba(0,0,0,.2)'
      }
      return map[col];
    },
    windowSizeChange(s){
      //set smithChartRadius and positioning in center of opening
      let [width, height] = this.getClientWidthHeight();
      let optionsPane = document.getElementById('options-pane');
      let optionsWidth = optionsPane ? optionsPane.offsetWidth : 0;
      
      this.smithChartRadius = 0;
      setTimeout(() => {        
        if(s == 'bypass')
          width = this.getClientWidthHeight()[0];
        if(width == this.getClientWidthHeight()[0]){
          if(width > 700)
            this.smithChartRadius = Math.min((width - Math.max(optionsWidth, 100) - 160) / 2, (height - 175) / 2,400)
          else{
            this.smithChartRadius = (width / 2) - 100;
            optionsWidth = 0;
          }
          document.getElementById('smith-chart-holder').style.marginLeft = optionsWidth + 'px';
          this.setCanvasPos();
          this.setChartTextSize();
        }
      }, 100);
    },
    setCanvasPos(){
      if(!document.getElementById('origin-ref-circle') || !this.$refs.chart){
        if(!this.updatingChart){
          setTimeout(this.setCanvasPos, 100);
          this.updatingChart = true;
        }
        return;
      }
      let p = document.getElementById('origin-ref-circle').getBoundingClientRect();
      let d = 2 * this.$refs.chart.radius;
      let canvas = document.getElementById('chart-canvas')
      canvas.width = d;
      canvas.height = d;
      canvas.style.left = p.left + 'px';
      canvas.style.top = p.top + 'px';
      this.updatingChart = false;
      this.clearCanvasNewLines();
    },
    setChartTextSize(){
      let texts = document.getElementsByClassName('text-labels');
      if(!texts.length){
        if(!this.updatingChartText){
          setTimeout(this.setChartTextSize, 100);
          this.updatingChartText = true;
        }
        return;
      }
      if(this.smithChartRadius < 200 && texts[0].style.fontSize != '8px'){
        for(let t of texts){
          t.style.fontSize = '8px';
        }
      }
      else if(this.smithChartRadius < 300 && texts[0].style.fontSize != '10px'){
        for(let t of texts){
          t.style.fontSize = '10px';
        }
      }
      else if(this.smithChartRadius > 300 && texts[0].style.fontSize != '12px'){
        for(let t of texts){
          t.style.fontSize = '12px';
        }
      }
      this.updatingChartText = false;
    },
    getClientWidthHeight(){
      var width = window.innerWidth
      || document.documentElement.clientWidth
      || document.body.clientWidth;

      var height = window.innerHeight
      || document.documentElement.clientHeight
      || document.body.clientHeight;
      
      return [width, height];
    },
    clearCanvasNewLines(){
      let canv = document.getElementById('chart-canvas')
      let ctx = canv.getContext('2d');
      let chartRadius = this.$refs.chart.radius;
      ctx.clearRect(0, 0, canv.width, canv.height);

      for(let wavelengthMark of this.marks.filter(m => m.type == 'Wavelength' && !m.hide)){
        let [r,x] = [wavelengthMark.r, wavelengthMark.x];
        let a = ((Math.pow(r, 2) - 1 + Math.pow(x, 2)) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * chartRadius;
        let b = ((2*x) / (Math.pow(r+1, 2) + Math.pow(x, 2))) * chartRadius;
        a = (a == 0 ? .01 : a);

        let theta = Math.atan(b / a);
        theta = (a < 0 && b > 0 ? theta + Math.PI : theta);
        theta = (a < 0 && b <= 0 ? theta - Math.PI : theta);

        ctx.strokeStyle = wavelengthMark.color;
        ctx.lineWidth = (wavelengthMark.select == this.indexSelect ? 4 : 2);

        ctx.beginPath();
        ctx.moveTo(chartRadius, chartRadius);
        ctx.lineTo(chartRadius + chartRadius*Math.cos(theta), chartRadius - chartRadius*Math.sin(theta));
        ctx.stroke();
      }
      ctx.strokeStyle = 'black';
      ctx.lineWidth = 1;
    },
    toggleOptionsPane(show){
      this.options.showPane = show;
      let w = this.getClientWidthHeight()[0];
      let optionsWidth = this.options.showPane ? Math.max(280, .2*w+160) : 0;
      if(this.getClientWidthHeight()[0] > 700){
        document.getElementById('smith-chart-holder').style.marginLeft = optionsWidth + 'px';
        this.setCanvasPos();
      }
    },
    specialClickMark(ind){
      let mark = this.marks[ind];
      if(this.curKeyDown['s']){
        let typeArr = ['Point','VSWR Circle','VSWR Circle Gamma','Resistance Circle','Reactance Curve','Wavelength'];
        let newInd = (typeArr.findIndex((el) => el == mark.type) + 1 ) % 6;
        if(newInd == 4 && mark.x == 0)
          mark.x = 0.0004;
        mark.type = typeArr[newInd];
        this.editMark(ind);
      }
      else if(this.curKeyDown['d']){
        this.marks.push({type: mark.type, color: this.colors[this.curColor], r: mark.r, x: mark.x});
        this.curColor = (this.curColor + 1) % this.colors.length;
        this.editMark(this.marks.length - 1);
      }
      else{
        let r = (mark.type == 'Reactance Curve' ? 0 : mark.r);
        let x = (mark.type == 'Resistance Circle' ? 0 : mark.x);

        let g = r / (Math.pow(r, 2) + Math.pow(x, 2));
        let b = -x / (Math.pow(r, 2) + Math.pow(x, 2));

        this.marks.push({type: mark.type, color: this.colors[this.curColor], r: g, x: b});
        this.curColor = (this.curColor + 1) % this.colors.length;
        this.editMark(this.marks.length - 1);
      }
      setTimeout(this.clearCanvasNewLines, 50);
    }
  }
}
</script>

<style scoped>
.smith-chart-page{
  display: flex;
  justify-content: space-evenly;
}
.options{
  height: 100vh;
  top: 0;
  left: 0;
  width: calc(20vw + 160px);
  min-width: 280px;
  webkit-box-shadow: 0 -4px 6px rgb(0 0 0 / 20%);
  box-shadow: 0 -4px 6px rgb(0 0 0 / 20%);
  position: fixed;
  background: white;
  overflow-y: auto;
  overflow-x: clip;
  z-index: 300;
}
@media screen and (max-width: 700px) {
  .options{
    height: calc(100vh - 100vw - 50px);
    bottom: 0;
    top: auto;
    width: 100vw;  
    margin-bottom: 10px;
  }
  #show-carrots{
    transform: rotate(270deg);
  }
  .pane-show-button{
    transform: rotate(270deg);
  }
}

.options-top-bar{
  position: relative;
  background: #e0e0e0;
  display: flex;
  justify-content: space-between;
  background: linear-gradient(#fcfcfc, #eaeaea);
  height: 46px;
  border-bottom: 1px solid rgba(0,0,0,0.2);
  border-right: 1px solid rgba(0,0,0,0.2);
  align-items: center; 
}
.show-add-items-button{
  border: none;
  cursor: pointer;
  border-radius: 4px;
  width: 30px;
  height: 30px;
  font-size: 30px;
  transition: .3s all;
  font-weight: 600;
  margin-left: 15px;
  background: transparent;
  vertical-align: middle;
  line-height: 30px;
}
.show-add-items-button:hover{
  background: #d0d0d0;
}
.pane-else{
  position: fixed;
  top: 30px;
  left: 30px;
}
.pane-show-button{
  background: transparent;
  font-size: 24px;
  font-weight: 600; 
  opacity: .6;
  width: 38px;
  height: 38px;
}
.pane-show-button:hover{
  opacity: .8;
  background: rgba(204, 204, 204, 0.5);
}

.pane-slide-move,
.pane-slide-enter-active,
.pane-slide-leave-active {
  transition: all 0.3s ease;
}
.pane-slide-enter-from,
.pane-slide-leave-to {
  opacity: 0;
  transform: translateX(-80px)
}
.pane-slide-leave-active {
  position: absolute;
}

.add-list-move,
.add-list-enter-active,
.add-list-leave-active {
  transition: all 0.3s ease;
}
.add-list-enter-from,
.add-list-leave-to {
  opacity: 0;
  transform: translateY(-20px)
}
.add-list-leave-active {
  position: absolute;
}

.plist-move,
.plist-enter-active,
.plist-leave-active {
  transition: all 0.5s ease;
}
.plist-enter-from,
.plist-leave-to {
  opacity: 0;
  transform: translate3d(30px, 100px, 0);
}
.plist-leave-active {
  position: absolute;
  margin-top: -50px;
}

.points-list{
  position: relative;
  flex-direction: row;
  justify-content: space-between;
  font-family: math;
  font-family: Symbola, "Times New Roman", serif;
  font-size: 20px;
  height: 52px;
  border-top: 1px solid rgba(206,206,206,0.8);
  border-bottom: 1px solid transparent;
}
.points-list-select{
  border: 1px solid #6399CB;
}
.point-index-tracker{
  width: 38px;
  height: 52px;
  background: #00000008;
  left: 0;
  top: 0;
  display: inline-block;
  position: absolute;
  color: white;
  cursor: pointer;
}
.point-index-tracker svg{
  margin-top: 5px;
  border-radius: 50%;
}
.index-label{
  text-align: left;
  margin-left: 3px;
  color: rgba(0,0,0,0.5);
  font-size: 10px;
}
.point-formatted{
  padding: 15px 35px 9px 53px;
  text-align: left;
  cursor: pointer;

  user-select: none; 
  -webkit-user-select: none; 
  -moz-user-select: none;
  -khtml-user-select: none; 
  -ms-user-select: none;
}
.remove-point{
  background: white;
  font-family: cursive;
  cursor: pointer;
  border: none;
  font-weight: 700;
  color: lightgray;
  font-size: 16px;
  position: absolute;
  top: 0;
  right: 0;
  padding: 8px 7px 20px 20px;
}
.remove-point:hover{
  transition: .3s all;
  color: gray;
}
.add-lines-wrapper{
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 48px;
  left: 0;
  background: white;
  padding: 4px;
  z-index: 100;
  border: 1px solid rgba(0,0,0,0.2);
  border-radius: 6px;
  box-shadow: 0 5px 10px rgb(0 0 0 / 20%);
}
.add-lines-wrapper button{
  padding: 10px;
  border: none;
  color: black;
  margin: 5px 0;
  cursor: pointer;
  opacity: .5;
  background: white;
  text-align: left;
  border-radius: 8px;
}
.add-lines-wrapper button:hover{
  opacity: 1;
}
.settings-row{
  font-size: 14px;
  padding: 10px;
  text-align: left;
  opacity: .6;
  transition: .2s all;
  cursor: pointer;
}
.settings-row:hover{
  opacity: 1;
}
.settings-row input{
  margin-right: 10px;
}
.smith-chart-holder{
  width: calc(80vw + 120px);
}
.chart-coords{
  font-size: 20px;
  font-family: math;
  margin-top: 10px;
}
#chart-canvas{
  position: absolute;
  pointer-events: none;
  z-index: 200;
  border-radius: 50%;
}
.more-info-box{
  text-align: left;
  font-size: 12px;
  padding: 15px 10px 15px 5px;
}

@media screen and (max-width: 800px) and (min-width: 700px), screen and (max-width: 320px){
  .point-formatted{
    font-size: 18px;
  }
}
</style>