<template>
  <div :class="{grab:hold}" @mousedown="mouseDown($event)" id="app">

    <Gui v-if="stage" :stage="stage" :strokes="strokes"/>

    <Alert v-if="alert"/>

    <svg
      class="render"
      v-if="ball"
      ref="render"
      xmlns="http://www.w3.org/2000/svg"
      version="1.1"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      :width="window.width"
      :height="window.height"
      :view-box.camel="follow ? (-(window.width / 2) + ball.position.x)+' '+(-(window.height / 2) + ball.position.y)+' '+window.width+' '+window.height : (-(window.width / 2) + ball.initial.x)+' '+(-(window.height / 2) + ball.initial.y)+' '+window.width+' '+window.height "
    >
    <defs>


    <!-- Ball Gradient -->
    <radialGradient id="ball"> 
      <stop offset="10%" stop-color="#FFFFFF"/> 
      <stop offset="95%" stop-color="#CCCCCC"/> 
    </radialGradient>

    </defs>

    <pattern id="fairway" patternUnits="userSpaceOnUse" width="20" height="20" patternTransform="scale(4,4) rotate(45)">
      <rect width="20" height="20" fill="#AABD51" />
      <rect width="10" height="10" fill="#B5C956" />
      <rect width="10" height="10" x="10" y="10" fill="#9FB14B" />
    </pattern>

    <radialGradient id="sand"> 
      <stop offset="5%" stop-color="#B5AC6B"/> 
      <stop offset="95%" stop-color="#EFE68F"/> 
    </radialGradient>

    <g id="stars">
      <circle v-for="star in stars" :cx="star.x" :cy="star.y" :r="star.r" :fill="'rgba(255,255,255,'+star.opacity/10+')'"></circle>
    </g>

      <!-- Course Fill -->
      <polygon
        v-if="poly"
        :points="poly"
        fill="none"
        stroke="#bfbfbf"
        stroke-width="14"
        stroke-linejoin="round"
        stroke-linecap="round"

      ></polygon>
      <polygon
        v-if="poly"
        :points="poly"
        fill="url(#fairway)"
        stroke="#EE6025"
        stroke-width="8"
        stroke-linejoin="round"
        stroke-linecap="round"
      ></polygon>

      <!-- Start -->
      <rect class="object_start" fill="#ABC256" stroke="#9BB441" rx="10" ry="10" stroke-width="4" :x="stage.start.x" :y="stage.start.y" :width="stage.start.width" :height="stage.start.height"></rect>

      <!-- Green -->
      <rect class="object_green" fill="#ABC256" stroke="#9BB441" rx="10" ry="10" stroke-width="4" :x="stage.green.x" :y="stage.green.y" :width="stage.green.width" :height="stage.green.height"></rect>

      <!-- Obstacles -->
      <g v-if="stage" v-for="(o,index) in stage.obstacles">
        <circle v-if="o.type == 'rough'" class="obstacle" fill="url(#sand)" stroke="#BFB464" stroke-width="4" :cx="o.x" :cy="o.y" :r="o.radius"></circle>
      </g>

      <!-- Hole -->
      <circle  class="object_hole" :cx="stage.hole.x" :cy="stage.hole.y" :r="10" fill="black" stroke-width="4" stroke="#9BB441"></circle>

      <!-- Ball -->
      <circle ref="ball" class="object_ball" :cx="ball.position.x" :cy="ball.position.y" :r="ball.circleRadius" fill="url(#ball)" stroke="#333333" stroke-width="1"></circle>

      <!-- Flag -->
      <g :transform="'translate('+stage.hole.x+' '+stage.hole.y+')'" id="flag">
        <rect x="-1.5" y="-80" width="4" height="83" fill="#EEEEEE"></rect>
        <polygon transform="translate(2.5,-80)" points="0,0 50,16 0,32" fill="#EE6025"></polygon>
      </g>

      <!-- Constraint -->
      <line
        v-if="!follow"
        :x1="ball.initial.x"
        :y1="ball.initial.y"
        :x2="ball.position.x"
        :y2="ball.position.y"
        stroke="black"
        stroke-width="2"
        stroke-linecap="round"
      ></line>

      <!--<line
      :x1="ball.initial.x"
      :y1="ball.initial.y"
      :x2="ball.position.x"
      :y2="ball.position.y"
      stroke="rgba(255,255,255,0.4)"
      stroke-width="2"
      stroke-linecap="round"
      ></line>-->

    </svg>
    <Course ref="course" style="display:none;"></Course>
  </div>
