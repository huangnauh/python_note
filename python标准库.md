python标准库学习

heapq
class TopKHeap(object):
        def __init__(self,k):
                self.k = k
                self.data = []
        def push(self,item):
                if len(self.data) < self.k:
                        heapq.heappush(self.data,item)
                else:
                        heapq.heappushpop(self.data,item)
        def topK(self):
                self.data.sort(reverse=True)
                return self.data

class BtmKHeap(object):
        def __init__(self,k):
                self.k = k
                self.data = []
        def push(self,item):
                item = -item
                if len(self.data) < self.k:
                        heapq.heappush(self.data,item)
                else:
                        heapq.heappushpop(self.data,item)
        def btmK(self):
                return sorted(-x for x in self.data)