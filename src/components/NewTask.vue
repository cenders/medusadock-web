<template>
<div class="index">
  <el-steps :active="step" finish-status="success">
    <el-step title="Select Receptor"></el-step>
    <el-step title="Select Ligand"></el-step>
    <el-step title="Set Binding Site"></el-step>
    <el-step title="Submit"></el-step>
  </el-steps>

  <div id="input-area" style="height: 500px">
    <div v-show="step<3">
      <template v-if="step===0">
        <div class="label">Please <el-button type='text'>upload the structure</el-button> or <el-button type='text'>input the PDB ID</el-button> of the receptor.</div>
        <input type="file" ref="receptor_pdb" @change="showReceptor($event)"><i class="prompt">Format: .pdb</i>
      </template>

      <template v-if="step===1">
        <div class="label">Please upload the ligand structure.</div>
        <input type="file" ref="ligand_pdb" @change="showLigand($event)"><i class="prompt">Format: .mol2</i>
      </template>

      <template v-if="step===2">
        <div class="label">Please select the binding site.</div>
        <input type="file" ref="binding_site" @change="showBindingSite($event)"><i class="prompt">Format: .mol2</i>
      </template>

      <div style="position:relative">
        <div id="cover" v-show="showCover">Loading the Structure<i class="el-icon-loading"></i></div>
        <div id="viewport"></div>
      </div>

    </div>

    <div v-show="step===3" style="height: 500px">
      <div class="label" v-if="!user">Email Address</div>
      <el-input v-model="form.email" style="width: 400px"></el-input>

      <div class="label">Number of rounds</div>
      <el-input v-model="form.num" style="width: 150px"></el-input>

      <div class="label">Constraints</div>
      <el-input type="textarea" :rows="5" v-model="form.constraints"></el-input>

      <div class="label">Cutoff</div>
      <el-input v-model="form.cutoff" style="width: 150px"></el-input>
    </div>
  </div>

  <el-button v-if="step>0" style="float: left; margin-top: 12px;" @click="prev" icon="el-icon-arrow-left">Prev</el-button>
  <el-button v-if="(step==0&&ngl.receptor)||(step==1&&ngl.ligand)||(step==2&&ngl.bindingSite)" type="primary" style="float: right; margin-top: 12px;" @click="next">Next<i class="el-icon-arrow-right el-icon--right"></i></el-button>
  <el-button v-if="step>=3" type="success" style="float: right; margin-top: 12px;" @click="submit">Submit<i class="el-icon-arrow-right el-icon--right"></i></el-button>
  <div style="clear: both"></div>

  <el-dialog :visible.sync="show_dialog" width="30%">
    <span>Submitted successfully!</span>
    <span slot="footer" class="dialog-footer">
    <el-button @click="show_dialog = false">New Task</el-button>
    <el-button type="primary" @click="show_dialog = false">Task Status</el-button>
    </span>
  </el-dialog>
</div>

</template>

<script>
import axios from 'axios'
import { bus } from '../bus.js'
import * as NGL from 'ngl'

axios.defaults.headers.post['Content-Type'] = 'multipart/form-data'

