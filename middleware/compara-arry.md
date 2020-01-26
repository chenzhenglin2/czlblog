**比较两个数组是否相同（忽略元素位置） java语言实现**

	public static boolean comparaarry(int[] arr1,int[] arr2) {
		if(arr1.length != arr2.length) {
			return false;
		}else {
			Arrays.sort(arr1);
			Arrays.sort(arr2);
			int wcount = 0;
			for (int i=0;i<arr1.length;i++) {
				if (arr1[i]==arr2[i]) {
					wcount++;
				}
			}
			if(wcount==arr1.length) {
				return true;
			}else {
				return false;
			}
			
		}
		
	}

方法很简单，就是先判断长度，如果长度相同，进行排序，然后逐个比较，每一个元素相同，计数器就自增一次，最后比较计数器和数组长度是否一致，如果一致，则两个数组相同。