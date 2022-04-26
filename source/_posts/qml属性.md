---
title: qml属性
date: 2019-08-06 10:50
categories: [编程]
tags:
 - [qt]
---
qml中文文档
http://cryfeifei.gitee.io/qmlbook-in-chinese/meet_qt_5/

https://www.kancloud.cn/kancloud/qt-study-road-2/99519 

电子书

https://www.jb51.net/books/565672.html

<!-- more -->

案例

https://www.kancloud.cn/cloudcastle/qt5-demo/109874

记录一些用到的qml属性

Text{

wrapMode: Text.WordWrap　　//自动换行
}

----------

使用Shape画圆，Canvas渲染性能并不太好
Shape {
    id: paiarc
    width: 200
    height: 150
    anchors.centerIn: parent
    ShapePath {
        strokeWidth: 1
        strokeColor: "black"
        startX: 50; startY: 60
        PathAngleArc {
            centerX: 50; centerY: 50
            radiusX: 10; radiusY: 10
            sweepAngle: 360
        }
    }
}