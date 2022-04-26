---
title: js中ownerDocument与documentElement的区别
date: 2019-12-20 10:20
categories: [编程]
tags:
 - [js]
---

ownerDocument是Node对象的一个属性。返回的是某个元素的根节点文档对象，即document 对象

documentElement是document对象的属性，返回的是文档根节点。

<!-- more -->

``` html
/**
* 触摸屏幕模拟键盘按下事件
*/

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>Document</title>
</head>
<style>
.panel {
width: 100%;
height: 200px;
position: absolute;
}
.key {
width: 160px;
height: 148px;
position: absolute;
left: 20px;
top: 16px;
}
.key div {
width: 60px;
height: 60px;
position: absolute;
}
.arrow_left {
left: 0;
top: 44px;
background: url('../img/arrow_left.png') no-repeat center/contain;
}
.arrow_down {
bottom: 0;
left: 51px;
background: url('../img/arrow_down.png') no-repeat center/contain;
}
.arrow_right {
right: 0;
top: 44px;
background: url('../img/arrow_right.png') no-repeat center/contain;
}
.arrow_up {
left: 51px;
top: 0;
background: url('../img/arrow_up.png') no-repeat center/contain;
}

</style>
<body>
<div class="panel">
<div class="key">
<div id="left" class="arrow_left"></div>
<div id="down" class="arrow_down"></div>
<div id="right" class="arrow_right"></div>
<div id="up" class="arrow_up"></div>
</div>
</div>
</body>
<script>

document.getElementById('left').addEventListener('touchstart',function (e) {
createKeyEvent(e.target, 'keydown', 37, "ArrowLeft");
});
document.getElementById('down').addEventListener('touchstart',function (e) {
createKeyEvent(e.target, 'keydown', 40, "ArrowDown");
});
document.getElementById('right').addEventListener('touchstart',function (e) {
createKeyEvent(e.target, 'keydown', 39, "ArrowRight");
});
document.getElementById('up').addEventListener('touchstart',function (e) {
createKeyEvent(e.target, 'keydown', 38, "ArrowUp");
});

/**
* 监听键盘按下
*
*/
window.addEventListener("keydown", function (event) {
console.log('down')
console.log(event)
if (event.defaultPrevented) {
return;
}
var handled = false;
if (event.key !== undefined) {
// Handle the event with KeyboardEvent.key and set handled true.
if(event.key === " "){
// space.classList.add('active')
console.log('space keydown')
}else if(event.key === 'a'){
// space.classList.remove('active')
console.log('a keydown')
}
} else if (event.keyIdentifier !== undefined) {
// Handle the event with KeyboardEvent.keyIdentifier and set handled true.
console.log( 'event.keyIdentifier',event.keyIdentifier);
} else if (event.keyCode !== undefined || event.which !== undefined) {
// Handle the event with KeyboardEvent.keyCode and set handled true.
if(event.keyCode === 32){
space.classList.add('active')
}else if(event.keyCode === 65){
space.classList.remove('active')
}
}
if (handled) {
// Suppress "double action" if event handled
event.preventDefault();
}
}, true);


/*
* 给元素添加事件（用来触发键盘事件）
* el模拟事件的触发元素，evtType事件类型，keycode对应的键盘事件的键值,key对应事件所触发的键名,
*/
const createKeyEvent = (el, evtType, keyCode ,key) => {
console.log(el.ownerDocument)
console.log(document)
var doc = document,
win = doc.defaultView || doc.parentWindow,
evtObj;
if(doc.createEvent){
if(win.KeyEvent) {
console.log(111)
evtObj = doc.createEvent('KeyEvents');
evtObj.initKeyEvent( evtType, true, true, win, false, false, false, false, keyCode, 0 );
}
else {

evtObj = doc.createEvent('UIEvents');
Object.defineProperty(evtObj, 'keyCode', {
get : function() { return this.keyCodeVal; }
});
Object.defineProperty(evtObj, 'which', {
get : function() { return this.keyCodeVal; }
});
evtObj.initUIEvent( evtType, true, true, win, 1 );
evtObj.key = key;
evtObj.keyCodeVal = keyCode;
if (evtObj.keyCode !== keyCode) {
console.log("keyCode " + evtObj.keyCode + " 和 (" + evtObj.which + ") 不匹配");
}
}
el.dispatchEvent(evtObj);
}
else if(doc.createEventObject){
evtObj = doc.createEventObject();
evtObj.keyCode = keyCode;
el.fireEvent('on' + evtType, evtObj);
}
};
</script>
</html>
```