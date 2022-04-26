---
title: gridview
date: 2019-08-30 14:53
categories: [编程]
tags:
 - [qt]
---

开始时使用flow布局，ajax请求的图片闪烁，后来改用gridview解决问题
<!-- more -->
``` qt
GridView {
    cellWidth: 200
    cellHeight: 150
    model: contactModel
    delegate: Column {
        width: 200
        height: 150
        Image {
            width: parent.width
            horizontalAlignment: Image.AlignHCenter
            sourceSize.height: 100
            fillMode: Image.PreserveAspectFit
            source: imgurl
            Text {
                anchors.bottomMargin: 30
                width: parent.width
                anchors.top: parent.bottom
                horizontalAlignment: Text.AlignHCenter
                font.pointSize: 10
                text: name
            }
        }
    }
}
```