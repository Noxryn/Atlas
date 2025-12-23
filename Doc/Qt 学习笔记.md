# 项目框架及文件
- .pro 工程文件，qmake自动生成的用于生成makefile的配置文件
- .h 
- .cpp
- .ui
## .pro
```
QT       += core gui                             // 组件模块
greaterThan(QT_MAJOR_VERSION, 4): QT += widgets  // Qt4 包含此模块
CONFIG += c++11                                  // C++版本
// TARGET = demo                                 // 应用程序名
// TEMPLATE = app                                // 模板类型
DEFINES += QT_DEPRECATED_WARNINGS                // 编译选项，若功能标记为过时则警告
SOURCES += \                                    // 源文件
    main.cpp \
    mainwindow.cpp
HEADERS += \   // 头文件
    mainwindow.h
FORMS += \
    mainwindow.ui
qnx: target.path = /tmp/$${TARGET}/bin
else: unix:!android: target.path = /opt/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target
```

## main.cpp
```
#include "mainwindow.h"
#include <QApplication> // 标准类名头文件
int main(int argc, char *argv[])
{
    QApplication a(argc, argv); // 应用程序类 - 处理应用程序的初始化和结束，事件处理调度 ，唯一且必须
    MainWindow w;
    w.show();
    return a.exec();  // 主事件循环， Qt接收并处理事件将其传递给对应控件
}
```

# 对象模型（对象树）
概念：Qt对象间的父子关系
解决问题：解决内存问题，简化内存回收
- QObject是以对象树的形式组织起来的
- 创建QObject对象时可以提供一个父对象，该对象会自动添加到其父对象的children()列表
- 父对象析构时，子对象也会被析构
- QWidget是一切UI显示组件的父类
- 注意不要重复析构

# 信号与槽机制
- 观察者模式 松散耦合
```
connect(sender,signal,receiver,slot);
/*
    参数：
        - 信号发出者
        - 信号
        - 接收者
        - 槽函数
*/

```
## 信号重载
```
// 1.重载信号
void hungury(QString food);
// 2.重载槽函数
void treat(QString food){}
// 3.函数指针连接
void (Teacher::*teachersignal)(QString) = &Teacher::hungury;
void (Student::*studentslot)(QString) = &Student::treat;
connect(tea,teachersignal,stu,studentslot);
```


## 一、派生于QThread
重写虚函数void_QThread:run()，在run方法里写具体的内容，外部通过Thread01的实例化对象去调用start方法，即可执行线程体run()。  
派生于QThread的类，构造函数属于主线程，run函数属于子线程，可以通过currentThreadId()方法打印线程id判断。
```cpp
#pragma once
#include <QThread>
 
class Thread01 : public QThread
{
 Q_OBJECT
 
public:
 Thread01();
 void run() override;
};

#include "Thread01.h"
#include <QDebug>
 
Thread01::Thread01()
{
 qDebug() << "Thread01 construct " << QThread::currentThreadId();
}
 
void Thread01::run()
{
 qDebug() << "Thread01 run " << QThread::currentThreadId();
 
 int index = 0;
 while (1)
 {
  qDebug() << index++;
  QThread::msleep(500); //加一个延迟执行，500毫秒
 }
}

#include <QtCore/QCoreApplication>
#include <iostream>
#include "Thread01.h"
 
using namespace std;
 
 
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    cout << "main thread" << QThread::currentThreadId() << endl;
 
    Thread01 th;
    th.start();
 
    cout << "main thread end" << endl;
    return a.exec();
}

```
## 二、派生于QRunnable
1、具体思路
创建一个类Thread02，派生于QRunnable，重写run()方法，在run方法里处理其它任务，不同的是，调用时不再是start方法，而是需要借助Qt线程池QThreadPool
MyThread *pTh = new MyThread();
QThreadPool:globallnstance()->start(pTh);

注意：
这种新建线程的方法的最大的缺点就是:不能使用Qt的信号槽机制，因为QRunnable不是继承自Q0bject。但是这种方法的好处就是，可以让QThreadPool来管理线程，QThreadPool会
自动的清理我们新建的QRunnable对象。 

2、区分线程
同样的，派生于QThread的类，构造函数属于主线程，run函数属于子线程，可以通过currentThreadId()方法打印线程id判断。
```cpp
#pragma once
#include <QRunnable>
 
class Thread02 : public QRunnable
{
public:
 Thread02();
 ~Thread02();
 void run() override;
};

#include "Thread02.h"
#include <QThread>
#include <QDebug>
 
Thread02::Thread02()
{
 qDebug() << "Thread02 construct " << QThread::currentThreadId();
}
 
Thread02::~Thread02()
{
 qDebug() << "Thread02 xigou func";
}
 
void Thread02::run()
{
 qDebug() << "Thread02 run " << QThread::currentThreadId();
}

#include <QtCore/QCoreApplication>
#include <iostream>
#include "Thread02.h"
#include <QThreadPool>
 
using namespace std;
 
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    cout << "main thread" << QThread::currentThreadId() << endl;
 
    Thread02 *th = new Thread02();
    QThreadPool::globalInstance()->start(th);
 
    cout << "main thread end" << endl;
    return a.exec();
}
```

## 三、moveToThread
1、具体思路
创建一个类Thread03，派生于QObject,使用moveToThread方法将QThread对象作为私有成员，在构造函数里moveToThread，然后调用start方法启动线程。
this->moveToThread(&m_th);
m_th.start(); .

```cpp
#include "ch71_moveToThread.h"
#include <QDebug>
 
ch71_moveToThread::ch71_moveToThread(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
 
 qDebug() << "main construct " << QThread::currentThreadId();
 
    m_pTh03 = new Thread03();
 
 //connect(this, &ch71_moveToThread::sig_fun, m_pTh03, &Thread03::fun);
}
 
ch71_moveToThread::~ch71_moveToThread()
{
 
}
 
void ch71_moveToThread::on_pushButton_clicked()
{
    //m_pTh03->fun();
 
 //emit sig_fun();
 
 int index = 0;
 while (1)
 {
  qDebug() << index++;
  QThread::msleep(300);
 }
}

#include "ch71_moveToThread.h"
#include <QDebug>
 
ch71_moveToThread::ch71_moveToThread(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
 
 qDebug() << "main construct " << QThread::currentThreadId();
 
    m_pTh03 = new Thread03();
 
 //connect(this, &ch71_moveToThread::sig_fun, m_pTh03, &Thread03::fun);
}
 
ch71_moveToThread::~ch71_moveToThread()
{
 
}
 
void ch71_moveToThread::on_pushButton_clicked()
{
    m_pTh03->fun();
 
 //emit sig_fun();
 
 /*int index = 0;
 while (1)
 {
  qDebug() << index++;
  QThread::msleep(300);
 }*/
}

#include "ch71_moveToThread.h"
#include <QDebug>
 
ch71_moveToThread::ch71_moveToThread(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
 
 qDebug() << "main construct " << QThread::currentThreadId();
 
    m_pTh03 = new Thread03();
 
 connect(this, &ch71_moveToThread::sig_fun, m_pTh03, &Thread03::fun);
}
 
ch71_moveToThread::~ch71_moveToThread()
{
 
}
 
void ch71_moveToThread::on_pushButton_clicked()
{
 emit sig_fun();
}
```
通过使用信号槽机制，将耗时操作放在另一个线程中执行，可以保持界面的流畅性和响应性。