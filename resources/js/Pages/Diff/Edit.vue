<template>
    <app-layout>
        <template #header>
            <!--
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <inertia-link class="text-indigo-400 hover:text-indigo-600" :href="route('diffs')">Diff一覧</inertia-link>
                /
                <span v-if="route().current('diffs.show')">
                    {{ form.title }}
                </span>
                <span v-else>
                    新規作成
                </span>
            </h2>
            -->
            <d-nav-diff :diff="diff" :member="member" v-if="diff" />

        </template>
        <div>
            <div>
                    

                <d-card-content>
                        <!-- Diff内Navigation -->
                        
                    <label class="text-gray-600 font-light">タイトル</label>
                    <input type='text' placeholder="Enter your input here" class="w-full mt-2 mb-6 px-6 py-3 border rounded-lg text-lg text-gray-700 focus:outline-none" @input="inputTitle" :value="form.title" />
                    <p>二つのテキストの差分を表示します。</p>
                    <div class="flex">
                        <div class="w-full w-1/2 pr-2">
                            <textarea @input="inputSourceText" :value="form.source_text" rows="5" class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none"></textarea>
                            <text-result :isAdded="true" :result="compared"></text-result>
                        </div>
                        <div class="w-full w-1/2 pl-2">
                            <textarea @input="inputComparedText" :value="form.compared_text" rows="5" class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none"></textarea>
                            <text-result :isRemoved="true" :result="compared"></text-result>
                        </div>

                    </div>
                    
                </d-card-content>
                <div class="bg-yellow-200 rounded justify-between items-center mt-4">
                            <!--font awesome test-->
                    <div v-if="diff && diff.locked && !alone" class="p-3 text-sm font-midium">
                            <i class="fas fas fa-lock"></i>

                        {{ diff.locked.user.user_name }}によってロックされています。
                    </div>
                </div>
                <div class="mt-4" v-if="member && member.authority <= 1 || diff == null">
                        
                        <div class="flex float-right mb-5">
                            <button v-if="canUnLock && !alone" type="button" class="bg-gray-600 text-white rounded hover:bg-gray-500 px-4 py-2 focus:outline-none mr-2" @click="unlock">ロック解除</button>
                            <button 
                                v-else-if="diff !== null && !alone" 
                                type="button" 
                                class="rounded px-4 py-2 focus:outline-none mr-2"
                                :class="[ locked ? 'text-gray-300 bg-gray-500' : 'text-white hover:bg-gray-500 bg-gray-600 ']"
                                @click="lock">ロック</button>

                            <button type="button" class="rounded  px-4 py-2 focus:outline-none" @click="trySave" :class="[locked ? 'text-gray-300 bg-blue-500' : 'text-gray-200 hover:bg-blue-500 bg-blue-600']">保存</button>

                        </div>
                    </div>
            </div>
        </div>
    </app-layout>
</template>
<script>
const Diff = require('diff');
import AppLayout from '@/Layouts/AppLayout';
import TextDiffResult from './TextDiffResult';
import JetNavLink from '@/Jetstream/NavLink'
import DiffNav from './DiffNav';
import CardContent from './../../Templetes/CardContent';
import throttle from 'lodash/throttle'


