---
title : "정렬 알고리즘-InsertionSort(java)"
excerpt : "InsertionSort코드 "

categories:
  - Algorithm
tags :
  - Algorithm
  - java
---

### InsertionSort 알고리즘

#### - 삽입정렬 :  선택정렬과 버블정렬은 모든경우에 위치를 바꾸는 반면, 삽입정렬은 필요할때만 위치를 바꾼다

##### 10 + 9 + 8 + 7 +... + 1

##### 10 * (10 + 1) / 2 = 55 번 동작

##### => N * (N+1) / 2

##### 시간복잡도 : O(N^2) 

##### 버블,삽입,선택중 삽입정렬이 가장 빠르며 거의 정렬된 상태라면 가장 효율적인 알고리즘!

##### 앞 배열은 이미 정렬이 되어있음, 적절한 위치를 찾아 하나씩 앞으로 가면서 바꿈


<pre>
public class InsertionSort {
	public static void main(String[] args) {
		int i,j,temp=0;
		int array[] = {1,10,5,8,7,6,4,3,2,9}; 
		//15810764329
		//15871064329 15781064329
		//15786104329 15768104329 15678104329
		//15678410329 15674810329 15647810329 15467810329 14567810329
		//14567831029 14567381029 14563781029 14536781029 14356781029 13456781029 ... 
		for(i=0;i<9;i++) {
			j = i;
			while(array[j]>array[j+1]) {
				temp = array[j];
				array[j] = array[j+1];
				array[j+1] = temp;
				j--;
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

![image-20200219211013647](https://user-images.githubusercontent.com/53978090/74833186-51a3b000-535c-11ea-8ac3-b0e8e452ebef.png)
