---
title: qml分类
date: 2019-08-28 15:28
categories: [编程]
tags:
 - [qt]
---

模拟treeview

<!-- more -->
```
RowLayout {
    anchors.fill: parent
    //  anchors.top: header.bottom
    spacing: 50
    Rectangle {
        Layout.preferredWidth: 150
        Layout.preferredHeight: 600
        ListView{
            id:listView
            anchors.fill: parent
            anchors.top: parent.top
            anchors.topMargin:20
            spacing: 20
            model: ListModel{
                id:listModel
                ListElement {
                    bgpId: 2
                    bgpName: '全部'
                    level: 0
                    subNode: []
                }
            }
            delegate: list_delegate
            Component.onCompleted: {
                //获取分类
                get("",
                    function(result,json){
                        for(var o in json) {
                            if(json[o].parentId == 2){
                                for(var cid in json){
                                    if(json[cid].parentId == json[o].id){
                                        addModelData(json[o].id,json[o].name,json[cid].id,json[cid].name)
                                    }
                                }
                            }
                        }
                    }
                )
                //获取全部背景
                getInfo(2)
            }
        }
    }
    Rectangle {
        Layout.alignment: Qt.AlignTop
        Layout.preferredWidth: parent.width-200
        Layout.preferredHeight: parent.height
        ScrollView {
            anchors.fill: parent
            verticalScrollBarPolicy: Qt.ScrollBarAlwaysOff
            Flow {
                width: parent.parent.width
                rightPadding: 30
                spacing: 30
                Repeater {
                    model: contactModel
                    Image {
                        sourceSize.height: 100
                        source: imgurl
                        Text {
                            width: parent.width
                            anchors.top: parent.bottom
                            horizontalAlignment: Text.AlignHCenter
                            font.pointSize: 10
                            text: name
                        }
                    }
                }
            }
        }
    }
}
ListModel {
    id: contactModel
}
Component{
    id:list_delegate
    Column{
        id:objColumn
        Component.onCompleted: {
            for(var i = 1; i < objColumn.children.length - 1; ++i) {
                objColumn.children[i].visible = false
            }
        }
        MouseArea{
            width:listView.width
            height: objItem.implicitHeight
        //    enabled: objColumn.children.length > 2
            onClicked: {
                console.log(bgpId)
                //根据分类Id获取背景数据
                getInfo(bgpId)
                var flag = false
                for(var i = 1; i < parent.children.length - 1; ++i) {
                    flag = parent.children[i].visible;
                    parent.children[i].visible = !parent.children[i].visible
                }
                if(!flag){
                    iconAin.from = 0
                    iconAin.to = 90
                    iconAin.start()
                }
                else{
                    iconAin.from = 90
                    iconAin.to = 0
                    iconAin.start()
                }
            }
            Row{
                id:objItem
                spacing: 10
                leftPadding: 20
                Image {
                    id: icon
                    width: 10
                    height: 20
                    source: "../res/blockicons/Foward.svg"
                    anchors.verticalCenter: parent.verticalCenter
                    RotationAnimation{
                        id:iconAin
                        target: icon
                        duration: 100
                    }
                }
                Label {
                    text: bgpName
                    color:"grey"
                    anchors.verticalCenter: parent.verticalCenter
                }
            }
        }
        Repeater {
            model: subNode
            delegate: Rectangle{
                width: parent.width
                height: 40
                Column{
                    anchors{
                        leftMargin: 20
                        top: parent.top
                    }
                    topPadding: 10
                    spacing: 2
                    Label{
                        horizontalAlignment: Text.AlignHCenter
                        width: 100
                        text: model.bgcName
                        MouseArea {
                            anchors.fill: parent
                            onClicked: {
                                console.log(model.bgcId)
                                //根据子分类Id获取背景数据
                                getInfo(model.bgcId)
                            }
                        }
                    }
                }
            }
        }
    }
}
//添加
function addModelData(bgpId,bgpName,bgcId,bgcName){
    var index = findIndex(bgpId)
    if(index === -1){
        listModel.append({"bgpId":bgpId,"bgpName":bgpName,"level":0,
            "subNode":[{"bgcId":bgcId,"bgcName":bgcName,"level":1,"subNode":[]}]})
    }
    else{
        listModel.get(index).subNode.append({"bgcId":bgcId,"bgcName":bgcName,"level":1,"subNode":[]})
    }
}
//查找
function findIndex(id){
    for(var i = 0 ; i < listModel.count ; ++i){
        if(listModel.get(i).bgpId === id){
            return i
        }
    }
    return -1
}
```