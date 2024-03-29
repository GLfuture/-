线程绑定CPU

```

void *thread_func(void)
{
	printf("%d",sch)
}


int main()
{
	thread_t threads[4];
	cpu_set_t cpus;
	CPU_ZERO(&cpus)
	int i=0;
	for(i=0;i<4;i++)
	{
		CPU_SET(i,&cpus);
	}
	for(i=0;i<4;i++)
	{
		pthread_create(&thread[i],NULL,thread_fuc,&i);
		pthread_setaffinity_np(thread[i],sizeof(cpu_set_t),&cpus);//		先于thread_fuc执行,四个线程粘合到四个CPU
	}
}

```

