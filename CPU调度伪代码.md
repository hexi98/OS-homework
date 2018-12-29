# OS-homework

1. FCFS

// 定义结构体

Typedef struct node{

	int pid;

	int arrivalTime;

	int needTime;

	int waitTime;

	int finish;	// 是否已经完成

}PCB



Int totalTime = 0;	//总的运行时间。用于确认进程是否到达



PCB PCBS[n];	//一共有n个进程



// 输入进程id、到达时间、需要时间



for(int i =0; i< n; I++)	//	初始化进程

{

	PCBS[I].waitTime = 0;

 	PCBS[I].finish = 0;

}



Sort(PCBS);	// 根据到达时间，对进程进行升序排序



for(int I=0; I<n ; I++)

{

	while( !PCB[I].finish )	//若该进程未执行完，则继续执行

	{

		PCBS[I].needTime —;

		totalTime ++;

		for( int j = 0;j<n; j++)

		{

			//如果进程j不是当前正在执行的进程

			//该进程没有完成，且已经到达，则等待时间++

			if( j!=I && PCBS[j].finish==0 && total>= PCBS[j].arrivalTime)

				PCBS[j].waitTime ++;

		}

		if(PCBS[I].needTime ==0)	//	进程执行完毕

		{ 

			PCB[I].finish == 1;

			continue;

		}

	}

}
