---
title: "Ukkonen's suffix tree construction - Part 6"
layout: post
comments: true
date: 2017-03-01
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

앞선 [Part 1][Part1], [Part 2][Part2], [Part 3][Part3], [Part 4][Part4]에서는 suffix tree의 기본 개념들, 고수준 Ukkonen 알고리즘, suffix link와 세 가지 구현 트릭, activePoint에 대한 몇 가지 세부 사항들, 그리고 예제 "abcabxabcd"에 대해서 6개의 phase를 거치며 suffix tree를 만들어 보았다.

그리고 이번 **Part 6**에서 Ukkonen's suffix tree construction algorithm을 python으로 구현함으로써 마치겠다. (원 글에서는 c++ 구현이 제시되어 있다.)

먼저 Suffix Tree의 Node를 나타내는 SuffixTreeNode 클래스를 정의한다.

```python
class SuffixTreeNode:
	def __init__(self, start=None, end=None, suffixLink=None):
		self.children = dict()
		self.start = start
		self.end = end
		self.suffixLink = suffixLink
		self.suffixIndex = -1
		self.leafCount = 0

	def has_outgoing_edge_starting_with(self, character):
		return character in self.children

	def is_root(self):
		return self.start == -1 and self.end == -1

	def is_leaf(self):
		# return self.suffixIndex != -1
		return len(self.children) == 0

	def get_edge_length(self):
		return self.end - self.start + 1

	def get_child(self, edge):
		return self.children[edge]

	def get_start_position(self):
		return self.start

	def get_end_position(self):
		return self.end

	def set_end_position(self, end):
		self.end = end

	def get_children(self):
		return self.children.values()

	def get_children_lexicographically(self):
		return [value for key, value in sorted(self.children.items(), key=lambda x: x[0])]

	def set_start_position(self, newStart):
		self.start = newStart

	def get_character_index_on_edge(self, edge, distance):
		return self.children[edge].get_start_position() + distance - 1

	def set_suffix_link_to(self, targetNode):
		self.suffixLink = targetNode

	def follow_suffix_link(self):
		return self.suffixLink

	def set_suffix_index(self, suffixIndex):
		self.suffixIndex = suffixIndex

	def get_suffix_index(self):
		return self.suffixIndex

	def set_child(self, edge, node):
		self.children[edge] = node

	def set_new_child(self, edge, start, end):
		child = SuffixTreeNode(start, end)
		self.children[edge] = child
```

그리고 Ukkonen 알고리즘을 구현한다.

