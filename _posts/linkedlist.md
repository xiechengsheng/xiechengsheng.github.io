---
layout:     post
title:      "链表面试题"
subtitle:   "链表"
date:       "2017-04-07"
author:     "xiechengsheng"
header-img: "img/post-bg-5.jpg"
catalog: true
tags:
    - interview
---

记录链表的一些面试题目：    

# 反转单链表：
四步：
1. 保留当前指针的下一节点
2. 将上一指针赋值给当前指针下一节点
3. 将当前指针赋值给上一个指针
4. 将下一个指针复制给当前指针
```cpp
    ListNode* reverseList(ListNode* head) {
    // 1 保存head下一节点
    // 2 将head所指向的下一节点改为prev
    // 3 将prev替换为head，波浪式前进
    // 4 将第一步保存的下一节点替换为head，用于下一次循环

        ListNode* prev=NULL;
        while(head) {
            ListNode* temp=head->next;
            head->next=prev;
            prev=head;
            head=temp;
        }
        return prev;
    }
```

# two points: fast and slow
T234：判断一个链表是否是回文链表    
1. 找到链表的中点（判断条件是一个fast、一个slow指针，有时候是需要fast&&fast->next，有时候是fast&&fast->next&&fast->next->next）
2. 链表中点后面的链表部分翻转
3. 从链表头到中点后面的部分依次判断
```cpp
    bool isPalindrome(ListNode* head) {
        if(!head||!head->next)
            return true;
        //1.找到链表中点
        ListNode *slow=head,*fast=head;
        while(fast&&fast->next) {
            fast=fast->next->next;
            slow=slow->next;
        }
        ListNode *middle=slow;

        //2.翻转中点后面的半个链表部分，以middle为分界线
        ListNode *boundary=middle;
        ListNode *prev=NULL;

        while(boundary){
            ListNode *temp=boundary->next;
            boundary->next=prev;
            prev=boundary;
            boundary=temp;
        }
        //假如原来的链表是1->2->3->4->5->6->NULL
        //在这里交换操作完毕之后的两条链表是:1->2->3->4->NULL，6->5->4->NULL

        //3.前半个链表和后半个链表依次比对
        ListNode *begin=head;
        while(begin!=middle){
            if(begin->val==prev->val){
                begin=begin->next;
                prev=prev->next;
            }
            else
                return false;
        }
        return true;
    }
```

T143：按照L(0)->L(n)->L(1)->L(n-1)的方式返回链表：   
三步，和上面T234的情况一样
```cpp
    void reorderList(ListNode* head) {
        if(!head||!head->next)
            return;
        //1.找到链表中点
        ListNode *slow=head,*fast=head;
        while(fast&&fast->next) {
            fast=fast->next->next;
            slow=slow->next;
        }
        ListNode *middle=slow;

        //2.翻转中点后面的半个链表部分，以middle为分界线
        ListNode *boundary=middle;
        ListNode *prev=NULL;

        while(boundary){
            ListNode *temp=boundary->next;
            boundary->next=prev;
            prev=boundary;
            boundary=temp;
        }

        //3.前半个链表和后半个链表依次配对
        ListNode *begin=head;
        while(begin!=middle){
            ListNode *temp1=begin->next, *temp2=prev->next;
            begin->next=prev;
            prev->next=temp1;
            begin=temp1;
            prev=temp2;
        }
        //这里的情况是如果链表长度为偶数，prev->next将指向temp1，但此时temp1就是prev自身
        //实际的情况是如果链表长度是偶数，进行到这里已经到达链表尾部，因此直接写成prev->next=NULL;
        //不然就返回没有尾结点的链表，必须加上尾结点=NULL
        begin->next=NULL;
        return;
    }
```

# dummy head node
T24：链表结点两两交换：   
这里的链表头结点会发生改变，因此以后链表头结点会发生变化的题目最好使用dummy指向原来的头结点，返回dummy->next，还是链表交换的结果
```cpp
    ListNode* swapPairs(ListNode* head) {
        if(!head||!head->next)
            return head;
        //这里的头结点是改变了位置的，所以最后返回链表头结点的时候是错的，需要新建一个指向原来头结点的结点，即dummy
        //然后再在新结点的后面两两交换结点
        ListNode *dummy=new ListNode(-1);
        dummy->next=head;
        ListNode *prev=dummy;
        ListNode *curr=head;
        while(curr&&curr->next){
            ListNode *next=curr->next;

            curr->next=next->next;
            next->next=curr;

            prev->next=next;
            prev=curr;

            curr=curr->next;
            //next=curr->next;
        }
        return dummy->next;
    }
```

