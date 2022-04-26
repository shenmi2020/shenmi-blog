---
title: qt画板
date: 2019-09-26 15:23
categories: [编程]
tags:
 - [qt]
---

完成qt绘画编辑器功能

<!-- more -->

``` qt
#ifndef JRPAINTER_H
#define JRPAINTER_H
#include <QQuickPaintedItem>
#include <QVector>
#include <QPointF>
#include <QImage>

class JRPainter : public QQuickPaintedItem
{
    Q_OBJECT

//    Q_PROPERTY(int penWidth READ penWidth WRITE setPenWidth)

public:
    JRPainter(QQuickItem *parent = 0);

    void paint(QPainter *painter);

    void mousePressEvent(QMouseEvent *event);
    void mouseMoveEvent(QMouseEvent *event);
    void mouseReleaseEvent(QMouseEvent *event);

    Q_INVOKABLE void init();
 //   Q_INVOKABLE void clear();
    Q_INVOKABLE void stop() {
        qDebug() << "Gemini::stop() called";
    }
    enum shape {
        Line = 1,Ellipse,Rect,Triangle
    };
    Q_ENUM(shape)
    Q_INVOKABLE void setShape(shape);
    Q_INVOKABLE void setPenWidth(int w);
    Q_INVOKABLE void setPenColor(QColor c);
signals:

public slots:

private:
    void drawShape(QImage &theImage);


private:
   QVector<QPointF> point;
   QImage m_bgImage;
   QImage m_tempImage;
   bool isDrawing;
   QPointF m_nowPoint;
   QPointF m_lastPoint;
   JRPainter::shape type;
   int penWidth;
   QColor penColor;
};

#endif // JRPAINTER_H
　　

#include "jrpainter.h"
#include <QPainter>
#include <QImage>

JRPainter::JRPainter(QQuickItem *parent):
    isDrawing(false),
    type(Line),
    penWidth(4),
    penColor(QColor(255,0,0))
{
    setAcceptedMouseButtons(Qt::LeftButton);
}

void JRPainter::setShape(JRPainter::shape t)    // 设置绘制类型
{
    type = t;
}

void JRPainter::setPenWidth(int w)    // 设置画笔宽度
{
    penWidth = w;
}

void JRPainter::setPenColor(QColor c)    // 设置颜色
{
    penColor = c;
}

void JRPainter::paint(QPainter *painter)
{
    if(isDrawing){
        painter->drawImage(0,0,m_tempImage);
    }else{
        painter->drawImage(0,0,m_bgImage);
    }
}
void JRPainter::init()
{
    QImage img(":/res/background/Farm.svg");
    m_bgImage = img;
}

void JRPainter::mousePressEvent(QMouseEvent *event)
{
    m_lastPoint = event->localPos();
    isDrawing = true;
}

void JRPainter::mouseMoveEvent(QMouseEvent *event)
{
    m_nowPoint = event->localPos();
    m_tempImage = m_bgImage;
    if(type == Line){
        drawShape(m_bgImage);
    }else{
        drawShape(m_tempImage);
    }
}

void JRPainter::mouseReleaseEvent(QMouseEvent *event)
{
    isDrawing = false;
    if (type != Line) {
       drawShape(m_bgImage);
    }
}

void JRPainter::drawShape(QImage &theImage)
{
    QPainter painter(&theImage);
    QPen apen;
    apen.setWidth(penWidth);
    apen.setColor(penColor);
    painter.setPen(apen);
    painter.setRenderHint(QPainter::Antialiasing, true);
    int x1, y1, x2, y2;
    x1 = m_lastPoint.x();
    y1 = m_lastPoint.y();
    x2 = m_nowPoint.x();
    y2 = m_nowPoint.y();

    switch (type) {
        case JRPainter::Line:
        {
            painter.drawLine(m_lastPoint,m_nowPoint);
            m_lastPoint = m_nowPoint;
            break;
        }
        case JRPainter::Ellipse:
        {
            painter.drawEllipse(x1,y1,x2-x1,y2-y1);
            break;
        }
        case JRPainter::Rect:
        {
            painter.drawRect(x1, y1, x2 - x1, y2 - y1);
            break;
        }
        case JRPainter::Triangle:
        {
            int top, buttom, left, right;
            top = (y1 < y2) ? y1 : y2;
            buttom = (y1 > y2) ? y1 : y2;
            left = (x1 < x2) ? x1 : x2;
            right = (x1 > x2) ? x1 : x2;
            if (y1 < y2)
            {
                QPoint points[3] = { QPoint(left,buttom), QPoint(right,buttom), QPoint((right + left) / 2,top) };
                painter.drawPolygon(points, 3);
            }
            else
            {
                QPoint points[3] = { QPoint(left,top), QPoint(right,top), QPoint((left + right) / 2,buttom) };
                painter.drawPolygon(points, 3);
            }
            break;
        }
    }
    update();
}
```

qml如果使用c++中定义的方法，需要

Q_INVOKABLE void init();
qml中 painter.init();调用即可
感谢https://blog.csdn.net/u013165921/article/details/79393265的博客