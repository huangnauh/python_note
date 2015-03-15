第一节 选择排序

```
void select_sort(int a[],int n){
    for(int i=0;i<n;i++){
        int min = i;
        for(int j=i+1;j<n;j++){
            if(a[min] > a[j])
                min = j;
        }
        if(min != i){
            int temp = a[min];
            a[min] = a[i];
            a[i] = temp;
        }
    }
}
```

```
def select_sort(a):
    n = len(a)
    for i in range(n):
        min = i
        for j in range(i+1,n):
            if a[min] > a[j]:
                min = j
        if min > i :
            a[min],a[i] = a[i],a[min]
```

第二节 插入排序
```
void insert_sort(int a[],int n){
    for(int i=1;i<n;i++){
        int j=i;
        int temp = a[i];
        while(j>0 && a[j-1]>temp){
            a[j] = a[j-1];
            j--;
        }
        a[j] = temp;
    }
}
```

```
def insert_sort(a):
    n = len(a)
    for i in range(1,n):
        j = i
        temp = a[i]
        while j>0 and a[j-1]>temp:
            a[j] = a[j-1]
            j -= 1
        a[j] = temp
```


第三节 冒泡排序
用flag来记录一次冒泡中最后一次交换的位置
```
void bubble_sort(int a[],int n){
    int flag = n-1;
    while(flag){
        int i = flag;
        flag = 0;
        for(int j=0;j<i;j++){
            if(a[j] > a[j+1]){
                int temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
                flag = j;
            }
        }
    }
}
```

```
def bubble_sort(a):
    n = len(a)
    flag = n-1
    while flag:
        i = flag
        flag = 0
        for j in range(i):
            if a[j] > a[j+1]:
                a[j],a[j+1] = a[j+1],a[j]
                flag = j
```

第四节 希尔排序
适当的选取gap

```
void shell_sort(int a[],int n){
    for(int gap=n/2;gap>0;gap/=2){
        for(int i=gap;i<n;i++){
            int temp = a[i];
            int j = i;
            while(j>gap-1 && a[j-gap]>temp){
                a[j] = a[j-gap];
                j -= gap;
            }
            a[j] = temp;
        }
    }
}
```

```
def shell_sort(a):
    n = len(a)
    gap = n//2
    while gap:
        for i in range(gap,n):
            j = i
            temp = a[i]
            while j>gap-1 and a[j-gap]>temp:
                a[j] = a[j-gap]
                j -= gap
            a[j] = temp
        gap = gap // 2
```

第五节 归并排序
每次merge过程中都需要一个的额外存储空间，为了防止内存的频繁申请和释放，在最外层申请内存storage作为参数传递给merge函数
```
void merge_with_storage(int target[],int mid,int n,int storage[]){
    int i=0;
    int j=mid;
    int s=0;
    while(i<mid && j<n){
        if(target[i]<=target[j]){
            storage[s] = target[i];
            i++;
        }
        else{
            storage[s] = target[j];
            j++;
        }
        s++;
    }
    while(i<mid){
        storage[s++] = target[i++];
    }
    while(j<n){
        storage[s++] = target[j++];
    }
    memcpy(target,storage,sizeof(int)*n);
}

void merge_sort_with_storage(int a[],int n,int storage[]){
    if(n<=1){
        return;
    }
    int i = n/2;
    merge_sort_with_storage(a,i,storage);
    merge_sort_with_storage(a+i,n-i,storage);
    merge_with_storage(a,i,n,storage);
}

void merge_sort(int a[],int n){
    int *storage = malloc(sizeof(int)*n);
    merge_sort_with_storage(a,n,storage);
    free(storage);
}
```

python递归版本
```
def merge(left,right):
    merged = []
    while left and right:
        merged.append(left.pop(0) if left[0] <= right[0] else right.pop(0))
    while left:
        merged.append(left.pop(0))
    while right:
        merged.append(right.pop(0))
    return merged
```

```
def merge_sort(a):
    n = len(a)
    if n<= 1:
        return a
    mid = n//2
    return merge(merge_sort(a[:mid]),merge_sort(a[mid:]))
```

python迭代版本
def iter_mergesort(a):
    lst = [[i] for i in a]
    while len(lst)>1:
        lst.append(merge(lst.pop(0),lst.pop(0)))
    return lst.pop()
    
