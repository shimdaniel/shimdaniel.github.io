# 순차검색(SequentialSearch)  
- 선형 검색 알고리즘(linear search algorithm)이라고도 하며, 특정한 값을 찾는 알고리즘의 하나다. 리스트에서 찾고자 하는 값을 맨 앞에서부터 끝까지 차례대로 찾아 나가는 것이다. 검색할 리스트의 길이가 길면 비효율적이지만, 검색 방법 중 가장 단순하여 구현이 쉽고 정렬되지 않은 리스트에서도 사용할 수 있다는 장점이 있다.  

```
public static int sequentialSearch(int[] arr, int key) {
    int arraySize = arr.length;
    for (int i = 0; i < arraySize; i++) {
        System.out.println(i+"번째 : "+arr[i]);
        if (arr[i] == key) {
            return i;
        }
    }
    return -1;
}

public static void main(String[] args) {
    int[] dataArray = {2, 80, 34, 3, 15, 77, 37, 12, 8, 3, 8};
    int searchKey = 12;
    sequentialSearch(dataArray, searchKey);
}
```