export default {
  name: 'Index',
  data () {
    return {
      showCover: false,
      step: 0,
      receptorLabel: 'Please upload the receptor structure.',
      ligandLabel: 'Please upload the ligand structure.',
      bindingSiteLabel: 'Please select the binding site.',
      ngl: {
        stage: '',
        receptor: '',
        ligand: '',
        bindingSite: ''
      },
      show_dialog: false,
      show_options: false,
      potential_options: [{
        value: 'spring',
        label: 'Spring Potential'
      }, {
        value: 'squarewell',
        label: 'Square Well Potential'
      }],
      form: {
        num: 1,
        email: '',
        constraints: '',
        cutoff: 10
      }
    }
  },
  computed: {
    user () {
      return this.$store.state.user
    }
  },
  methods: {
    next () {
      var v = this
      v.step += 1
    },
    prev () {
      var v = this
      v.step -= 1
    },
    showReceptor (e) {
      let v = this
      let f = e.target.files[0]

      v.showCover = true
      console.log('Loading receptor...')
      v.ngl.stage.getComponentsByName(v.ngl.receptor).forEach(function (c) {
        v.ngl.stage.removeComponent(c)
      })
      v.ngl.receptor = f.name

      v.ngl.stage.loadFile(f, { ext: 'pdb' }).then(function (comp) {
        comp.addRepresentation('surface', { multipleBond: true })
        comp.autoView()
        v.showCover = false
//        v.ngl.stage.autoView()
//        v.ngl.stage.signals.clicked.add(function (pickingProxy) {
//          console.log(pickingProxy ? pickingProxy.getLabel() : 'nothing')
//        })
        v.ngl.stage.mouseControls.add('drag-left', function (stage, x, y) {
//          var o = v.ngl.stage.mouseObserver
//          var p = v.ngl.stage.pickingControls.pick(o.down.x, o.down.y).component
//          var center = p.getCenter()
//          p.setPosition([center.x + x / 10.0, center.y + y / 10.0, center.z + 0])
          console.log(v.ngl.stage.viewerControls.position)
        })
      })
    },

    showLigand (e) {
      let v = this
      let f = e.target.files[0]

      console.log('Loading ligand...')
      v.showCover = true
      v.ngl.stage.getComponentsByName(v.ngl.ligand).forEach(function (c) {
        v.ngl.stage.removeComponent(c)
      })
      v.ngl.ligand = f.name

      v.ngl.stage.loadFile(f, { ext: 'mol2' }).then(function (comp) {
        comp.addRepresentation('ball+stick', { multipleBond: true })
        v.showCover = false
      })
    },

    showBindingSite (e) {
      let v = this
      let f = e.target.files[0]

      console.log('Loading ligand...')
      v.showCover = true
      v.ngl.stage.getComponentsByName(v.ngl.bindingSite).forEach(function (c) {
        v.ngl.stage.removeComponent(c)
      })
      v.ngl.bindingSite = f.name

      v.ngl.stage.loadFile(f, { ext: 'mol2' }).then(function (comp) {
        var shape = new NGL.Shape('shape')
        var center = comp.getCenter()
        shape.addSphere([center.x, center.y, center.z], [1, 0, 0], 1)
//        var sphereBuffer = new NGL.SphereBuffer({
//          position: comp.getCenter(),
//          color: new Float32Array([1, 0, 0]),
//          radius: new Float32Array([1])
//        })
//        shape.addBuffer(sphereBuffer)
        var shapeComp = v.ngl.stage.addComponentFromObject(shape)
        shapeComp.addRepresentation('buffer')

//        comp.addRepresentation('ball+stick', { multipleBond: true })
        v.showCover = false
      })
    },

    submit () {
      let v = this
      let formData = new FormData()
      for (let i in v.form) {
        formData.append(i, v.form[i])
      }
      if (v.user) {
        formData.append('email', v.user.email)
      }
      formData.append('receptor_pdb', this.$refs.receptor_pdb.files[0])
      formData.append('ligand_pdb', this.$refs.ligand_pdb.files[0])
      formData.append('binding_site', this.$refs.binding_site.files[0])

      axios({
        method: 'post',
        url: 'http://redshift.med.unc.edu/medusadock/actions/submit.php',
        data: formData,
        config: {headers: {'Content-Type': 'multipart/form-data'}}
      }).then(response => {
        alert('Submitted successfully!')
        window.location.href = '#/' + 'TaskManagement'
      }).catch(() => {
        alert('Submitted failed!')
      })
    }
  },
  mounted () {
    let v = this

    v.$nextTick(function () {
      bus.$emit('switch-router', 'NewTask')

      v.ngl.stage = new NGL.Stage('viewport', { backgroundColor: 'white' })
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

i.prompt {
  font-size: 12px;
}

.el-form-item {
  margin-bottom: 3px;
}

#input-area {
  margin-top: 10px;
}

.label {
  font-size: 14px;
  height: 40px;
  line-height: 40px;
  vertical-align: middle;
}

#viewport {
  width: 600px;
  height: 400px;
  position: absolute;
  left: 0;
  top: 0;
}

#cover {
  font-size: 30px;
  width: 600px;
  height: 400px;
  line-height: 400px;
  vertical-align: middle;
  text-align: center;
  color: rgb(180, 188, 204);
  position: absolute;
  left: 0;
  top: 0;
  z-index: 99;
}

</style>