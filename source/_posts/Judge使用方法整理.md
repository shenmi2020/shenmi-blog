---
title: hustoj特判程序special Judge使用方法整理
date: 2019-05-05 16:03
categories: [编程]
tags:
 - [hustoj]
---

Special Judge

通常的ACM题目包括以下几项内容：题目描述(Description)、输入描述(Input)、输出描述(Output)、样例输入(Sample Input)、样例输出(Sample Out)，在后台则包括测试输入(Input Data)和测试输出(Output Data)两项。在评测用户提交的程序正确与否时，系统会将样例输入和测试输入重定向作为程序的标准输入，通过判断程序对应的输出是否与期待的输出完全相同，来判断解答是否正确。

对于同一道题目，用户可能使用各种不同的方法来解答，所以对于某些特殊的题目，其结果可能不唯一，但都符合题目要求。此类题目就需要进行特判(Special Judge)。HUSTOJ便提供了特判功能。

<!-- more -->

这些题目主要有两种：
1、答案不唯一。
2、控制精度。题目要求输出精度误差在某eps之内。

 

【使用方法】

第一步，在添加题目时，Special judge勾选Y，以打开系统的特判命令。

 

 

第二步，编写spj代码，模板中标注了spj代码区域，自行根据题目要求编写。注意文件名最后以.cc为后缀。

【模板】控制精度为例，对比输出文件和用户结果文件，误差不超过1e-5则Accepted。

#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<math.h>
#include<set>
#include<utility>
using namespace std;

int main(int argc, char* argv[]) {

FILE * f_in=fopen(argv[1],"r");//测试输入
FILE * f_out=fopen(argv[2],"r");//测试输出
FILE * f_user=fopen(argv[3],"r");//用户输出
int ret=0; //AC=0,WA=1

/*****spj代码区域*******/
int T,n;
fscanf(f_in,"%d",&T);
while(T--)
{
double a,b;
int n;
fscanf(f_in,"%d",&n);
fscanf(f_out,"%lf",&a);
fscanf(f_user,"%lf",&b);
if(fabs(a-b)>1e-5) //WA
{
ret=1;
break;
}
}

/*****spj-end********/

fclose(f_in);
fclose(f_out);
fclose(f_user);
return ret;
}
【模板解释】

HUSTOJ的spj用了命令行参数argc和argv，其中argv[1]指向输入数据（即题目测试输入），argv[2]指向输出数据（题目测试的输出数据，有时不需要），argv[3]指向用户提交的程序所运行输出的文件。分别读取三个文件，做相应的对比。main函数的返回值表示判断结果。 当发现用户的输出文件有误时，return 1；  完全正确时，return 0；

注意读取三个文件时全部用fscanf（C++中fstream库操作也是可以的），读取出来后，只需要自己编写一下如何判断用户的输出是否正确。  可能有些题目很难，不好编写，这时候可以把标程放在spj区域（注意输入都改成fscanf），对比标程运行出来的结果和用户结果的差异是否在题目允许范围之内，来判断用户是否正确。

 

第三步，将.cc文件上传到服务器对应题目的数据文件夹下（也就是保存测试数据的那个文件夹），可以在上传数据时一同上传

 

第四步，在服务器上测试数据文件夹下，命令行执行   g++ -o spj spj.cc    （spj是编译后的可执行程序，不要改名。spj.cc是你上传的代码文件，名称随意），个别服务器可能还需要权限，再执行一下：chmod +x spj              推荐一个windows下远程服务器命令：Putty，百度搜索即可下载。

 

第五步，测试。该步骤可省略。在上一步的命令行中，继续执行：./spj data.in data.out user.out  ，然后再执行：echo $?   然后窗口会显示1（代表WA）或者0（代表AC），其中data.in 和data.out是测试数据，user.out是用户代码运行的结果。该步操作可模拟进行判断用户输出是否正确。

 

第六步，提交代码测试题目。建议提交一遍正确代码，再故意改动一下AC代码，提交错误代码，看是否成功WA。

 
---------------------
作者：雪的期许
来源：CSDN
原文：https://blog.csdn.net/winter2121/article/details/81841696
版权声明：本文为博主原创文章，转载请附上博文链接！