---
layout: post
title:  常见排序算法
categories:
- 算法
tags:
- sort

---

本文是前一段时间关于排序算法的一些讨论及基本实现思路。

## 冒泡算法(Bubble)

冒泡算法的计算步骤如下：

1. 比较相邻的两个元素，如果第一个比第二个大，就交换他们两个
2. 对每一对相邻的元素做（1）中的工作，从开始第一对到结尾的最后一对
3. 针对所有的元素重复以上步骤，除了最后一个
4. 每次将会越来越少的重复上面的步骤，知道没有任何一对数字需要比较为止

冒泡算法的简单实现：

	public static int[] sort(int[] array){
			boolean flag = false;
			int finished = 0;
			do{
				flag = false;
				int tmp;
				for(int i = 0;i<array.length-1-finished;i++){
					if(array[i]>array[i+1]){
						tmp = array[i];
						array[i] = array[i+1];
						array[i+1] = tmp;
						flag = true;
					}
				}
				finished = finished+1;
			}while(flag);
		
			return array;
	}

## 插入排序(Insertion)

1. 首先建立一个空的列表
2. 从原列表中取出一个数字，插入新建的列表中，保持新列表有序
3. 重复2号步骤，直至原来的列表为空

【7,&nbsp;&nbsp;  4,1,8,5,2,3,6,9】
【4,7,&nbsp;&nbsp;  1,8,5,2,3,6,9】
【1,4,7,&nbsp;&nbsp;  8,5,2,3,6,9】
【1,4,7,8,&nbsp;&nbsp;  5,2,3,6,9】

插入排序的基本实现，采用原地插入排序方式实现：

	public static int[] sort(int[] array){
		for(int i = 1; i<array.length;i++){
			for(int j = i;j>0;j--){
				if(array[j]<array[j-1]){
					int tmp = array[j-1];
					array[j-1] = array[j];
					array[j] = tmp;
				}else{
					break;
				}
			}
		}
		
		return array;
	}

## 选择排序(Selection)

1. 每次从未排序的元素中选出最小的一个，放在已经排序的元素的后面
2. 开始时，元素都未排序，从所有的元素中选择最小的一个，和下标为0的元素交换位置
3. 接下俩，从未排序的元素中选择最小的一个，和小标为1的元素交换位置。
4. 重复步骤3，直到倒数第二个元素

选择排序和上面的**插入排序**很类似，插入排序是插入是进行排序的操作，选择排序是选择时进行排序的操作，选择排序的基本实现如下：

	public static int[] sort(int[] array){
		for(int i = 0; i < array.length-1;i++){
			int tmp = array[i],index = i;
			for(int j = i;j < array.length;j++){
				if(array[j] < tmp){
					tmp = array[j];
					index = j;
				}
			}
			array[index] = array[i];
			array[i] = tmp;
		}
		return array;
	}
	

## 归并排序(Merge)

归并排序是建立在归并操作上的一种有效的排序算法,该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。

归并排序的基本实现，采用二路归并：

	public static int[] merge(int[] a, int[] b) {
		int[] res = new int[a.length + b.length];
		int i = 0, j = 0, k = 0;
		while (i < a.length && j < b.length) {
			if (a[i] <= b[j]) {
				res[k] = a[i];
				i++;
				k++;
			} else {
				res[k] = b[j];
				j++;
				k++;
			}
		}
		System.arraycopy(a, i, res, k, a.length - i);
		System.arraycopy(b, j, res, k+a.length - i, b.length - j);
		return res;
	}
	
	public static int[] sort(int[] array){
		if(array.length<=1){
			return array;
		}
		int middle = array.length/2;
		int[] left = new int[middle];
		int[] right = new int[array.length-middle];
		
		System.arraycopy(array, 0, left, 0, middle);
		System.arraycopy(array, middle, right, 0, array.length - middle);
		return merge(sort(left),sort(right));
	}
	
## 鸡尾酒排序(Cocktail)

鸡尾酒排序也就是定向冒泡排序,是冒泡排序的一种变形。此演算法与冒泡排序的不同处在于排序时是以双向在序列中进行排序。
数组中的数字本是无规律的排放，先找到最小的数字，把他放到第一位，然后找到最大的数字放到最后一位。然后再找到第二小的数字放到第二位，再找到第二大的数字放到倒数第二位。以此类推，直到完成排序。

	public static int[] sort(int[] array){
		int left_index = 0;
		int right_index = array.length-1;
		int tmp ;
		int current_index;
		while(left_index<=right_index){
			//把未排序中最大的移到右侧
			tmp = array[left_index];
			current_index = left_index;
			for(int i = left_index+1;i<=right_index;i++){
				if(array[i]>tmp){
					tmp = array[i];
					current_index= i;
				}
			}
			array[current_index] = array[right_index];
			array[right_index] = tmp;
			right_index--;
			
			//把未排序中最小的移到左侧
			tmp = array[right_index];
			current_index = right_index;
			for(int j = right_index-1;j>=left_index;j--){
				if(array[j]<tmp){
					tmp = array[j];
					current_index = j;
				}
			}
			array[current_index] = array[left_index];
			array[left_index] = tmp;
			left_index++;
			
		}
		return array;
	}
	
