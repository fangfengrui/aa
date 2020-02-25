<template>
    <canvas id="canvas" width="520" height="350" style="border: 1px solid #999;"></canvas>
    <div id="keyword-box">
        <span>Keyword: </span>
        <span id="keyword"></span>
    </div>
    <button id="btn">清空画布</button>
    <input id='binput' type="color"/>
    <button  id="bbtn">白色</button>
    <button  id="ablack">黑色</button>
    <button  id="ared">红色</button>
    <button  id="agreen">绿色</button>
    <button id="addWidth" >增加宽度+5</button>
    <button id="deWidth" >减少宽度-5</button>
    <button id='eraser'>橡皮擦</button>
</template>

<script>
'use strict'

class Draw {
    constructor(el) {
        this.el = el
        this.canvas = document.getElementById(this.el)
        this.cxt = this.canvas.getContext('2d')
        this.stage_info = canvas.getBoundingClientRect()
        this.path = {
            beginX: 0,
            beginY: 0,
            endX: 0,
            endY: 0
        }
        this.color="#2196f3"
        this.width=5
        this.clear=false
    }

    init(ws, btn) {
        this.canvas.onmousedown = () => {
            this.drawBegin(event, ws)
        }
        this.canvas.onmouseup = () => {
            this.drawEnd()
            ws.send('stop')
        }
        this.setwColor(ws,bbtn)
        this.setbColor(ws,ablack)
        this.setrColor(ws,ared)
        this.setgColor(ws,agreen)
        this.setalineWidth(ws,addWidth)
        this.setdlineWidth(ws,deWidth)
        this.changeColor(ws,binput)
        this.clearCanvas(ws, btn)
        this.trytoEraser(ws,eraser)
    }
    drawBegin(e, ws) {
        window.getSelection() ? window.getSelection().removeAllRanges() : document.selection.empty()
        this.cxt.strokeStyle = this.color
        this.cxt.lineWidth=this.width
        this.cxt.beginPath()
        this.cxt.moveTo(
            e.clientX - this.stage_info.left,
            e.clientY - this.stage_info.top
        )

        this.path.beginX = e.clientX - this.stage_info.left
        this.path.beginY = e.clientY - this.stage_info.top

        document.onmousemove = () => {
            this.drawing(event, ws)
        }
    }
    drawing(e, ws) {

            this.cxt.lineTo(
                e.clientX - this.stage_info.left,
                e.clientY - this.stage_info.top
            )

            this.path.endX = e.clientX - this.stage_info.left
            this.path.endY = e.clientY - this.stage_info.top
            ws.send(this.path.beginX + '.' + this.path.beginY + '.' + this.path.endX + '.' + this.path.endY)

            this.cxt.stroke()
    }
    drawEnd() {
        document.onmousemove = document.onmouseup = null
    }
    
    setwColor(ws, bbtn) {
        bbtn.onclick = () =>{
            this.cxt.globalCompositeOperation="source-over"
            this.color="#FFF"
            console.log(this.color)
            ws.send(this.color+".source-over")
        }
    }
    setbColor(ws, ablack) {
        ablack.onclick = () =>{
            this.cxt.globalCompositeOperation="source-over"
            this.color="black"
            ws.send(this.color+".source-over")
        }
    }
    setrColor(ws, ared) {
        ared.onclick = () =>{
            this.cxt.globalCompositeOperation="source-over"
            this.color="red"
            ws.send(this.color+".source-over")
        }
    }
    setgColor(ws, agreen) {
        agreen.onclick = () =>{
            this.cxt.globalCompositeOperation="source-over"
            this.color="green"
            ws.send(this.color+".source-over")
        }
    }
    changeColor(ws,binput) {
        binput.onchange= ()=>{
            this.cxt.globalCompositeOperation="source-over"
            var Color=document.getElementById("binput");
            this.color=Color.value
            ws.send(this.color+".source-over")
        }
    }
    setalineWidth(ws,addWidth) {
        addWidth.onclick=()=>{
            if(this.width>0){
                this.width+=5
            }
            ws.send(this.width+".width")
        }

    }
    setdlineWidth(ws,deWidth) {
        deWidth.onclick=()=>{
            if(this.width>6){
                this.width-=5
            } 
            ws.send(this.width+".width")
        }
    }
    clearCanvas(ws, btn) {
        btn.onclick = () => {
            this.color="#FFF"
            this.cxt.clearRect(0, 0, 520, 520)
            ws.send('clear')
            console.log('delete了')
        }
    }
    trytoEraser(ws,eraser) {
        let eraser123 = document.getElementById("eraser")
        eraser.onclick= ()=> {
            this.cxt.globalCompositeOperation="destination-out"
            ws.send("destination-out")
        }
    }
}

export default {
    ready() {
        const ws = new WebSocket('ws://localhost:8090')
        let draw = new Draw('canvas')
        let btn = document.getElementById('btn')
        let binput = document.getElementById('binput')
        ws.onopen = () => {
            draw.init(ws, btn)
        }
        
        ws.onmessage = (msg) => {
            msg.data.split(':')[0] == 'keyword' ?
                document.getElementById('keyword').innerHTML = msg.data.split(':')[1] :false
                
        }
    },
    
}
</script>

<style lang="less">
    #canvas {
        background: pink;
        cursor: default;
    }
    #keyword-box {
        margin: 10px 0;
    }
    #bbtn {
        background: white;
    }
</style>