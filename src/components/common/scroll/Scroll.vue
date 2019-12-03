<template>
    <div class="wrapper" ref="wrapper">
    <div class="content">
     <slot>
         
     </slot>
     </div>
    </div>
</template>

<script>
import BScroll from 'better-scroll'
    export default {
        name:"Scroll",
        props:{
          probeType:{
              type:Number,
              default:0
          },
          pullUpload:{
            type:Boolean,
            default:false
          }
        },
        data() {
            return {
                scroll:null
            }
        },
        mounted() {
            this.scroll = new BScroll(document.querySelector(this.$ref.wrapper),{
                  click:true,
                  probeType:this.probeType,
                  pullUpload:this.pullUpload    
            })
            this.scroll.on('scroll',(position)=>{

                this.$emit('scroll',position)
            })
           
        },
        methods: {
            scrollTo(x,y,time=300){
               this.scroll.scrollTo(x,y,time)
            }
        }, 
        finishPullup(){
            this.scroll.finishPullup()
        },
        refresh(){
            this.scroll.refresh()
        }
    }
</script>

<style scoped>

</style>