---
title : "정렬 알고리즘-BubbleSort(java)"
excerpt : "BubbleSort코드 "

categories:
  - Algorithm
tags :
  - Algorithm
  - java
---

### Bubble Sort 알고리즘

#### - 버블정렬 :  제일 작은걸 앞으로 하나씩 보내는 선택정렬과 달리, 매번 교체해줘야하기 때문에 더 오래걸리는 알고리즘

##### 10 + 9 + 8 + 7 +... + 1

##### 10 * (10 + 1) / 2 = 55 번 동작

##### => N * (N+1) / 2

 ##### 시간복잡도 : O(N^2) 



<pre>
public class BubbleSort {
	
	public static void main(String[] args) {
		int i,j,temp=0;
		int array[] = {2,5,9,1,7,3,4,10,8,6};
		
		for(i=0;i<10;i++) {
			for(j=0;j<9-i;j++) { //선택정렬과 다르게 뒤에가 점점 줄어들음(뒤는정렬되어있음)
				if(array[j]>array[j+1])//양옆비교
				{
					temp = array[j];
					array[j]=array[j+1];
					array[j+1]=temp;
				}
			}
		}
		for(int a=0;a<array.length;a++)
		{
			System.out.print(array[a]+" ");
		}
	}
}

</pre>



##### 실행 결과


![image-20200219210009599](https://user-images.githubusercontent.com/53978090/74832527-dbeb1480-535a-11ea-8a59-ffe4af551abf.png)
