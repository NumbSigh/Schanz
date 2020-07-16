<template>
  <div id="space">
    <div id="cont"></div>
  </div>
</template>

<script>

import * as THREE from 'three'

import { MapControls } from 'three/examples/jsm/controls/OrbitControls'

import * as GEOLIB from 'geolib'

export default {
  name: 'space',
  data(){
    return{
      //THREE space
      scene: null,
      camera: null,
      renderer: null,
      controls: null,

      center: [119.32746, 26.07275],

      MAT_BUILDING: null
    }
  },

  mounted(){
    this.Awake()

    let that = this

    //when user resize window 111
    window.addEventListener( 'resize', onWindowResize, false )

    function onWindowResize(){
      that.camera.aspect = window.innerWidth / window.innerHeight
      that.camera.UpdateProjectionMatrix()
      that.renderer.setSize( window.innerWidth, window.innerHeight )
    }

    onWindowResize()
  },

  methods:{
    Awake(){
      let cont = document.getElementById("cont")

      //Init scene
      this.scene = new THREE.Scene()
      window.scene = this.scene
      this.scene.background = new THREE.Color(0x222222)

      //Init camera
      this.camera = new THREE.PerspectiveCamera(25, window.innerWidth/window.innerHeight, 1, 100)
      this.camera.position.set(8, 4, 0)

      //Init Light
      let light0 = new THREE.AmbientLight(0xfafafa, 0.25)

      let light1 = new THREE.PointLight(0xfafafa, 0.4)
      light1.position.set(200, 90, 40)

      let light2 = new THREE.PointLight(0xfafafa, 0.4)
      light2.position.set(200, 90, -40)

      this.scene.add( light0 )
      this.scene.add( light1 )
      this.scene.add( light2 )

      let gh = new THREE.GridHelper( 60, 160, new THREE.Color(0x555555), new THREE.Color(0x333333) )
      this.scene.add(gh)

      let geometry = new THREE.BoxGeometry( 1, 1, 1 )
      let material = new THREE.MeshBasicMaterial( {color: 0x00ff00} )
      let cube = new THREE.Mesh( geometry, material )
      this.scene.add( cube )


      //Init Renderer
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(window.innerWidth, window.innerHeight)

      cont.appendChild(this.renderer.domElement)

      //Init Controls
      this.controls = new MapControls(this.camera, this.renderer.domElement)
      this.controls.enableDamping = true
      this.controls.dampingFactor =0.25
      this.controls.screenSpacePanning = false
      this.controls.maxDistance = 800

      this.controls.update()

      this.Update()

      this.MAT_BUILDING = new THREE.MeshPhongMaterial()
      this.GetgeoJson()

    },

    Update(){
      requestAnimationFrame(this.Update)

      this.renderer.render(this.scene, this.camera)
      this.controls.update()
    },

    GetgeoJson(){
      fetch("./assets/fuzhou.geojson").then((res)=>{
        res.json().then((data)=>{
          this.LoadBuilding(data)
        })  
      })
    },

    LoadBuilding(data){
      let features = data.features

      for(let i=0;i<features.length;i++){
        let fel = features[i]

        if(!fel['properties']) return

        if(fel.properties['building']){
          this.addBuilding(fel.geometry.coordinates, fel.properties, fel.properties['building:levels'])
        }
      }
    },

    addBuilding(data, info, height=1){
      
      for(let i=0;i<data.length;i++){
        let el = data[i]

        let shape = this.genShape(el, this.center)
        let geometry = this.genGeometry(shape, {
          curveSegment: 1,
          depth: 0.5 * height,
          bevelEnabled: false,
        })

        geometry.rotateX(Math.PI / 2)
        geometry.rotateZ(Math.PI)

        let mesh = new THREE.Mesh(geometry, this.MAT_BUILDING)

        this.scene.add(mesh)
      }
    },

    genShape(points, center){
      let shape = new THREE.Shape

      for(let i=0;i<points.length;i++){
        let elp = points[i]

        elp = this.GPSRelativePosition(elp, center)

        if(i == 0){
          shape.moveTo(elp[0], elp[1])
        } else {
          shape.lineTo(elp[0], elp[1])
        }
      }

      return shape
    },

    genGeometry(shape, config){
      let geometry = new THREE.ExtrudeBufferGeometry(shape, config)
      geometry.computeBoundingBox()

      return geometry
    },
    GPSRelativePosition(objPosi, centerPosi){

      //Get GPS distance
      let dis = GEOLIB.getDistance(objPosi, centerPosi)

      //Get bearing angle
      let bearing = GEOLIB.getRhumbLineBearing(objPosi, centerPosi)

      //Calculate X by centerPosi.x + distance * cos(rad)
      let x = centerPosi[0] + (dis * Math.cos(bearing * Math.PI / 180))

      //Calculate Y by centerPosi.y + distance * sin(rad)
      let y = centerPosi[1] + (dis * Math.sin(bearing * Math.PI / 180))

      //Reverse x (it work)
      return [-x/100, y/100]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