python利用闭包实现就地排序
```
def merge_sort(a):
    n = len(a)
    if n<=1:
        return
    mid = n//2
    left = a[:mid]
    right = a[mid:]
    merge_sort(left)
    merge_sort(right)
    def merge(left,right):
        i = j = k = 0
        l = len(left)
        r = len(right)
        while j<l and k<r:
            if left[j] <= right[k]:
                a[i] = left[j]
                j+=1
            else:
                a[i] = right[k]
                k+=1
            i += 1
        while j<l:
            a[i] = left[j]
            i += 1
            j += 1
        while k<r:
            a[i] = right[k]
            i += 1
            k += 1
    merge(left,right)
```
第六节 快速排序
```
int partition(int a[],int n){
    int p = a[0];
    int first = 1;
    int last = n-1;
    int done = 1;
    while(done){
        while(first<=last && a[first]<=p){
            first++;
        }
        while(last>=first && a[last]>p){
            last--;
        }
        if(first>last){
            done = 0;
        }
        else{
            int temp = a[first];
            a[first] = a[last];
            a[last] = temp;
            first++;
            last--;
        }
    }
    a[0] = a[last];
    a[last] = p;
    return last;
}

void q_sort(int a[],int n){
    if(n<=1){
        return;
    }
    int q = partition(a,n);
    q_sort(a,q);
    q_sort(a+q+1,n-q-1);
}
```

```
def q_sort(a):
    def q_sort_helper(a,first,last):
        if first<last:
            def partition(a,first,last):
                done = 0
                left = first+1
                right = last
                while not done:
                    while left<=right and a[left] <= a[first]:
                        left += 1
                    while right>=left and a[right] > a[first]:
                        right -= 1
                    if left > right:
                        done = 1
                    else:
                        a[left],a[right] = a[right],a[left]
                        left += 1
                        right -= 1
                a[first],a[right] = a[right],a[first]
                return right
            q = partition(a,first,last)
            q_sort_helper(a,first,q-1)
            q_sort_helper(a,q+1,last)
    q_sort_helper(a,0,len(a)-1)
```

第七节 堆排序
升序排列，需要建立最大堆

```
void max_heap_fixdown(int a[],int i,int n){
    int parent = i;
    int child = 2*i+1;
    int temp = a[parent];
    while(child < n){
        if(child+1<n && a[child+1]>a[child]){
            child = child+1;
        }
        if(a[child]<=temp){
            break;
        }
        a[parent] = a[child];
        parent = child;
        child = 2*parent+1;
    }
    a[parent] = temp;
}
void make_max_heap(int a[],int n){
    for(int i=(n-2)/2;i>=0;i--){
        max_heap_fixdown(a,i,n);
    }
}

void max_heap_sort(int a[],int n){
    for(int i=n-1;i>0;i--){
        int temp = a[0];
        a[0] = a[i];
        a[i] = temp;
        max_heap_fixdown(a,0,i);
    }
}

void heap_sort(int a[],int n){
    make_max_heap(a,n);
    max_heap_sort(a,n);
}
```

