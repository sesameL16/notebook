

```java
public class MyTest3 {

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool(3);
        MyTask myTask = new MyTask(0,1000000);
        Integer result = forkJoinPool.invoke(myTask);
        System.out.println(result);
        forkJoinPool.shutdown();
    }

}

class MyTask extends RecursiveTask<Integer>{

    private int limit = 1000;

    private int firstIndex;

    private int lastIndex;

    public MyTask(int firstIndex, int lastIndex) {
        this.firstIndex = firstIndex;
        this.lastIndex = lastIndex;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        int gap = lastIndex - firstIndex;
        boolean flag = gap <= limit;

        if (flag){
            System.out.println(Thread.currentThread().getName());
            for (int i = firstIndex; i <= lastIndex; i++){
                result = result + i;
            }
        }else {
            int middleIndex = (firstIndex + lastIndex)/2;
            MyTask leftTask = new MyTask(firstIndex,middleIndex );
            MyTask rightTask = new MyTask(middleIndex + 1,lastIndex );
            invokeAll(leftTask,rightTask );

            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            result = leftResult + rightResult;
        }
        return result;
    }
}
```