## 快速排序(Quick)

一趟快速排序的算法是：

1. 设置两个变量i、j，排序开始的时候：i=0，j=N-1；
2. 以第一个数组元素作为关键数据，赋值给key，即key=A[0]；
3. 从j开始向前搜索，即由后开始向前搜索(j--)，找到第一个小于key的值A[j]，将A[j]和A[i]互换；
4. 从i开始向后搜索，即由前开始向后搜索(i++)，找到第一个大于key的A[i]，将A[i]和A[j]互换；
5. 重复第3、4步，直到i=j； (3,4步中，没找到符合条件的值，即3中A[j]不小于key,4中A[i]不大于key的时候改变j、i的值，使得j=j-1，i=i+1，直至找到为止。找到符合条件的值，进行交换的时候i， j指针位置不变。另外，i==j这一过程一定正好是i+或j-完成的时候，此时令循环结束）。

快速排序的基本实现：

	public static void sort(int[] array,int left,int right){
		if(left>= right){
			return ;
		}
		int l = left,r = right;
		int key = array[left];
		while(l<r){
			while(l<r && key<=array[r]){
				r--;
			}
			array[l] = array[r];
			while(l < r && key>=array[l]){
				l++;
			}
			array[r] = array[l];
		}
		array[l] = key;
		sort(array,left,l-1);
		sort(array,l+1,right);
	}
	
## 计数排序(Counter)

计数排序法，时间复杂度为O(n)，以空间换时间。首先需要了解计数排序法的适用场景。：待排序元素分布比较集中，没有较大或者较小的异常值
 
 
1. 扫描待排序元素，确定其中的最大值和最小值. 以最大和最小值的间距长度申请一个临时数组。
2. 扫描待排序元素，利用临时数组对这些元素进行计数，计数方法为：以（元素 - 最小值）作为临时数组的下标，
3. 以（元素 - 最小值）出现的次数为对应的值。这样可以实现把待排序元素的顺序信息压缩到临时数组中。
4. 这样该临时数组中就存储了按照待排序元素的大小为数组下标，元素出现次数为值的数据。
5. 对临时数组进行累加：tmp[i] = tmp[i]+tmp[i-1]，这样得到的历史数组中就存储了待排序元素的顺序信息
6. 把利用待排序元素和临时数组，构建出已排序的数组

		public static int[] sort(int[] array){
			int[] res = new int[array.length];
		
			int max = array[0], min = array[0];
			for(int i : array){
				if(i > max){
					max = i;
				}
				if(i < min){
					min = i;
				}
			}
		
			int k = max - min + 1;
			//初始化的array每个元素默认值为0
			int[] tmp = new int[k];
		
			for(int i = 0; i < array.length;++i){
				//把array数组的元素压缩到tmp中，
				tmp[array[i] - min] +=1;
			}
			for(int i=1;i<tmp.length;++i){
				//tmp中的值是（下标+min）对应的元素在排序后的顺序值。
				tmp[i]=tmp[i]+tmp[i-1];
	        }
		
			for( int i = 0 ; i < array.length; ++i){
				res[--tmp[array[i] - min]] = array[i];
		//		tmp[array[i] - min] = tmp[array[i] - min]-1;
		//		res[tmp[array[i] - min]] = array[i];
			}
			return res;
		}

## 希尔排序(Shell)

希尔排序（缩小增量排序）算法描述如下：

1. 选择一个初始增量（一般为数组长度的一半）
2. 按照增量把数组元素分组，所有距离为d的倍数的数组放在同一个组，然后在各组内进行直接排序。
3. 取第二个增量(小于第一个增量)，重复进行分组和组内直接排序。直到增量值为1.


希尔排序的基本实现：


	public static int[] sort(int[] array){
		
		int d = array.length/2;
		while(d>=1){
			for(int i = 0; i< d;i++){
				//分组
				for(int j = i+d;j < array.length;j=j+d){
					//分组内的插入排序
					for(int k = j; k>0&&k-d>=0;k = k-d){
						if(array[k] < array[k-d]){
							int tmp = array[k-d];
							array[k-d] = array[k];
							array[k] = tmp;
						}else{
							break;
						}
					}
				}
			}
			d = d/2;
		}		
		return array;
	}
	

完成于2015-9-12，成渝动车D5129.