<template>
  <main>
    <section id="section1">section 1</section>
    <section><div v-on:mousemove="mouseMove" id="app" ref="canva"></div></section>
    <section>section 3</section>
    <section>section 4</section>
  </main>
</template>

<script>
import 'pathseg'
import Matter from 'matter-js'
import Ola from 'ola'

// const paths = [
//   'M289.05,84.63L98.225,308.136c-6.35,7.44-9.126,17.291-7.594,26.954l10.514,66.144c1.32,8.312,8.484,14.441,16.901,14.44h66.971c9.785,0,19.079-4.283,25.434-11.725l206.015-241.29c5.93-6.943,5.104-17.379-1.841-23.305l-42.83-36.57c-6.946-5.928-17.38-5.104-23.31,1.837l-152.4,178.477c-3.312,3.872-1.941,7.557-0.754,15.562c0.647,4.358,4.409,7.572,8.817,7.528c8.201-0.082,11.959,0.677,15.254-3.186l104.745-122.653l30.031,25.641l-108.396,126.93c-5.823,6.821-14.307,10.795-23.272,10.904h-40.639c-10.349,0.124-19.191-6.943-20.691-17.18l-5.896-40.215c-1.304-8.875,1.292-17.878,7.118-24.697L321.757,75.111c18.261-21.39,50.409-23.929,71.803-5.664l50.576,43.18c21.395,18.265,23.932,50.414,5.666,71.806L235.457,435.479c-10.447,12.243-25.735,19.284-41.83,19.271h-89.321c-19.688-0.012-36.449-14.382-39.549-33.821l-14.072-88.213c-2.53-15.89,2.027-32.097,12.479-44.339L259.017,58.988L289.05,84.63z'
// ]