T82：移除掉链表中所有的重复的结点（有点难想到）    
```cpp
    ListNode* deleteDuplicates(ListNode* head) {
        if(!head||!head->next)
            return head;
        //又是可能会改变头结点的题目，所以需要造一个假结点
        ListNode *dummy = new ListNode(-1);
        ListNode *prev=dummy;
        prev->next=head;
        ListNode *curr=head;
        while(curr&&curr->next){
            while(curr&&curr->next&&curr->val==curr->next->val){
                curr=curr->next;
            }
            if(prev->next==curr){
                prev=prev->next;
            }
            //这个else里面其实是在不断地替换prev的next，直到遇到不重复的数为止
            else{
                prev->next=curr->next;
            }
            curr=curr->next;
        }
        //返回的其实是一大串prev构成的链表
        return dummy->next;
    }
```

T21：合并两个已经排好序的链表：   
可能会改变头结点，需要造一个假结点
```cpp
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *dummy=new ListNode(-1);
        ListNode *prev=dummy;
        while(l1&&l2){
            if(l1->val>l2->val) {
                prev->next=l2;
                prev=l2;
                l2=l2->next;
            }
            else {
                prev->next=l1;
                prev=l1;
                l1=l1->next;
            }
        }
        //这里直接把剩下的链表拼接到prev后面就好
        prev->next=l1?l1:l2;
        return dummy->next;
    }
```

# 其他
T445. Add Two Numbers II：这次是链表的顺序求和    
```cpp
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        // 栈 存储
        stack<int> s1, s2;
        while(l1) {
            s1.push(l1->val);
            l1=l1->next;
        }
        while(l2) {
            s2.push(l2->val);
            l2=l2->next;
        }
        
        //计算，这里使用tail=NULL代替尾部节点，因此不需要翻转链表
        int carry=0;    //进位
        ListNode *tail = NULL, *curr = NULL;
        
        while(!s1.empty()||!s2.empty()) {
            int num1, num2;
            if(s1.empty()) {
                num1=0;
            }
            else {
                num1=s1.top();
                s1.pop();
            }    
            if(s2.empty()) {
                num2=0;
            }
            else {
                num2=s2.top();
                s2.pop();
            }    
            
            int sum = num1+num2+carry;
            carry = sum/10;
            int value = sum % 10;
            curr = new ListNode(value);
            curr->next = tail;
            tail = curr;
        }
        if(carry) {
            curr = new ListNode(carry);
            curr->next=tail;
        }
        return curr;
    }
```

T160：判断两个链表的交点：    
方法1：简单粗暴，首先得到两个链表的长度，之后遍历长度更长的链表，到两个链表长度相等的时候，依次遍历两链表，找到相同的结点即为答案    
```cpp
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(!headA||!headB)
            return NULL;
        int lenA=getLength(headA),lenB=getLength(headB);
        ListNode *nodeA=headA,*nodeB=headB;
        while(lenA>lenB){
            nodeA=nodeA->next;
            lenA--;
        }
        while(lenB>lenA){
            nodeB=nodeB->next;
            lenB--;
        }
        //不同考虑指针是否会为空的问题，因为nodeA为空的时候nodeB也为空（长度相同），因此不会进行循环
        while(nodeA!=nodeB){
            nodeA=nodeA->next;
            nodeB=nodeB->next;
        }
        return nodeA;
    }

    int getLength(ListNode *head){
        int length=0;
        while(head){
            head=head->next;
            length++;
        }
        return length;
    }
```

解法2：把两个链表看成两个跑道，两个跑道拼接起来构成一个环，假设A的跑道长度是5，B的跑道长度是3，A的跑道的第四个结点和B的跑道第二个结点相同，就这样绕着跑道环跑，在一圈之内A和B必然将到达相同状态的结点。   
![将两个链表构成一个圆环](/img/in-post/linkedlist/getIntersectionNode.png)
```cpp
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode *nodeA=headA,*nodeB=headB;
        while(nodeA!=nodeB){
            nodeA=nodeA?nodeA->next:headB;
            nodeB=nodeB?nodeB->next:headA;
        }
        return nodeA;
    }
```

# 总结：    
多刷题，没啥好说的......
