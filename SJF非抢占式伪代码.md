# OS-homework
//	 SJF算法	-抢占式、非抢占式

//	个人觉得的难点：判断某个时间点是否有进程到达。如果有，判断其所需的执行时间是否满足服务条件。

//	非抢占式：如果最新到达的进程所需执行时间 在已经到达的进程中 为最小，则等待当前正在执行的进程执行完毕后，再执行。---> 需要保存下一个执行的进程。

//	 抢占式：如果最新到达的进程所需时间最短，则抢占。需要对剩下的进程重新排序？

// 定义结构体

typedef struct node{

    int pid;
	
    int waiteTime;
	
    int needTime;
	
    int arrivalTime;
	
    int finish;
	
} PCB;


int total =0;	//	总的执行时间

PCB PCBS[n];	// n: 总的进程数

PCB next;	//	用于保存下一个执行的进程


// 输入进程id、执行时间、到达时间


Sort(PCBS);		//根据进程到达时间进行升序排序

for(int i =0;i<n;i++)	// 初始化进程等待时间，完成情况

{

    PCBS[i].waiteTime = 0;
	
    PCBS[i].finish = 0;
	
}


//	 非抢占式.已经按照到达时间排完序。

//	如果到达时间相同---按照执行时间排序

//	如果p2 比 p1所需时间短，但是p1 比 p2 先到 ---- p1先执行


//	开始进程

void startProcess()

{

    int firstArrivalTime = PCBS[0].arrivalTime;	//	第一个到达的进程时间
	
    int nextIndex = 0;	//	下一个执行的进程的下标
    
    while(1)
	
    {
	
        total ++;
		
        if(total < firstArrivalTime)	//	第一个进程没有到
		
        //do nothing
		
        
        else if(total == firstArrivalTime)	//	第一个进程到达，开始执行
		
        {
		
            runProcess(0);
			
        }
        else	//	第一个进程执行完毕，开始下一个进程
		
        {
		
            nextIndex = selectNext();
			
            if(nextIndex != -1)		//存在下一个进程
			
            {
			
                runProcess(nextIndex);
				
            }
			
            else break;
			
        }
		
    }
	
}

// 执行进程
void runPorcess(int run){

    int need = 0;
    
    while( !PCBS[run].finish )	//程序未执行完
    
    {
    
        total++;
        
        PCBS[run].needTime --;
        
        need = PCBS[run].needTime;
        
        if(need == 0)	//	 进程运行结束
        
        {
        
            PCBS[run].finish = 1;
            
            break;
            
        }
        
    }
    
}

// 选择下一个进程：处于等待状态且时间最短

int selectNext(){

    int shortest = 100;
    
    int next = 0;
    
    for(int i = 0;i<n;i++)
    
    {
    
        if(PCBS[i].finish==0 && PCBS[i].arrivalTime >total)	//	未完成且已经到达
        
        {
        
            if(PCBS[i].needTime <shortest)
            
            {
            
                next = i;
                
                shortest = PCBS[i].needTime;
                
            }
            
        }
        
    }
    
    return next;
    
}



