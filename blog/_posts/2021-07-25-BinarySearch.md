# 이진검색(BinarySearch)  
- 검색 원리상 정렬된 리스트에만 사용할 수 있다는 단점.
- 이진 탐색은 '퀵정렬'과 유사하게 '분할 후 정복(divide and conquer)'의 전략을 사용한다. 이 전략을 사용하는 알고리즘은 문제를 나누어 해결해 나가는 방법이기 때문에 실행시간은 log의 설징을 갖는다. 특히 문제의 크기를 정확히 양분하는 경우에는 최악의 경우라고 logN의 성능을 보장한다.
데이터의 순서가 오름차순으로 되어있기 때문에 왼쪽은 중앙요소보다 작고, 오른쪽은 큽니다. 검색하려는 데이터가 중앙값보다 크다면 오른편에서 중앙 요소를 다시 선택합니다. 그리고 다시 이진 탐색을 시작합니다.

```

public static void main(String[] args) {
    int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    System.out.println("binarySearch(4) : " + binarySearch(array, 4, array[0], array[array.length - 1]));
    System.out.println("---------------------------------------------");
    System.out.println("binarySearch(2) : " + binarySearch(array, 4));
}

public static int binarySearch(int array[], int value) {
    int low = 0;
    int high = array.length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        System.out.println("index value : " + mid);
        if (array[mid] > value)
            high = mid - 1;
        else if (array[mid] < value)
            low = mid + 1;
        else
            return mid; // found
    }
    return -1; // not found
}

public static int binarySearch(int array[], int value, int low, int high) {
    if (high <= low)
        return -1; // not found

    int mid = (low + high) / 2;

    System.out.println("index value : " + mid);

    if (array[mid] > value)
        return binarySearch(array, value, low, mid - 1);
    else if (array[mid] < value)
        return binarySearch(array, value, mid + 1, high);
    else
        return mid; // found
}


```