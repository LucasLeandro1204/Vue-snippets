<template>
  <div id="wrapper">
    <div class="header">
      <p>We still need <strong>$ {{ needed }}</strong> to beat this project</p>
    </div>

    <div class="bar">
      <div :style="{ width: remainPercent + '%', 'background-color': barBackground }"></div>
    </div>

    <div class="wrapper">
      <p>There's <strong>{{ deadLine }} left</strong> to help this project.</p>
      <p>We have <strong v-text="donors"></strong> donors who believe and support this project, every dollar helps.</p>
      <div>
        <input type="text" v-model.number="donation" @focus="show = true" @blur="show = false" @keydown="show = false">
        <span>$</span>
        <button @click.prevent="donate">Contribute</button>
        <transition name="fade">
          <ul v-show="show">
            <li :class="{ active: suggestion == donation }" v-for="suggestion in suggestions" @click.prevent="donation = suggestion">$ {{ suggestion }}</li>
          </ul> 
        </transition>
      </div>
    </div>

    <div class="inputs">
      <button>Why give $ {{ donation || 0 }}?</button>
      <button>Share this project</button>
    </div>
  </div>
</template>

<script>
  import Store from './store';
  import Tween from '@tweenjs/tween.js';

  const _lastId = (lastID, donation) => (lastID > donation.id ? lastID : donation.id);

  const donationFactory = (value, lastDonations) => ({
    value,
    date: Date.now(),
    id: lastDonations.reduce(_lastId, 0) + 1,
  });

  export default {
    mixins: [Store],

    props: {
      deadLine: {},
      total: Number,
      default: Number,
      suggestions: {
        type: Array,
        default: () => ([
          15, 25, 50, 150,
        ]),
      },
    },

    data () {
      return {
        donation: this.default,
        show: false,
        needed: 0,
      };
    },

    watch: {
      donation (value, oldValue) {
        if (value < 0) {
          this.donation = 0;
        } else if (/[^0-9]/.test(value)) {
          this.donation = oldValue;
        }
      },

      donations () {
        this.animateRemain();

        const animate = () => (
          Tween.update() ? requestAnimationFrame(animate) : null
        );

        animate();
      },
    },

    computed: {
      remain () {
        const r = this.donations.reduce((remain, donation) => remain - donation.value, this.total);

        return r <= 0 ? 0 : r;
      },

      remainPercent () {
        return 100 - ((this.remain * 100) / this.total);
      },

      barBackground () {
        const r = (2.5 * (100 - this.remainPercent)).toFixed(0);
        const b = (2.5 * this.remainPercent).toFixed(0);

        return `rgb(${r}, ${b}, 0)`;
      },
    },

    created () {
      this.needed = this.remain;
    },

    methods: {
      donate () {
        this.donations.unshift(donationFactory(this.donation, this.donations));
      },

      animateRemain () {
        new Tween.Tween({ tweeningNumber: this.needed })
          .easing(Tween.Easing.Exponential.Out)
          .to({ tweeningNumber: this.remain }, 1000)
          .onUpdate((to) => {
            this.needed = to.tweeningNumber.toFixed(0);
          })
          .start();
      },
    },
  };
</script>

<style lang="scss" scoped>
  $py: 1rem;
  $px: 1rem;
  $radius: .25rem;

  $background: #6585ff;

  $gray: #7d7d7d;
  $white: #FFFFFF;
  $black: #000000;
  $header: #7591ff;
  $daysleft: #e76322;

  #wrapper {
    padding: 0;
    width: 100%;
    max-width: 350px;
    line-height: 1.7;
  }

  strong {
    font-weight: 500;
  }

  input,
  button {
    width: 50%;
    color: $gray;
    cursor: pointer;
    font-weight: 700;
    border-radius: $radius;
    background-color: $white + -5%;
    border: 2px solid $white + -15%;

    &:hover {
      background-color: $white + -10%;
    }
  }

  .header,
  .wrapper {
    padding: $py $px;
    margin-bottom: 1rem;
  }

  .header {
    color: $white;
    position: relative;
    text-align: center;
    border-radius: $radius;
    background-color: $header;
    // border-bottom: 2px solid $white;

    strong {
      color: $white;
    }

    &::after {
      width: 0;
      height: 0;
      content: '';
      right: 1rem;
      bottom: -.6rem;
      position: absolute;
      border-style: solid;
      border-width: .6rem .6rem 0 .6rem;
      border-color: $header transparent transparent transparent;
    }
  }

  .bar {
    background-color: $white;

    div {
      padding: .3rem 0;
      transition: all 1s cubic-bezier(0.19, 1, 0.22, 1);
    //  background-color: #2ecc71;
    }
  }

  .wrapper {
    color: $gray;
    background-color: $white;
    border: 1px solid $white + -15%;

    strong {
      color: $daysleft;
    }

    p {
      margin-bottom: 1rem;
    }

    div {
      display: flex;
      position: relative;
    }

    span,
    input,
    button {
      font-size: 1.4rem;
    }

    input,
    button {
      padding: .5rem $px;
    }

    input {
      padding-left: 2rem;
      margin-right: .5rem;
      padding-right: 2rem;
    }

    button {
      color: $white;
      margin-left: .5rem;
      border-color: #2ecc71;
      background-color: #2ecc71;

      &:hover {
        background-color: #2ecc71 + 10%;
      }
    }

    span {
      left: .7rem;
      top: .125rem;
      position: absolute;
    }
  }

  ul {
    top: 44px;
    padding: 0;
    width: 50%;
    list-style: none;
    position: absolute;
    border-radius: $radius;
    background-color: $white;
    border: 2px solid $white + -15%;
    box-shadow: 3px 3px 6px 1px rgba($black, .2);

    li {
      cursor: pointer;
      padding: 5px 15px;

      &:hover,
      &.active {
        background-color: $white + -10%;
      }
    }

    &::after {
      width: 0;
      height: 0;
      left: .3rem;
      content: '';
      top: -.6rem;
      position: absolute;
      border-style: solid;
      border-width: 0 .6rem .6rem .6rem;
      border-color: transparent transparent $white + -15% transparent;
    }
  }

  .inputs {
    display: flex;

    button {
      font-size: 1rem;
      padding: .5rem $px;

      &:first-of-type {
        margin-right: .5rem;
      }

      &:last-of-type {
        margin-left: .5rem;
      }
    }
  }

  .fade-enter-active, .fade-leave-active {
    transition: all 1s cubic-bezier(0.19, 1, 0.22, 1);
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
    transform: translateY(5px);
  }

  @media screen and (max-width: 540px) {
    #wrapper {
      margin: 0 auto;
    }

    .wrapper {
      span,
      input,
      button {
        font-size: 1rem;
      }

      input {
        padding-top: .5rem;
        padding-bottom: .4rem;
      }

      span {
        top: .3rem;
      }
    }

    .inputs {
      flex-wrap: wrap;

      button {
        width: 100%;
        margin: 0 0 .5rem !important;
      }
    }
  }
</style> 