</template>

<script>
import * as Matter from 'matter-js'
import Line from "matter-lines";
import Course from './assets/hole12.svg'; // Course
import Gui from './components/Gui.vue'
import Alert from './components/Alert.vue'
import Pizzicato from 'pizzicato'


const sounds = {
  putt: new Pizzicato.Sound('/sound/putt.mp3'),
  hole: new Pizzicato.Sound('/sound/hole.mp3'),
  bounce: [new Pizzicato.Sound('/sound/bounceA.mp3'),new Pizzicato.Sound('/sound/bounceB.mp3'),new Pizzicato.Sound('/sound/bounceC.mp3')],
  clap: new Pizzicato.Sound('/sound/clap.mp3')
}

//Matter.use('matter-collision-events')

let Engine          = Matter.Engine,
    World           = Matter.World,
    Body            = Matter.Body,
    Bodies          = Matter.Bodies,
    Events          = Matter.Events,
    Constraint      = Matter.Constraint,
    Composite       = Matter.Composite,
    Composites      = Matter.Composites,
    Bounds          = Matter.Bounds,
    Mouse           = Matter.Mouse,
    MouseConstraint = Matter.MouseConstraint,
    Render          = Matter.Render,
    Vector          = Matter.Vector,
    Vertices        = Matter.Vertices,
    Common          = Matter.Common,
    Runner          = Matter.Runner,
    Pairs           = Matter.Pairs;

let engine, ball, holl, constraint, canvas, ctx, mouseConstraint;

