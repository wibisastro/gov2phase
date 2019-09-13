<template>
    <div v-if="isActive">
        <div id="myWizard" class="wizard">
            <div class="wizard-inner">
                <ul class="steps">
                <template v-for="i in stages">
                    <li v-if="i.id == active" class="active" v-bind:key="i.id">
                        <span class="badge badge-primary">{{ i.id }}</span>{{ i.label }}<span class="chevron"></span>
                    </li>
                    <li 
                        v-else-if="i.id < active" 
                        class="complete" 
                        v-bind:key="i.id" 
                        :title="i.label"
                        :class="showCollapse ? 'collapsed' : null"
                        :aria-expanded="showCollapse ? 'true' : 'false'"
                        aria-controls="collapse-4"
                        @click="mCollapse(i.label)">
                        <span class="badge badge-success">{{ i.id }}</span>{{ i.label }}<span class="chevron"></span>
                    </li>
                    <li v-else v-bind:key="i.id" :title="i.label">
                        <span class="badge">{{ i.id }}</span>{{ i.label }}
                        <span v-if="i.id == 4">{{ state['triwulan'] }}</span>
                    <span class="chevron"></span>
                    </li>
                </template>
                </ul>
                <div class="actions">
                    <button type="button" class="btn btn-secondary btn-mini btn-prev" v-if="active != 1" @click="submitPrev">
                        <i class="icon-arrow-left"></i>{{ prev }}
                    </button>
                    <button v-if="active != 4" type="button" class="btn btn-success btn-mini btn-next" data-last="Finish" @click="submitNext">
                        {{ next }}<i class="icon-arrow-right"></i>
                    </button>
                </div>
            </div>
        </div>
        
        
    </div>
</template>

<script>
    module.exports = {
        name: "gov2phasebar",
        props: ['isActive','getUrl','stateNo'],
        data: function () {
            return {
                active: 0,
                stages: [],
                text: `Lorem Ipsum is simply dummy text of the printing and typesetting industry.`,
                urlPrev: '',
                urlNext: '',
                state: {},
                showCollapse: false,
                myCollapse: ''
            }
        },
        methods: {
            initialize: function() {
                 axios.get(`${this.getUrl}/phases.json`)
                    .then(response => this.loadStages(response.data))
                 .catch(error => this.openNotif(error.response.data))
            },
            getState: function() {
                 axios.get(`${this.getUrl}/state.json`)
                    .then(response => this.loadState(response.data))
                 .catch(error => this.openNotif(error.response.data))
            },
            loadStages: function (data) {
                if (data) {
                    this.stages = Array.from(Object.keys(data), k => data[k]);
                }
                this.active = this.stateNo
                this.getState();
            },
            loadState: function (data) {
                if (data) {
                    this.state=data[0];
                    this.active = data[0]['id'];
                }
                
            },
            submitPrev: function() {
                eventBus.$emit('setCurrentState', this.active--);
            },
            submitNext: function() {
                eventBus.$emit('setCurrentState', this.active++);
            },
            openNotif: function (data) {
                eventBus.$emit('openNotif', data)
            },
            mCollapse: function(n) {
                 this.showCollapse = !this.showCollapse
                 this.myCollapse = n
            },
            changeState: function(data) {
                this.stateNo=data;
            },
         },
         computed: {
             prev: function() {
                 if (this.active == 1) {
                     return
                 }
                 else if (this.active == 2) {
                     return 'Kembali ke PIU'
                 }
                 else if (this.active == 3) {
                     return 'Kembali ke PMU'
                 }
                 else if (this.active == 4) {
                     return 
                 }
             },
             next: function() {
                 if(this.active == 1) {
                     return 'Kirim ke PMU'
                 }
                 else if(this.active == 2) {
                     return 'Kirim ke Penelaah'
                 }
                 else if (this.active == 3) {
                     return 'Triwulan '+this.state['triwulan']
                 } 
                 else if (this.active == 4) {
                     return 
                 }
             }
         },
         created: function () {
            this.initialize()
            eventBus.$on("changeState", this.changeState);
         },
     }
</script>
<style src="css/wizard.css"></style>
<style>

.wizard-active {
    background-color: #007bff;
    border-radius: 5px;
    padding: 0px 0px 0px 50px;
}

.wizard-passive {
    background-color: #808080;
    border-radius: 5px;
    padding: 0px 0px 0px 50px;
}

</style>
