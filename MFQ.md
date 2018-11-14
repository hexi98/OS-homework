`#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
typedef struct node   /*进程节点信息*/  
{  
char name[20];   /*进程的名字*/  
int prio;    /*进程的优先级*/  
int round;    /*分配CPU的时间片*/ 
int arrivaltime; 
int cputime;   /*CPU执行时间*/  
int needtime;   /*进程执行所需要的时间*/  
int waitetime; /*等待时间*/
char state;    /*进程的状态，W——就绪态，R——执行态，F——完成态*/  
int count;    /*记录执行的次数*/  
int len;
int sta[4];	/* 保存进程在各个队列运行的时间*/
int singal;
int turnOverTime;
struct node *next;  /*链表指针*/  
}PCB;  

typedef struct Queue  /*多级就绪队列节点信息*/  
{  
PCB *LinkPCB;   /*就绪队列中的进程队列指针*/  
int prio;    /*本就绪队列的优先级*/  
int round;    /*本就绪队列所分配的时间片*/  
struct Queue *next;  /*指向下一个就绪队列的链表指针*/  
}ReadyQueue;  

PCB *wait = NULL, *run=NULL,*finish=NULL; /*定义三个队列，就绪队列，执行队列和完成队列*/  
ReadyQueue *Head = NULL; /*定义第一个就绪队列*/  
int num;     /*进程个数*/  
int ReadyNum;    /*就绪队列个数*/
int allTime =0;  
int t_time=0;
int w_time = 0;
double weightTurnOverTime = 0.0;
PCB tmp_array[4];

void Output();          /*进程信息输出函数*/  
void InsertFinish(PCB *in);       /*将进程插入到完成队列尾部*/  
void InsertPrio(ReadyQueue *in);     /*创建就绪队列，规定优先数越小，优先级越低*/  
void PrioCreate();         /*创建就绪队列输入函数*/  
void GetFirst(ReadyQueue *queue);     /*取得某一个就绪队列中的队头进程*/  
void InsertLast(PCB *in,ReadyQueue *queue);   /*将进程插入到就绪队列尾部*/  
void ProcessCreate();        /*进程创建函数*/  
void RoundRun(ReadyQueue *timechip);    /*时间片轮转调度算法*/  
void MultiDispatch();        /*多级调度算法，每次执行一个时间片*/  

int main(void)  
{  
PrioCreate(); /*创建就绪队列*/  
ProcessCreate();/*创建就绪进程队列*/  
MultiDispatch();/*算法开始*/  
Output();  /*输出最终的调度序列*/  
return 0;  
}  


void outPutGantt(PCB *run){
  allTime += run->len;
}
void Output()  /*进程信息输出函数*/  
{  
ReadyQueue *print = Head;  
PCB *p;  
printf("-----------------------------------------------------------------");
printf("\n进程名\tcpu时间\t进程状态\t计数器\t等待时间\t周转时间\n");  

p = finish;  
while(p!=NULL)  
{  
  printf("%s\t%d\t%c\t\t%d\t",p->name,p->cputime,p->state,p->count);  
  for(int i = 0;i<3;i++)
  {
	if(i == p->singal)
	{
	  printf("%d\t\t%d\n",tmp_array[i].waitetime,tmp_array[i].waitetime + p->cputime);
	  w_time += tmp_array[i].waitetime;
	  t_time += tmp_array[i].waitetime + p->cputime;
	  weightTurnOverTime += (tmp_array[i].waitetime + p->cputime)*1.0/(p->cputime);
	}

  }
  p = p->next;  
}  

printf("平均等待时间:%0.2f\n",w_time*1.0/3);
printf("平均周转时间:%0.2f\n",t_time*1.0/3);
printf("平均带权周转时间:%0.2f\n",weightTurnOverTime*1.0/3);



}  
void InsertFinish(PCB *in)  /*将进程插入到完成队列尾部*/  
{  
PCB *fst;  
fst = finish;  

if(finish == NULL)  
{  
  in->next = finish;  
  finish = in;  
}  
else  
{  
  while(fst->next != NULL)  
  {  
   fst = fst->next;  
  }  
  in ->next = fst ->next;  
  fst ->next = in;  
}  
}  
void InsertPrio(ReadyQueue *in)  /*创建就绪队列，规定优先数越小，优先级越低*/  
{  
ReadyQueue *fst,*nxt; //  fst 頭 nxt 尾
fst = nxt = Head;  

if(Head == NULL)    /*如果没有队列，则为第一个元素*/  
{  
  in->next = Head;  
  Head = in;  
}  
else       /*查到合适的位置进行插入*/  
{  
  if(in ->prio >= fst ->prio)  /*比第一个还要大，则插入到队头*/  
  {  
   in->next = Head;  
   Head = in;  
  }  
  else  
  {  
   while(fst->next != NULL)  /*移动指针查找第一个别它小的元素的位置进行插入*/  
   {  
    nxt = fst;  
    fst = fst->next;  
   }  

   if(fst ->next == NULL)  /*已经搜索到队尾，则其优先级数最小，将其插入到队尾即可*/  
   {  
    in ->next = fst ->next;  
    fst ->next = in;  
   }  
   else     /*插入到队列中*/  
   {  
    nxt = in;  
    in ->next = fst;  
   }  
  }  
}  
}  

void PrioCreate() /*创建就绪队列输入函数*/  
{  
ReadyQueue *tmp;  //就绪队列的链表指针
int i;  
ReadyNum = 3;  //就绪队列的个数

for(i = 0;i < ReadyNum; i++)  
{  
  if((tmp = (ReadyQueue *)malloc(sizeof(ReadyQueue)))==NULL)  
  {  
   perror("malloc");  
   exit(1);  
  }  
  (tmp->round) = i+8;  /*此就绪队列中给每个进程所分配的CPU时间片*/ 
  tmp ->prio = 10 - tmp->round;  /*设置其优先级，时间片越高，其优先级越低*/  
  tmp ->LinkPCB = NULL;    /*初始化其连接的进程队列为空*/  
  tmp ->next = NULL;  
  InsertPrio(tmp);     /*按照优先级从高到低，建立多个就绪队列*/  
}  
}  

void GetFirst(ReadyQueue *queue)     /*取得某一个就绪队列中的队头进程*/  
{  
run = queue ->LinkPCB;  

if(queue ->LinkPCB != NULL)  
{  
  run ->state = 'R';  
  queue ->LinkPCB = queue ->LinkPCB ->next;  
  run ->next = NULL;  
}  
}  

void InsertLast(PCB *in,ReadyQueue *queue)  /*将进程插入到就绪队列尾部*/  
{  
PCB *fst;  
fst = queue->LinkPCB;  

if( queue->LinkPCB == NULL)  
{  
  in->next =  queue->LinkPCB;  
  queue->LinkPCB = in;  
}  
else  
{  
  while(fst->next != NULL)  
  {  
   fst = fst->next;  
  }  
  in ->next = fst ->next;  
  fst ->next = in;  
}  
}  
PCB tmp_array[4];
void ProcessCreate() /*进程创建函数*/  
{   
PCB *tmp; 

int i;  
freopen("input.txt","r",stdin);
printf("----------------------------MFQ--------------------------\n");  

num = 3;
printf("进程名字\t到达时间\t进程所需时间\t标识符\n"); 
for(i = 0;i < num; i++)  
{  
  if((tmp = (PCB *)malloc(sizeof(PCB)))==NULL)  
  {  
   perror("malloc");  
   exit(1);  
  }  
  scanf("%s",tmp->name); 
  scanf("%d",&(tmp->arrivaltime)); 
  scanf("%d",&(tmp->needtime));
  tmp_array[i].needtime = tmp->needtime;
  tmp ->cputime = 0;  
  tmp ->state ='W';  
  tmp ->prio = 25 - tmp->needtime;  /*设置其优先级，需要的时间越多，优先级越低*/  
  tmp ->round = Head ->round;  
  tmp ->count = 0;  
  tmp->singal = i;

  printf("%s\t\t%d\t\t%d\t\t%d\n",tmp->name,tmp->arrivaltime,tmp->needtime,tmp->singal);
  InsertLast(tmp,Head);      /*按照优先级从高到低，插入到就绪队列*/  
} 

}  

void RoundRun(ReadyQueue *timechip)    /*时间片轮转调度算法*/  
{  

int flag = 1;  

GetFirst(timechip);  
while(run != NULL)  
{  
  printf("|");
  while(flag)  
  {   
   printf("_");

   run->count++;  
   run->len = run->count;
   run->cputime++;  
   run->needtime--;
     for(int i = 0;i<3;i++)
   {
	   if(i == run->singal)
		tmp_array[i].needtime --;
   }  
   for(int i =0;i<3;i++)
   { 
	if(i!=run->singal && tmp_array[i].needtime!=0)
           tmp_array[i].waitetime++;
   }
   if(run->needtime == 0) /*进程执行完毕*/  
   {  
    run->state = 'F';  
    InsertFinish(run);  
    flag = 0;  
   }  
   else if(run->count == timechip ->round)/*时间片用完*/  
   {  
    run->state = 'W';  
    run->count = 0;   /*计数器清零，为下次做准备*/  
    InsertLast(run,timechip);  
    flag = 0;  
   }  
  }  
  flag = 1;  
  allTime += run->len;
  GetFirst(timechip);  
}  
}  
;
void MultiDispatch()   /*多级调度算法，每次执行一个时间片*/  
{  
int flag = 1;  
int k = 0;  

ReadyQueue *point;  
point = Head;  
printf("-------------------------Gantt chart-------------------------\n");
GetFirst(point);  

while(run != NULL)  
{  
  printf("|");
  if(Head ->LinkPCB!=NULL)  
   point = Head;  
  while(flag)  
  {  

   printf("_");
   run->count++; 
   run->sta[run->singal]++;
   run->len = run->count; 
   run->cputime++;  
   run->needtime--;  
  // printf("singal:%d\n",run->singal);
     for(int i = 0;i<3;i++)
   {
	   if(i == run->singal)
		tmp_array[i].needtime --;
   }  
   for(int i =0;i<3;i++)
   {
        //printf("needtime%d:%d\n",i,tmp_array[i].needtime);
	if(i!=run->singal && tmp_array[i].needtime!=0)
           tmp_array[i].waitetime++;
   }
   if(run->needtime == 0) /*进程执行完毕*/  
   {  
    run ->state = 'F';  
    InsertFinish(run);  
    flag = 0;  
   }  
   else if(run->count == run->round)/*时间片用完*/  
   {  
    run->state = 'W';  
    run->count = 0;   /*计数器清零，为下次做准备*/  

    if(point ->next!=NULL)  
    {  
     run ->round = point->next ->round;/*设置其时间片是下一个就绪队列的时间片*/ 
     InsertLast(run,point->next);  /*将进程插入到下一个就绪队列中*/  
     flag = 0;  
    }
    else  
    {  
     RoundRun(point);   /*如果为最后一个就绪队列就调用时间片轮转算法*/  
     break;  
    }  
   }  

  }  
  flag = 1;  
  if(point ->LinkPCB == NULL)/*就绪队列指针下移*/  
 {   
  point =point->next;  
  printf("|\n\n");//Gantt 换行
}
  if(point ->next ==NULL)  
  {  
   allTime += run->len;
   RoundRun(point);  
   printf("|\n\n");
   break;  
  }  
  allTime += run->len;
  GetFirst(point);  
}  
}   `