export default {
  data () {
    return {
      window: {
        width: window.innerWidth,
        height: window.innerHeight
      },
      alert: false,
      hold: false,
      follow: false,
      speedCheck: false,
      ball: null,
      lines: [],
      poly: null,
      constraint: null,
      stars: [],
      starCount: 120,
      stage: {
        id: 1,
        title: 'Practice Course',
        width: 1600,
        height: 1600,
        spawn: {
          x: 0,
          y: 0
        },
        hole: null,
        start: null,
        green: null,
        par: 3,
        complete: false,
        obstacles: []
      },
      strokes: 0
    }
  },
  mounted () {
    this.init()

    window.addEventListener('resize',((event) => {
      this.window.width = window.innerWidth;
      this.window.height = window.innerHeight;
    }))
  },
  methods: {
    mouseDown(event) {
      if (this.follow) {
        event.preventDefault()
        event.stopPropagation()
        event.stopImmediatePropagation()
      }
    },
    init () {
      engine = Engine.create()
      let world = engine.world;

      engine.world.gravity.x = 0;
      engine.world.gravity.y = 0;
      
      this.makeStars()
      this.parseCourse()
      this.makePoints()
      this.addBall()
      // add mouse constraint
      let mouseConstraint = MouseConstraint.create(engine, {
              mouse: Mouse.create(this.$el),
              constraint: {
              stiffness: 0.2,
              angularStiffness: 0.2,
                  render: {
                      visible: false
                  }
              }
      });

      World.add(world, mouseConstraint);

      Events.on(mouseConstraint, "enddrag", (e) => {
        if (!e.body || !e.body.ball) return;
        let dx = e.body.initial.x - e.body.position.x;
        let dy = e.body.initial.y - e.body.position.y;
        
        if (dx > 6 || dx < -6 || dy > 6 || dy < -6) {
          this.$nextTick(() => {
              sounds.putt.play()
              this.constraint.bodyB = this.constraint.bodyA;
              this.follow = true;
              this.strokes++;
              setTimeout(() => {
                this.speedCheck = true;
              }, 1000);
          })
        }
      })

      Events.on(mouseConstraint, "mousedown", (e) => {
        this.hold = true;
      })
      Events.on(mouseConstraint, "mouseup", (e) => {
        this.hold = false;
      })


      Events.on(engine, 'beforeUpdate', ((event) => {
        Mouse.setOffset(mouseConstraint.mouse, {x: -(this.window.width / 2) + this.ball.position.x, y: -(this.window.height / 2) + this.ball.position.y});
        if (this.follow){
          if (this.speedCheck && this.ball.speed < 0.05) {
            Body.setVelocity(ball, {x: 0, y:0})
            this.follow = false;
            this.speedCheck = false;
            this.ball.initial.x = this.ball.position.x;
            this.ball.initial.y = this.ball.position.y;
            this.constraint.pointA = {x: this.ball.initial.x, y: this.ball.initial.y}
            this.constraint.bodyB = this.ball;
            console.log('ball stopped')
          }
        }
        if (this.constraint.bodyB == this.ball) {
          this.ball.isSensor = true;
        } else {
          this.ball.isSensor = false;
        }

        if (this.stage.complete) {
          if (this.ball.circleRadius > 1) {
            this.ball.circleRadius -= 0.2;
          } else {
            this.$refs.ball.classList.add('hidden')
          }
        }

        if (this.ball.velocity.x > 9) {
          Body.setVelocity(this.ball, { x: 9, y: this.ball.velocity.y })
        }
        if (this.ball.velocity.x < -9) {
          Body.setVelocity(this.ball, { x: -9, y: this.ball.velocity.y })
        }

        if (this.ball.velocity.y > 9) {
          Body.setVelocity(this.ball, { x: this.ball.velocity.x, y: 9 })
        }
        if (this.ball.velocity.y < -9) {
          Body.setVelocity(this.ball, { x: this.ball.velocity.x, y: -9 })
        }
      }))

      // Collisions 
      Events.on(engine, 'collisionStart', ((event) => {
          var pairs = event.pairs;
          for (var i = 0; i < pairs.length; i++) {
            var pair = pairs[i];

            if (pair.bodyB.ball || pair.bodyA.ball) { // ball

              this.ball.isSensor = false;
              if (this.hold) {
                this.ball.isSensor = true;
              }

              if (pair.bodyA.rough) {
                return;
              }

              if (this.ball.speed > 1 && !this.hold) {
                sounds.bounce[Math.floor(Math.random() * 2) + 0 ].play() 
              }
              
              this.$nextTick(() => {
                pair.isActive = false
              })
            
              if (pair.bodyA.hole) { // ball > hole
                sounds.hole.play()
                setTimeout(() => {
                  sounds.clap.play()
                }, 1000);
                this.completeHole()
              }
            }
          }
      }))

      Events.on(engine, 'collisionActive', ((event) => {
          var pairs = event.pairs;
          for (var i = 0; i < pairs.length; i++) {
            var pair = pairs[i];
            if (pair.bodyA.rough) {
              this.ball.frictionAir = 0.1;
            }
          }
      }))

      Events.on(engine, 'collisionEnd', ((event) => {
          var pairs = event.pairs;
          for (var i = 0; i < pairs.length; i++) {
            var pair = pairs[i];
            if (pair.bodyA.rough) {
              this.ball.frictionAir = 0.01;
            }
          }
      }))

      Engine.run(engine);

    },
    addBall () {
      ball = Bodies.circle(this.stage.spawn.x, this.stage.spawn.y, 7, {
        density: 0.01,
        frictionAir: 0.01,
        restitution: 0.9,
        render: { fillStyle: 'white' },
        ball: true
      })
      ball.initial = {x: this.stage.spawn.x, y: this.stage.spawn.y}
      constraint = Constraint.create({ 
              pointA: { x: ball.position.x, y: ball.position.y }, 
              bodyB: ball,
              damping: 0.5,
              render: {
                type: 'line'
              }
          });

      World.add(engine.world, [ball, constraint]);
      this.constraint = constraint; 
      this.ball = ball; 
    },
    makePoints () {
      let poly = ''
      this.lines.forEach((line,i) => {
        poly += this.lines[i].x + ',' + this.lines[i].y + ' ';
      })
      this.poly = poly;
    },
    parseCourse () {
      let course = this.$refs.course;
      // Make lines
      let courseArr = [];
      course.querySelectorAll('polygon').forEach((p) => {
        if (p.id == 'course') {
          let points = p.getAttribute('points').split(' ')
          points.forEach((pt) => {
            let point = pt.split(',')
            courseArr.push({x:parseFloat(point[0]),y:parseFloat(point[1])})
            this.lines.push({x:parseFloat(point[0]),y:parseFloat(point[1])})
          })
          let lineA = new Line(engine.world,courseArr,6,5)
          let lineB = new Line(engine.world,[{x: courseArr[courseArr.length-1].x, y: courseArr[courseArr.length-1].y },{x: courseArr[0].x, y: courseArr[0].y}],6,5)
          lineA.body.wall = true;
          lineB.body.wall = true;
        }
      })
      // Make spawn
      course.querySelectorAll('rect').forEach((r) => {
        if (r.id == 'spawn') {
          this.stage.spawn.x = parseFloat(r.getAttribute('x'))
          this.stage.spawn.y = parseFloat(r.getAttribute('y'))
        } else if (r.id == 'green') {
          this.stage.green = {};
          this.stage.green.x = parseFloat(r.getAttribute('x'))
          this.stage.green.y = parseFloat(r.getAttribute('y'))
          this.stage.green.width = parseFloat(r.getAttribute('width'))
          this.stage.green.height = parseFloat(r.getAttribute('height'))
        } else if (r.id == 'start') {
          this.stage.start = {};
          this.stage.start.x = parseFloat(r.getAttribute('x'))
          this.stage.start.y = parseFloat(r.getAttribute('y'))
          this.stage.start.width = parseFloat(r.getAttribute('width'))
          this.stage.start.height = parseFloat(r.getAttribute('height'))
        } else if (r.id == 'hole') {
          this.stage.hole = {};
          this.stage.hole.x = parseFloat(r.getAttribute('x'))
          this.stage.hole.y = parseFloat(r.getAttribute('y'))
          let hole = Bodies.circle(this.stage.hole.x, this.stage.hole.y, 2, {
            isStatic: true,
            hole: true
          })
          World.add(engine.world, [hole]);

        } else if (r.id == 'rough') {
          let obstacle = {
            type: 'rough',
            x: parseFloat(r.getAttribute('x')),
            y: parseFloat(r.getAttribute('y')),
            width: parseFloat(r.getAttribute('width')),
            height: parseFloat(r.getAttribute('height'))
          }
          obstacle.x = obstacle.x + obstacle.width/2;
          obstacle.y = obstacle.y + obstacle.height/2;
          obstacle.radius = obstacle.width/2;
          this.stage.obstacles.push(obstacle)
          let oBody = Bodies.circle(obstacle.x, obstacle.y, obstacle.width/2, {
            isStatic: true,
            isSensor: true,
            rough: true
          })
          World.add(engine.world, [oBody]);
        }
      })
    },
    makeStars () {
      for (let i = 0; i < this.starCount; i++) {
        let posX = Math.floor(Math.random() * this.stage.width * 1.5) + 1;
        let posY = Math.floor(Math.random() * this.stage.height * 1.5) + 1;
        let radius = Math.floor(Math.random() * 3) + 1;
        let op = Math.floor(Math.random() * 9) + 2;
        this.stars.push({x: posX, y: posY, r: radius, opacity: op})
      }
    },
    completeHole () {
      Body.setStatic(this.ball,true)
      this.stage.complete = true;
      setTimeout(() => {
        this.alert = true;
      }, 3000);
    }
  },
  components: {
    Course,
    Gui,
    Alert
  }
}
</script>