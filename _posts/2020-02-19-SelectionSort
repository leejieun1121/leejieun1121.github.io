---
title : "정렬 알고리즘-SelectionSort(java)"
excerpt : "SelectionSrot 코드"

categories:
  - Algorithm
tags :
  - Algorithm
  - Java

---

### Selection Sort 알고리즘

#### - 선택정렬 : 작은것을 찾아 맨앞으로 보내는 알고리즘

##### 10 + 9 + 8 + 7 +... + 1

##### 10 * (10 + 1) / 2 = 55 번 동작

##### => N * (N+1) / 2

 ##### 시간복잡도 : O(N^2)

<pre>
    public class SelectionSort {
	public static void main(String[] args) {
		int array[] = {2,5,9,1,7,3,4,10,8,6};
		int i,j,min,temp;
		int index=0;
		

		for(i=0;i<10;i++) { //기준이되는애
			min = 9999;
			for(j=i;j<10;j++) {//앞에가 점점 줄어들음 끝까지돌면서 비교하는애(앞은 정렬되어있음)
				if(min>array[j])
				{
					min = array[j];
					index = j;
				}
			}
			temp = array[i];
			array[i]=array[index];
			array[index]=temp;			
		}
		for(int a=0;a<array.length;a++)
		{
			System.out.print(array[a]+" ");
		}
	}

}
</pre>



##### 실행 결과

![image-20200219205017319](https://user-images.githubusercontent.com/53978090/74831919-8eba7300-5359-11ea-9b32-a4d2a130e037.png)