```python

from SuffixTreeNode import SuffixTreeNode

class SuffixTree:
  def __init__(self, string):
    self.root = SuffixTreeNode(start=-1, end=-1, suffixLink=None)
    self.string = string + '$'

    self.activeNode = self.root
    self.activeEdge = None
    self.activeLength = 0

    self.END = EndValue()
    self.REMAINING_SUFFIX_COUNT = RemainingSuffixCount()

    self.lastInternalNode = None

    self._construct_suffix_tree()
    self._set_suffix_index_by_dfs(self.root, labelLength=0)

  def _construct_suffix_tree(self):
    for position, character in enumerate(self.string):
      self.END.increment()
      self.REMAINING_SUFFIX_COUNT.increment()

      while self.REMAINING_SUFFIX_COUNT > 0:
        if self.activeLength == 0:
          self._active_point_change_for_active_length_zero(position)

        activeEdgeCharacter = self.string[self.activeEdge]

        if self.activeNode.has_outgoing_edge_starting_with(activeEdgeCharacter):
          walkDownDone = self._active_point_change_for_walk_down(currNode=self.activeNode.get_child(activeEdgeCharacter))
          if walkDownDone:
            continue

          nextCharacterIndex = self.activeNode.get_character_index_on_edge(edge=activeEdgeCharacter, distance=self.activeLength+1)
          nextCharacter = self.string[nextCharacterIndex]
          
          if nextCharacter == character:
            if self.lastInternalNode:
              self.lastInternalNode.set_suffix_link_to(self.activeNode)
              self.lastInternalNode = None

            # Rule 3 applied, terminate the phase immediately
            self._active_point_change_for_extension_rule_3()
            break
          else:
            # Rule 2 applied
            self._split(activeEdgeCharacter, character, position)

        else:
          self.activeNode.set_new_child(edge=activeEdgeCharacter, start=position, end=self.END)

          if self.lastInternalNode:
            self.lastInternalNode.set_suffix_link_to(self.activeNode)
            self.lastInternalNode = None

        self.REMAINING_SUFFIX_COUNT.decrement()

        if self.activeNode.is_root() and self.activeLength > 0:
          self._active_point_change_for_extension_rule_2_case_1(position)
        elif not self.activeNode.is_root():
          self._active_point_change_for_extension_rule_2_case_2()

  def _split(self, activeEdgeCharacter, character, position):
    splitEndIndex = self.activeNode.get_character_index_on_edge(edge=activeEdgeCharacter, distance=self.activeLength)
    nextNode = self.activeNode.get_child(edge=activeEdgeCharacter)

    # Note that suffix link of an internal node is initially set to the root
    newNode = SuffixTreeNode(start=nextNode.get_start_position(), end=splitEndIndex, suffixLink=self.root)
    self.activeNode.set_child(edge=activeEdgeCharacter, node=newNode)
    
    # Insert new node
    nextNode.set_start_position(nextNode.get_start_position() + self.activeLength)
    newNode.set_new_child(edge=character, start=position, end=self.END)
    newNode.set_child(edge=self.string[nextNode.get_start_position()], node=nextNode)

    # Set suffix link of the previous internal node to new node
    if self.lastInternalNode:
      self.lastInternalNode.set_suffix_link_to(newNode)

    self.lastInternalNode = newNode

  def _get_active_point(self):
    return (self.activeNode, self.string[self.activeEdge], self.activeLength)

  def _active_point_change_for_extension_rule_3(self):
    self.activeLength += 1

  def _active_point_change_for_walk_down(self, currNode):
    if self.activeLength < currNode.get_edge_length():
      return False  # we don't need walk down

    self.activeEdge += currNode.get_edge_length()
    self.activeLength -= currNode.get_edge_length()
    self.activeNode = currNode
    return True

  def _active_point_change_for_active_length_zero(self, position):
    self.activeEdge = position

  def _active_point_change_for_extension_rule_2_case_1(self, position):
    self.activeLength -= 1
    self.activeEdge = position - self.REMAINING_SUFFIX_COUNT + 1

  def _active_point_change_for_extension_rule_2_case_2(self):
    self.activeNode = self.activeNode.follow_suffix_link()

  def _set_suffix_index_by_dfs(self, node, labelLength):
    if node.is_leaf():
      node.set_suffix_index(len(self.string) - labelLength)
      return 1

    else:
      leafCount = 0
      for child in node.get_children():
        newLabelLength = labelLength + child.get_edge_length()
        leafCount += self._set_suffix_index_by_dfs(node=child, labelLength=newLabelLength)

      node.set_leaf_count(leafCount)
      return leafCount


def singleton(class_):
  '''http://stackoverflow.com/questions/6760685/creating-a-singleton-in-python'''
  instances = {}
  def getinstance(*args, **kwargs):
    if class_ not in instances:
      instances[class_] = class_(*args, **kwargs)
    return instances[class_]
  return getinstance


class GlobalIntValue:
  def __init__(self):
    self.value = 0

  def __str__(self):
    return str(self.value)

  def __add__(self, other):
    return self.value + other

  def __radd__(self, other):
    return self.value + other

  def __sub__(self, other):
    return self.value - other

  def __rsub__(self, other):
    return other - self.value

  def __gt__(self, other):
    return self.value > other

  def decrement(self):
    self.value -= 1

  def increment(self):
    self.value += 1


@singleton
class EndValue(GlobalIntValue):
  def __init__(self):
    self.value = -1


@singleton
class RemainingSuffixCount(GlobalIntValue):
  def __iter__(self):
    for i in range(self.value):
      yield i
```

### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-6/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-6/
[2]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/
[Part1]: ../Ukkonen's-suffix-tree-construction-Part-1
[Part2]: ../Ukkonen's-suffix-tree-construction-Part-2
[Part3]: ../Ukkonen's-suffix-tree-construction-Part-3
[Part4]: ../Ukkonen's-suffix-tree-construction-Part-4
[Part5]: ../Ukkonen's-suffix-tree-construction-Part-5
[Part6]: ../Ukkonen's-suffix-tree-construction-Part-6