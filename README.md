#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#define _CRT_SECURE_NO_WARNINGS
#define MAX_STUDENTS 100
#define NAME_LENGTH 50
#define MAJOR_LENGTH 50
#define BIRTHDAY_LENGTH 20
#define DORMITORY_LENGTH 20

// 学生信息结构体
typedef struct Student
{
    int id;
    char name[NAME_LENGTH];
    char major[MAJOR_LENGTH];
    char birthday[BIRTHDAY_LENGTH];
    char gender[NAME_LENGTH];
    char dormitory[DORMITORY_LENGTH];
} Student;

// 线性表结构体
typedef struct 
{
    Student data[MAX_STUDENTS];
    int length;
} SeqList;

// 引入链表结点
typedef struct Node
{
    Student data;
    struct Node* next;
} Node, * LinkedList;

// 栈结构体(用于操作记录)
typedef struct 
{
    Student data[MAX_STUDENTS];
    int top;
} Stack;

// 全局变量
SeqList seqList;        // 线性表存储学生信息
LinkedList linkedList = NULL; // 链表存储学生信息
Stack operationStack;   // 活动操作栈（用于撤销操作）

// 初始化数据结构
void initDataStructures() 
{
    seqList.length = 0;
    linkedList = NULL;
    operationStack.top = -1;
}

// 检查线性表中ID是否已存在，避免重复操作
bool isIdExist(int id) 
{
    for (int i = 0; i < seqList.length; i++)
    {
        if (seqList.data[i].id == id) return true;
    }
    return false;
}

// 添加学生信息到线性表，表长加1
void addToSeqList(Student student) 
{
    if (seqList.length >= MAX_STUDENTS) 
    {
        printf("线性表已满，无法添加更多学生。\n");
        return;
    }
    seqList.data[seqList.length++] = student;
}

// 添加学生信息到链表
void addToLinkedList(Student student)
{
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = student;
    newNode->next = NULL;
    if (linkedList == NULL)
    {
        linkedList = newNode;
    }
    else 
    {
        Node* current = linkedList;
        while (current->next != NULL)
        {
            current = current->next;
        }
        current->next = newNode;
    }
}

// 压栈操作(记录最近添加的学生)
void pushToStack(Student student) {
    if (operationStack.top >= MAX_STUDENTS - 1)
    {
        printf("操作栈已满，无法记录更多操作。\n");
        return;
    }
    operationStack.data[++operationStack.top] = student;
}

// 从线性表删除学生
void deleteFromSeqList(int id) 
{
    for (int i = 0; i < seqList.length; i++) 
    {
        if (seqList.data[i].id == id) 
        {
            // 记录被删除的学生到栈中，便于撤销操作
            pushToStack(seqList.data[i]);
            // 移动元素补位
            for (int j = i; j < seqList.length - 1; j++) 
            {
                seqList.data[j] = seqList.data[j + 1];
            }
            seqList.length--;
            printf("已从线性表中删除学生ID: %d\n", id);
            return;
        }
    }
    printf("未在线性表中找到学生ID: %d\n", id);
}

// 从链表删除学生
void deleteFromLinkedList(int id)
{
    Node* current = linkedList;
    Node* previous = NULL;
    while (current != NULL)
    {
        if (current->data.id == id)
        {
            // 记录被删除的学生到栈中，便于撤销操作
            pushToStack(current->data);

            if (previous == NULL)
            {
                linkedList = current->next;
            }
            else {
                previous->next = current->next;
            }
            free(current);
            printf("已从链表中删除学生ID: %d\n", id);
            return;
        }
        previous = current;
        current = current->next;
    }
    printf("未在链表中找到学生ID: %d\n", id);
}

// 撤销上一次删除操作（调用栈）
void undoDelete()
{
    if (operationStack.top < 0) 
    {
        printf("没有可撤销的操作。\n");
        return;
    }
    Student student = operationStack.data[operationStack.top--];
    if (isIdExist(student.id)) 
    {
        printf("ID %d 已存在，无法恢复\n", student.id);
        return;
    }
    addToSeqList(student);
    addToLinkedList(student);
    printf("已撤销删除操作，恢复学生ID: %d\n", student.id);
}

// 显示学生信息
void displayStudent(Student student) 
{
    printf("\n=== 学生详细信息 ===\n");
    printf("学生ID: %d\n", student.id);
    printf("学生姓名: %s\n", student.name);
    printf("学生专业: %s\n", student.major);
    printf("学生出生日期: %s\n", student.birthday);
    printf("学生性别: %s\n", student.gender);
    printf("学生宿舍信息: %s\n", student.dormitory);
}