python标准库有 [heapq模块](https://hg.python.org/cpython/file/2.7/Lib/heapq.py)

Python的timsort [Understanding timsort](http://www.drmaciver.com/2010/01/understanding-timsort-1adaptive-mergesort/)


选择排序
```
function selectSort(a){
    var n = a.length;
    for(var i=0;i<n;i++){
        var min = i;
        for(var j=i+1;j<n;j++){
            if(a[min] > a[j]){
                min = j;
            }
        }
        if(min != i){
            var temp = a[i];
            a[i] = a[min];
            a[min] = temp;
        }
    }
}
```
插入排序
```
function insertSort(a){
    var n = a.length;
    for(var i=1;i<n;i++){
        var temp = a[i];
        var j = i;
        while(j>0 && temp<a[j-1]){
            a[j] = a[j-1];
            j--;
        }
        a[j] = temp;
    }
}
```
冒泡排序
```
function bubbleSort(a){
    var n = a.length;
    var flag = n-1;
    while(flag){
        var i = flag;
        flag = 0;
        for(var j=0;j<i;j++){
            if(a[j] > a[j+1]){
                var temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
                flag = j;
            }
        }
    }
}
```
希尔排序
```
function shellSort(a){
    var n = a.length;
    var gap = Math.floor(n/2);
    while(gap){
        for(var i=gap;i<n;i+=gap){
            var j=i;
            var temp = a[i];
            while(j>0 && temp<a[j-gap]){
                a[j] = a[j-gap];
                j -= gap;
            }
            a[j]=temp;
        }
        gap = Math.floor(gap/2);
    }
}
```
归并排序

递归版本
```
Array.prototype.mergeSort = function(){
    var merge = function(left,right){
        var merged = [];
        while(left.length && right.length){
            merged.push(left[0] <= right[0] ? left.shift():right.shift());
        }
        return merged.concat(left.concat(right));
    }
    if (this.length < 2) return this;
    var mid = Math.floor(this.length/2);
    return merge(this.slice(0,mid).mergeSort(),this.slice(mid).mergeSort())
}

```

迭代版本
```
Array.prototype.mergeSort = function(){
    var merge = function(left,right){
        var merged = [];
        while(left.length && right.length){
            merged.push(left[0] <= right[0] ? left.shift():right.shift());
        }
        return merged.concat(left.concat(right));
    }
    var myarray = [];
    this.forEach(function(item){
        myarray.push([item]);
    });
    while(myarray.length > 1){
       myarray.push(merge(myarray.shift(),myarray.shift()))
    }
    return myarray[0];
}

```



为了使数组原地排序，构造splice方法的参数,对数组进行先删后增。
```
function merge(left,right){
    var result = [],
        l = left.length,
        r = right.length,
        il = 0,
        ir = 0;
    while(il < l && ir < r){
        if(left[il] <= right[ir]){
            result.push(left[il++]);
        }else{
            result.push(right[ir++]);
        }
    }
    return result.concat(left.slice(il)).concat(right.slice(ir));
}

function mergeSort(a){
    var n = a.length;
    if(n < 2){
        return a;
    }
    var mid = Math.floor(n/2),
        left = a.slice(0,mid),
        right = a.slice(mid);
    var params = merge(mergeSort(left),mergeSort(right));
    params.unshift(0,n);
    a.splice.apply(a,params)
    return a;
}
```




也可以使用闭包来实现就地排序
```
function mergeSort(a){
    var n = a.length;
    if(n <= 1){
        return;
    }
    var mid = Math.floor(n/2);
    function merge(left,right){
        var l = left.length;
        var r = right.length;
        var i=0,j=0,k=0;
        while(i<l && j<r){
            if(left[i] <= right[j]) {
                a[k++] = left[i++];
            }else{
                a[k++] = right[j++];
            }
        }
        while(i<l){
            a[k++] = left[i++];
        }
        while(j<r){
            a[k++] = right[j++];
        }
    }
    var left = a.slice(0,mid);
    var right = a.slice(mid,n);
    mergeSort(left);
    mergeSort(right);
    merge(left,right);
}
```



快速排序
选择数组中间的值作为基准
```
function partition(a,first,last){
    var p = Math.floor((first+last)/2),
        left = first,
        right = last,
        temp;
    while(left <= right){
        while(left <= right && a[left] < a[p]){
            left++;
        }
        while(left <= right && a[right] > a[p]){
            right--;
        }
        if(left > right){
            break;
        }
        temp = a[left];
        a[left] = a[right];
        a[right] = temp;
        left++;
        right--;
    }
    return left;
}

function quickSortHelp(a,first,last){
    if(first >= last){
        return;
    }
    var p = partition(a,first,last);
    quickSortHelp(a,first,p-1);
    quickSortHelp(a,p,last);
}

function quickSort(a){
    quickSortHelp(a,0,a.length-1);    
}
```
选择数组开始的值作为基准
```
function partition(a,first,last){
    var p = first;
    left = first+1;
    right = last;
    var temp;
    while(left <= right){
        while(left <= right && a[left] <= a[p]){
            left++;
        }
        while(left <= right && a[right] > a[p]){
            right--;
        }
        if(left > right){
            break;
        }
        temp = a[left];
        a[left] = a[right];
        a[right] = temp;
        left++;
        right--;
   }
   temp = a[right];
   a[right] = a[first];
   a[first] = temp;
   return right;
}

function quickSortHelp(a,first,last){
    if(first >= last){
        return;
    }
    var p = partition(a,first,last);
    quickSortHelp(a,first,p-1);
    quickSortHelp(a,p+1,last);
}

function quickSort(a){
    quickSortHelp(a,0,a.length-1);    
}
```

因为Javascript语言允许省略参数
quickSortHelp和quickSort可以合并为一个函数，以上面的函数为例
```
function quickSort(a,first,last){
    if (a.length < 2) return a;
    first = (typeof first === "number" ? first : 0);
    last = (typeof last === "number" ? last : a.length-1);
    var p = partition(a,first,last);
    if (first < p-1){
        quickSort(a,first,p-1);
    }
    if(p+1 < last){
        quickSort(a,p+1,last);
    }
}
```