export default {
    props: {
        diff: {
            type: Object,
            required: false,
            default: null
        },
        me: {
            type: Object,
            required: true,
        },
        client_id: {
            type: String,
            required: false
        },
        member: {
            type: Object,
            required: false
        }
    },

    data(){
        return {
            form: {
                source_text: this.diff ?this.diff.source_text : '',
                compared_text: this.diff ? this.diff.compared_text : '',
                title: this.diff ? this.diff.title : ''
            },
        }
    },
    components: {
        'app-layout': AppLayout,
        'text-result':TextDiffResult,
        'd-nav-diff': DiffNav,
        JetNavLink,
        'd-card-content': CardContent
    },
    /*watch: {
        form: {
            handler: throttle(function(){
                console.log("変更があった");
                if(this.lockedByMe){
                    this.update();
                }
            }, 200),
            deep: true
        }
    },*/

    computed: {
        compared(){
            if(this.form.source_text !=null && this.form.compared_text != null){
                let diffs = Diff.diffChars(this.form.source_text, this.form.compared_text);
                console.log(diffs);
                return diffs;
            }
        },
        locked(){
            return this.diff != null && this.diff.locked != null && this.diff.locked.user.id !== this.me.id;
        },
        alone(){
            return this.diff.members_count == 1;
        },
        lockedByMe(){
            return this.diff != null && this.diff.locked != null && this.diff.locked.user.id === this.me.id;
        },
        canUnLock(){
            return this.diff && this.diff.locked && (this.diff.locked.user.id === this.me.id || this.member.authority == 0 ||  !this.alone) 
        }
    },

    

    methods: {
        trySave(){
            if(!this.locked){
                if(this.diff == null){
                    this.$inertia.post(this.route('diffs.create'), this.form, {
                        onFinish(){
                            console.log("保存完了")
                        }
                    });
                }else{
                    let data = {
                        source_text: this.form.source_text,
                        compared_text: this.form.compared_text,
                        title: this.form.title,
                        unlock: true,
                        client_id: this.client_id
                    }
                    this.$inertia.put(this.route('diffs.save', this.diff.id), data);
                }
            }
        },
        update(){
            if(this.lockedByMe){
                console.assert(this.client_id != null, 'client_idが異常値である');
                let data = {
                    source_text: this.form.source_text,
                    compared_text: this.form.compared_text,
                    title: this.form.title,
                    unlock: false,
                    client_id: this.client_id
                };
                console.log(data);
                this.$inertia.put(this.route('diffs.update', this.diff.id), data);
            }
        },

        lock(){
            if(this.diff != null && !this.diff.locked){
                this.$inertia.put(this.route('diffs.lock', this.diff.id), null, {
                    onFinish(){
                        console.log('ロック完了');
                    }
                })
            }
        },
        unlock(){
            if(this.canUnLock){
                this.$inertia.delete(this.route('diffs.unlock', this.diff.id), {
                    onFinish(){
                        console.log('ロック解除完了');
                    }
                })
            }
        },

        inputTitle(e){
            let value = e.target.value;
            if(value !== this.form.title){
                this.form.title = value;
                this.tryUpdate();

            }
        },
        inputSourceText(e){
            let value = e.target.value;
            if(value !== this.form.source_text){
                this.form.source_text = value;
                this.tryUpdate();

            }
        },
        inputComparedText(e){
            let value = e.target.value;
            if(value !== this.form.compared_text){
                this.form.compared_text = value;
                this.tryUpdate();
            }
        },

        tryUpdate: throttle(
            function(){
                this.update();
            },
            500
        ),

        startListen(){
            this.$echo.private('diffs.updated.' + this.diff.id)
                .listen('DiffUpdated', (e)=>{
                        console.log(this.client_id == e.client_id);

                    if(this.client_id != e.client_id){
                        this.form.source_text = e.diff.source_text;
                        this.form.compared_text = e.diff.compared_text;
                        this.form.title = e.diff.title;
                        this.$inertia.replace(route('diffs.show', this.diff.id));
                    }
                    

                });
            this.$echo.private('diffs.locked.' + this.diff.id)
                .listen('DiffLocked', (e)=>{
                    console.log(e);
                    this.$inertia.replace(route('diffs.show', this.diff.id));
                });

            this.$echo.private('diffs.unlocked.' + this.diff.id)
                .listen('DiffUnlocked', (e)=>{
                    console.log(e);
                    this.$inertia.replace(route('diffs.show', this.diff.id));
                }) 
            }
    },
    beforeDestroy(){
        if(this.diff != null){
            this.$echo.leave('diffs.updated.' + this.diff.id)
        }
    },

    mounted(){
        
        if(this.diff){

            this.startListen();
        }
           
    },
    
}
</script>
<style scoped>


.area1{
    background-color: #e2e2e2;
}
.area2{
    background-color: #c4c4c4;
}


</style>