// 添加学生信息
void addStudent() 
{
    Student student;

    printf("请输入学生ID: ");
    if (scanf_s("%d", &student.id) != 1)
    {
        printf("输入无效的ID\n");
        while (getchar() != '\n'); // 清空输入缓冲区
        return;
    }
    if (isIdExist(student.id)) 
    {
        printf("该ID已存在\n");
        return;
    }
    printf("请输入学生姓名: ");
    scanf_s("%s", student.name, NAME_LENGTH);
    printf("请输入学生专业: ");
    scanf_s("%s", student.major, MAJOR_LENGTH);
    printf("请输入学生出生日期: ");
    scanf_s("%s", student.birthday, BIRTHDAY_LENGTH);
    printf("请输入学生性别: ");
    scanf_s("%s", student.gender, NAME_LENGTH);
    printf("请输入学生宿舍信息: ");
    scanf_s("%s", student.dormitory, DORMITORY_LENGTH);

    addToSeqList(student);
    addToLinkedList(student);
    printf("学生信息添加成功。\n");
}
// 删除学生信息
void deleteStudent()
{
    int id;
    printf("请输入要删除的学生的ID: ");
    if (scanf_s("%d", &id) != 1) {
        printf("输入无效的ID\n");
        while (getchar() != '\n'); // 清空输入缓冲区
        return;
    }
    deleteFromSeqList(id);
    deleteFromLinkedList(id);
}
// 同步链表数据
void syncLinkedList(int id, Student newData)
{
    Node* current = linkedList;
    while (current != NULL) 
    {
        if (current->data.id == id)
        {
            current->data = newData;
            break;
        }
        current = current->next;
    }
}
//选择性修改学生信息
void modifyStudent() 
{
    int id;
    printf("请输入要修改的学生的ID: ");
    if (scanf_s("%d", &id) != 1) 
    {
        printf("输入无效的ID\n");
        while (getchar() != '\n');
        return;
    }
    int index = -1;
    for (int i = 0; i < seqList.length; i++)
    {
        if (seqList.data[i].id == id)
        {
            index = i;
            break;
        }
    }
    if (index == -1)
    {
        printf("未找到该学生信息\n");
        return;
    }
    Student original = seqList.data[index];
    Student modified = original;
    while (1) 
    {
        displayStudent(modified);
        printf("\n请选择要修改的信息:\n");
        printf("1. 姓名\n2. 专业\n3. 出生日期\n");
        printf("4. 性别\n5. 宿舍信息\n6. 完成修改\n");
        printf("请输入选择: ");
        int choice;
        if (scanf_s("%d", &choice) != 1)
        {
            while (getchar() != '\n');
            printf("无效输入\n");
            continue;
        }
        switch (choice) 
        {
        case 1:
            printf("新姓名: ");
            scanf_s("%s", modified.name, NAME_LENGTH);
            break;
        case 2:
            printf("新专业: ");
            scanf_s("%s", modified.major, MAJOR_LENGTH);
            break;
        case 3:
            printf("新生日: ");
            scanf_s("%s", modified.birthday, BIRTHDAY_LENGTH);
            break;
        case 4:
            printf("新性别: ");
            scanf_s("%s", modified.gender, NAME_LENGTH);
            break;
        case 5:
            printf("新宿舍: ");
            scanf_s("%s", modified.dormitory, DORMITORY_LENGTH);
            break;
        case 6:
            seqList.data[index] = modified;
            syncLinkedList(id, modified);
            printf("修改已保存\n");
            return;
        default:
            printf("无效选项\n");
        }
    }
}
// 查询学生信息
void searchStudent() 
{
    int id;
    printf("请输入要查询的学生ID: ");
    if (scanf_s("%d", &id) != 1)
    {
        printf("输入无效的ID\n");
        while (getchar() != '\n');
        return;
    }
    // 从线性表查询
    for (int i = 0; i < seqList.length; i++) 
    {
        if (seqList.data[i].id == id)
        {
            displayStudent(seqList.data[i]);
            return;
        }
    }
    printf("未找到该学生\n");
}
// 显示所有学生信息
void displayAllStudents() 
{
    printf("\n=== 线性表数据 ===\n");
    printf("%-8s%-20s%-20s%-12s%-8s%-15s\n", "ID", "姓名", "专业", "生日", "性别", "宿舍");
    for (int i = 0; i < seqList.length; i++) 
    {
        printf("%-8d%-20s%-20s%-12s%-8s%-15s\n",
            seqList.data[i].id,
            seqList.data[i].name,
            seqList.data[i].major,
            seqList.data[i].birthday,
            seqList.data[i].gender,
            seqList.data[i].dormitory);
    }
    printf("\n=== 链表数据 ===\n");
    printf("%-8s%-20s%-20s%-12s%-8s%-15s\n", "ID", "姓名", "专业", "生日", "性别", "宿舍");
    Node* current = linkedList;
    while (current != NULL) 
    {
        printf("%-8d%-20s%-20s%-12s%-8s%-15s\n",
            current->data.id,
            current->data.name,
            current->data.major,
            current->data.birthday,
            current->data.gender,
            current->data.dormitory);
            current = current->next;
    }
}
// 释放链表内存
void freeLinkedList()
{
    Node* current = linkedList;
    while (current != NULL) {
        Node* temp = current;
        current = current->next;
        free(temp);
    }
    linkedList = NULL;
}
// 主菜单
void mainMenu() 
{
    while (1) 
    {
        printf("\n学生管理系统\n");
        printf("1. 添加学生\n2. 删除学生\n3. 修改学生\n");
        printf("4. 查询学生\n5. 显示全部\n6. 撤销操作\n7. 退出\n");
        printf("请选择操作: ");
        int choice;
        if (scanf_s("%d", &choice) != 1)//无效输入
        {
            while (getchar() != '\n');
            printf("无效输入\n");
            continue;
        }
        switch (choice) 
        {
        case 1: addStudent(); break;
        case 2: deleteStudent(); break;
        case 3: modifyStudent(); break;
        case 4: searchStudent(); break;
        case 5: displayAllStudents(); break;
        case 6: undoDelete(); break;
        case 7:
            freeLinkedList();
            printf("系统已退出\n");
            exit(0);
        default:
            printf("无效选择，请重新输入\n");
        }
    }
}
int main()
{
    initDataStructures();
    mainMenu();
    return 0;
}