export default {
  name: 'App',
  components: {},
  data() {
    return {
      gravity: null,
      movingShape1: null,
      movingShape2: null,
      mouse: null,
      cursor: null,
      accelerometer: 'i',
      Engine: Matter.Engine,
      render: null,
      Render: Matter.Render,
      Runner: Matter.Runner,
      Composites: Matter.Composites,
      Common: Matter.Common,
      MouseConstraint: Matter.MouseConstraint,
      Mouse: Matter.Mouse,
      Composite: Matter.Composite,
      Bodies: Matter.Bodies,
      Body: Matter.Body
    }
  },

  mounted() {
    this.init()
    // fit the render viewport to the scene
    // Render.lookAt(this.render, {
    //   min: { x: 0, y: 0 },
    //   max: { x: window.innerWidth, y: window.innerHeight }
    // })

    // window.addEventListener('deviceorientation', this.updateGravity())
    // window.addEventListener('devicemotion', this.updateGravity())
    // window.addEventListener('MozOrientation', this.updateGravity())
  },
  created() {
    // add gyro control
    if (typeof window !== 'undefined') {
      window.addEventListener('resize', this.resizeWindow)
      window.addEventListener('deviceorientation', this.updateGravity)
      console.log('add lsitener gravity')
    } else {
      console.log('nogravity: ')
    }
  },
  methods: {
    init() {
      this.xCursor = Ola({ x: 0 }, 1000)
      var Engine = Matter.Engine,
        Render = Matter.Render,
        Runner = Matter.Runner,
        MouseConstraint = Matter.MouseConstraint,
        Mouse = Matter.Mouse,
        World = Matter.World,
        Bodies = Matter.Bodies

      // create engine
      var engine = Engine.create(),
        world = engine.world

      // create renderer
      this.render = Render.create({
        element: this.$refs.canva,
        engine: engine,
        options: {
          background: '#f3ff00',
          width: window.innerWidth,
          height: window.innerHeight,
          pixelRatio: 'auto',
          wireframes: false,
          showDebug: false,
          showBroadphase: false,
          showBounds: false,
          showVelocity: false,
          showCollisions: false,
          showSeparations: false,
          showAxes: false,
          showPositions: false,
          showAngleIndicator: false,
          showIds: false,
          showShadows: false,
          showVertexNumbers: false,
          showConvexHulls: false,
          showInternalEdges: false,
          showMousePosition: false
        }
      })

      Render.run(this.render)

      // create runner
      var runner = Runner.create()
      Runner.run(runner, engine)

      // add mouse control
      this.mouse = Mouse.create(this.render.canvas)

      this.mouseConstraint = MouseConstraint.create(engine, {
        mouse: this.mouse,
        constraint: {
          stiffness: 0.2,
          render: {
            visible: false
          }
        }
      })

      // allow scroll on canva
      this.mouseConstraint.mouse.element.removeEventListener('mousewheel', this.mouseConstraint.mouse.mousewheel)
      this.mouseConstraint.mouse.element.removeEventListener('DOMMouseScroll', this.mouseConstraint.mouse.mousewheel)

      World.add(world, this.mouseConstraint)

      // keep the mouse in sync with rendering
      this.render.mouse = this.mouse

      const circle1 = Bodies.circle(window.innerWidth - 300, window.innerHeight - 300, 80, {
        friction: 1,
        restitution: 1.2,
        isStatic: false,
        render: { strokeStyle: 'black', lineWidth: 3, fillStyle: 'transparent' }
      })

      const circle2 = Bodies.circle(window.innerWidth - 200, window.innerHeight - 500, 80, {
        friction: 0.2,
        restitution: 1,
        isStatic: false,
        render: { strokeStyle: 'black', lineWidth: 3, fillStyle: 'black' }
      })

      this.movingShape1 = Bodies.rectangle(-window.innerWidth, window.innerHeight - 50, window.innerWidth, window.innerHeight * 0.25, {
        friction: 1,
        restitution: 1,
        isStatic: true,
        render: { fillStyle: 'black' }
      })

      this.movingShape2 = Bodies.rectangle(window.innerWidth * 2, window.innerHeight - 50, window.innerWidth, window.innerHeight * 0.25, {
        friction: 1,
        restitution: 1,
        isStatic: true,
        render: { fillStyle: 'black' }
      })

      const leftWall = Bodies.rectangle(-25, 300, 50, 800, {
        isStatic: true,
        friction: 0
      })

      // const rightWall = Bodies.rectangle(window.innerWidth, 300, 50, 800, {
      //   isStatic: true,
      //   friction: 0
      // })

      World.add(world, [circle1, circle2, this.movingShape1, this.movingShape2])
      World.add(world, [Bodies.rectangle(300, window.innerHeight, window.innerWidth * 2, 20, { isStatic: true }), leftWall])
    },
    resizeWindow() {
      this.init()
      // this.render.bounds.max.x = this.$refs.canva.getBoundingClientRect().width
      // this.render.bounds.max.y = this.$refs.canva.getBoundingClientRect().height
      // this.render.options.width = this.$refs.canva.getBoundingClientRect().width
      // this.render.options.height = this.$refs.canva.getBoundingClientRect().height
      // this.render.canvas.width = window.innerWidth
      // console.log(window.innerWidth, this.render.canvas.width, this.$refs.canva.getBoundingClientRect().width)
    },
    mouseMove: function (event) {
      // this.xCursor = event.clientX
      this.xCursor.set({ x: event.clientX })
      setInterval(() => {
        this.Body.setPosition(this.movingShape2, { x: this.xCursor.x + window.innerWidth * 0.65, y: this.movingShape2.position.y })
        this.Body.setPosition(this.movingShape1, { x: this.xCursor.x - window.innerWidth / 2, y: this.movingShape1.position.y })
      }, 10)
      // this.Body.setPosition(this.movingShape1, { x: this.xCursor - window.innerWidth / 2, y: this.movingShape1.position.y })
    },
    updateGravity(event) {
      var orientation = typeof window.orientation !== 'undefined' ? window.orientation : 0
      this.gravity = this.engine.gravity

      if (orientation === 0) {
        this.gravity.x = this.Common.clamp(event.gamma, -90, 90) / 90
        this.gravity.y = this.Common.clamp(event.beta, -90, 90) / 90
      } else if (orientation === 180) {
        this.gravity.x = this.Common.clamp(event.gamma, -90, 90) / 90
        this.gravity.y = this.Common.clamp(-event.beta, -90, 90) / 90
      } else if (orientation === 90) {
        this.gravity.x = this.Common.clamp(event.beta, -90, 90) / 90
        this.gravity.y = this.Common.clamp(-event.gamma, -90, 90) / 90
      } else if (orientation === -90) {
        this.gravity.x = this.Common.clamp(-event.beta, -90, 90) / 90
        this.gravity.y = this.Common.clamp(event.gamma, -90, 90) / 90
      }
    },
    listen() {
      this.accelerometer = window.screen.orientation.type
      // var x = event.accelerationIncludingGravity.x
      // var y = event.accelerationIncludingGravity.y
      // var z = event.accelerationIncludingGravity.z
      // this.accelerometer = [x, y, z]
      // Do something awesome.
    }
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background: #f3ff00;
  width: 100vw;
  height: 100vh;
}

#section1 {
  background: #f3ff00;
}
</style>
