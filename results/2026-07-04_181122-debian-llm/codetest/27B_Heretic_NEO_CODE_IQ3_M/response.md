```python
class LRUCache:
    class _Node:
        __slots__ = ('key', 'value', 'prev', 'next')
        def __init__(self, key=None, value=None):
            self.key = key
            self.value = value
            self.prev = None
            self.next = None

    def __init__(self, capacity: int):
        self._capacity = capacity
        self._cache = {}
        self._head = self._Node()  # dummy head (MRU side)
        self._tail = self._Node()  # dummy tail (LRU side)
        self._head.next = self._tail
        self._tail.prev = self._head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_front(self, node):
        node.next = self._head.next
        node.prev = self._head
        self._head.next.prev = node
        self._head.next = node

    def _move_to_front(self, node):
        self._remove(node)
        self._add_to_front(node)

    def _pop_tail(self):
        node = self._tail.prev
        self._remove(node)
        return node

    def get(self, key):
        node = self._cache.get(key)
        if node is None:
            return None
        self._move_to_front(node)
        return node.value

    def put(self, key, value):
        node = self._cache.get(key)
        if node is not None:
            node.value = value
            self._move_to_front(node)
        else:
            if len(self._cache) == self._capacity:
                evict = self._pop_tail()
                del self._cache[evict.key]
            new_node = self._Node(key, value)
            self._cache[key] = new_node
            self._add_to_front(new_node)

    def peek(self, key):
        node = self._cache.get(key)
        return node.value if node is not None else None

    def __len__(self):
        return len(self._cache)
```
