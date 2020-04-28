---
title: C语言总结小项目-图书管理系统
date: 2020-04-06 21:00:52
tags:
    - C语言
    - 图书
categories: [学习笔记]
---

学习软件逆向的第一步，用一个小程序总结一下。

<!-- more -->

## main.h

```
#pragma once
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Node
{
	char BookName[50];	//书名
	float BookPrice;	//价格
	int BookNumber;		//书号
	struct Node *next;	//指向下一个节点的指针
}Node;
//添加
Node * AppendNode(Node * head, char * BookName, float BookPrice, int BookNumber);
//按书名查询
void QueryNode(Node * head, char * BookName);
//按书号查找
void QueryNode2(Node * head, int BookNumber);
//修改
void ModifyNode(Node * head, char * BookName, float BookPrice);
//删除
Node * DeleteNode(Node * head, char * BookName);
//查看全部
void AllNode(Node * head);
```

## main.c

```
#include "main.h"


int main()
{
	printf("------图书管理系统------\n");
	Node * head = NULL;	//链表头指针
	while (1)
	{
		char szBookName[50];
		float fBookPrice;
		float fNewBookPrice;
		int nBookNumber;
		int Readflag = 0;
		int Queryflag = 0;
		printf("      1.添加书籍\n");
		printf("      2.查询书籍\n");
		printf("      3.修改书籍\n");
		printf("      4.删除书籍\n");
		printf("      5.查看全部\n");
		printf("      6.退出系统\n");
		printf("请选择您要使用的功能：\n");
		scanf("%d", &Readflag);

		switch (Readflag)
		{
		case 1:
			//添加书籍信息

			printf("请输入书名：\n");
			scanf("%s", szBookName);
			printf("请输入定价：\n");
			scanf("%f", &fBookPrice);
			printf("请输入书号：\n");
			scanf("%d", &nBookNumber);
			printf("添加成功！\n");
			head = AppendNode(head, szBookName, fBookPrice, nBookNumber);
			break;
		case 2:
			printf("请选择查找方式：\n");
			printf("     1.按书名查找\n");
			printf("     2.按书号查找\n");
			printf("请选择需要使用的功能：");
			scanf("%d", &Queryflag);
			if (Queryflag == 1)
			{
				printf("请输入书名：\n");
				scanf("%s", szBookName);
				QueryNode(head, szBookName);
			}
			else if (Queryflag == 2)
			{
				printf("请输入书号：\n");
				scanf("%d", &nBookNumber);
				QueryNode2(head, nBookNumber);
			}
			break;
		case 3:
			printf("请输入书名：\n");
			scanf("%s", szBookName);
			printf("请输入新的定价：");
			scanf("%f",&fNewBookPrice);
			ModifyNode(head, szBookName, fNewBookPrice);
			break;
		case 4:
			printf("请输入书名：\n");
			scanf("%s", szBookName);
			printf("删除成功！\n");
			head = DeleteNode(head, szBookName);
			break;
		case 5:
			AllNode(head);
			break;
		case 6:
			exit(0);
		default:
			break;
		}
	}

	return 0;
}

//增
Node * AppendNode(Node * head, char * BookName, float BookPrice, int BookNumber)
{
	//声明一个新的节点
	Node * pNewNode = NULL;
	//声明一个指向头指针的指针
	Node * pHeadNode = head;
	//申请一个内存
	pNewNode = (Node *)malloc(sizeof(Node));
	//判断内存有没有申请成功
	if (pNewNode == NULL)
	{

		printf("memory malloc failed!\n");
		exit(0);
	}

	if (head == NULL)
	{
		//如果没有头节点，就把头指针指向新申请的内存
		head = pNewNode;
	}
	else
	{
		//判断有没有到链表结尾
		while (pHeadNode->next != NULL)
		{
			pHeadNode = pHeadNode->next;
		}
		//如果指向最后一个节点就重新指向到新的最后一个节点
		pHeadNode->next = pNewNode;
	}
	//书名
	strcpy(pNewNode->BookName, BookName);
	//定价
	pNewNode->BookPrice = BookPrice;
	//书号
	pNewNode->BookNumber = BookNumber;
	//把最后一个节点指向NULL
	pNewNode->next = NULL;
	return head;
}
//查找
void QueryNode(Node * head, char * BookName)
{
	int flag = 0;
	if (head == NULL)
	{
		printf("head NULL!\n");
		exit(0);
	}
	//判断信息是否一致
	if (strcmp(BookName,head->BookName) == 0)
	{
		printf("书价\t书名\t书号\n");
		printf("%0.2f\t%s\t%d\n", head->BookPrice, head->BookName, head->BookNumber);
		flag = 1;
	}
	//如果不是空
	while (head->next != NULL)
	{
		head = head->next;
		if (strcmp(BookName, head->BookName) == 0)
		{
			printf("书价\t书名\t书号\n");
			printf("%0.2f\t%s\t%d\n", head->BookPrice, head->BookName, head->BookNumber);
			flag = 1;
		}
	}

	if (flag == 0)
	{
		printf("查询失败!!\n");
	}
	
}
//按书号查找
void QueryNode2(Node * head, int BookNumber)
{
	int flag = 0;
	if (head == NULL)
	{
		printf("head NULL!\n");
		exit(0);
	}
	//判断信息是否一致
	if (BookNumber == head->BookNumber)
	{
		printf("书价\t书名\t书号\n");
		printf("%0.2f\t%s\t%d\n", head->BookPrice, head->BookName, head->BookNumber);
		flag = 1;
	}
	//如果不是空
	while (head->next != NULL)
	{
		head = head->next;
		if (BookNumber == head->BookNumber)
		{
			printf("书价\t书名\t书号\n");
			printf("%0.2f\t%s\t%d\n", head->BookPrice, head->BookName, head->BookNumber);
			flag = 1;
		}
	}

	if (flag == 0)
	{
		printf("查询失败!!\n");
	}
}
//修改
void ModifyNode(Node * head, char * BookName, float BookPrice)
{
	int flag = 0;
	float OldPrice;//历史价格
	if (head == NULL)
	{
		printf("head NULL!\n");
		exit(0);
	}
	if (strcmp(BookName, head->BookName) == 0)
	{
		OldPrice = head->BookPrice;//历史价格
		head->BookPrice = BookPrice;
		printf("成功修改价格!\n");
		printf("书号\t书名\t新书价\t历史价格\n");
		printf("%d\t%s\t%0.2f\t%0.2f\n", head->BookNumber, head->BookName, head->BookPrice,OldPrice);
		flag = 1;
	}
	while (head->next != NULL)
	{
		head = head->next;
		if (strcmp(BookName, head->BookName) == 0)
		{
			OldPrice = head->BookPrice;//历史价格
			head->BookPrice = BookPrice;
			printf("书号\t书名\t新书价\t历史价格\n");
			printf("%d\t%s\t%0.2f\t%0.2f\n", head->BookNumber, head->BookName, head->BookPrice, OldPrice);
			flag = 1;
		}
	}
	if (flag == 0)
	{
		printf("查询失败!!\n");
	}
}
//删除
Node * DeleteNode(Node * head, char * BookName)
{
	Node * pNode = NULL;
	if (head == NULL)
	{
		printf("head NULL!\n");
		exit(0);
	}
	if (strcmp(BookName, head->BookName) == 0)
	{
		if (head ->next != NULL)
		{
			pNode = head->next;
			free(head);
			return pNode;
		}
		else
		{
			printf("List null!\n");
			return NULL;
		}
	}
	if (strcmp(BookName, head->next->BookName) == 0)
	{
		if (head ->next ->next->BookName != NULL)
		{
			pNode = head->next->next;
			head->next = pNode;
			return head;
		}
	}
	while (head->next->next != NULL)
	{
		if (strcmp(BookName, head->next->next->BookName) == 0)
		{
			pNode = head->next->next->next;
			head->next->next = pNode;
			return head;
		}
	}
}
//查看全部
void AllNode(Node * head)
{
	printf("书号\t书名\t定价\n");
	while (head != NULL)
	{
		printf("%d\t%s\t%0.2f\n", head->BookNumber, head->BookName, head->BookPrice);
		head = head->next;
	}
}
```