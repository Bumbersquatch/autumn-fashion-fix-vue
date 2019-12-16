<template>
  <section class="container text-center carousel-text">
    <div class="event-brand">
      <h1 class="display-3">
        Autumn<br>Fashion<br>Fix
      </h1>
    </div>
    <div class="event-date">
      <p>
        <span>23-26</span> September <span>2020</span>
      </p>
    </div>
    <div class="event-info row">
      <div class="col-12 text-center">
        <div class="countdown countdown-container container">
          <div :class="loaded ? 'clock row show' : 'clock row'">
            <div class="col-6 col-lg-3">
              <div ref="clockItem" class="countdown-item">
                <div class="content">
                  <svg class="countdown-svg">
                    <path :stroke="variables.accentcolor" :d="describeArc(cdidiameter, cdidiameter, cdidiameter-2, 0, daysRadius)" stroke-width="4" fill="none" />
                  </svg>
                  {{ days }}
                  <span>days</span>
                </div>
              </div>
            </div>
            <div class="col-6 col-lg-3">
              <div class="countdown-item">
                <div class="content">
                  <svg class="countdown-svg">
                    <path :stroke="variables.accentcolor" :d="describeArc(cdidiameter, cdidiameter, cdidiameter-2, 0, hoursRadius)" stroke-width="4" fill="none" />
                  </svg>
                  {{ hours }}
                  <span>hours</span>
                </div>
              </div>
            </div>
            <div class="col-6 col-lg-3">
              <div class="countdown-item">
                <div class="content">
                  <svg class="countdown-svg">
                    <path :stroke="variables.accentcolor" :d="describeArc(cdidiameter, cdidiameter, cdidiameter-2, 0, minutesRadius)" stroke-width="4" fill="none" />
                  </svg>
                  {{ minutes }}
                  <span>minutes</span>
                </div>
              </div>
            </div>
            <div class="col-6 col-lg-3">
              <div class="countdown-item">
                <div class="content">
                  <svg class="countdown-svg">
                    <path :stroke="variables.accentcolor" :d="describeArc(cdidiameter, cdidiameter, cdidiameter-2, 0, secondsRadius)" stroke-width="4" fill="none" />
                  </svg>
                  {{ seconds }}
                  <span>seconds</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import moment from 'moment'
import variables from '~/assets/sass/_variables.scss'

export default {
  data () {
    return {
      days: 0,
      hours: 0,
      minutes: 0,
      seconds: 0,
      diameter: 0,
      cdidiameter: 0,
      variables,
      loaded: false
    }
  },
  computed: {
    daysRadius () {
      return this.mapNumber(this.days, 365, 0, 0, 360)
    },
    hoursRadius () {
      return this.mapNumber(this.hours, 24, 0, 0, 360)
    },
    minutesRadius () {
      return this.mapNumber(this.minutes, 60, 0, 0, 360)
    },
    secondsRadius () {
      return this.mapNumber(this.seconds, 60, 0, 0, 360)
    }
  },
  mounted () {
    if(this.$refs.clockItem) {
      this.diameter = this.$refs.clockItem ? this.$refs.clockItem.offsetWidth : 0
      this.cdidiameter = this.diameter / 2
      setTimeout(() => {
        this.loaded = true
      }, 1000)
      
    }
    const then = moment('23-09-2020', 'DD-MM-YYYY')
    setInterval(() => {
      const now = moment()
      const countdown = moment(then - now)
      this.days = then.diff(now, 'days')
      this.hours = countdown.format('HH')
      this.minutes = countdown.format('mm')
      this.seconds = countdown.format('ss')
    }, 1000)

  },
  methods: {
    mapNumber (number, inMin, inMax, outMin, outMax) {
      return (number - inMin) * (outMax - outMin) / (inMax - inMin) + outMin
    },
    polarToCartesian (centerX, centerY, radius, angleInDegrees) {
      const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0
      return {
        x: centerX + (radius * Math.cos(angleInRadians)),
        y: centerY + (radius * Math.sin(angleInRadians))
      }
    },
    describeArc (x, y, radius, startAngle, endAngle) {
      const start = this.polarToCartesian(x, y, radius, endAngle)
      const end = this.polarToCartesian(x, y, radius, startAngle)
      const largeArcFlag = endAngle - startAngle <= 180 ? '0' : '1'
      const d = [
        'M', start.x, start.y,
        'A', radius, radius, 0, largeArcFlag, 0, end.x, end.y
      ].join(' ')
      return d
    }
  }
}

</script>
