---
layout:     post
title:      对于c++中有关iterator的学习以及几个std方法的理解
category: blog
description: 通过在网上查找实现技能buff系统的资料，复习了有关iterator的概念，同时学习到了几个std方法
---

**以下仅是我在目前的能力下对相关知识的理解，也许有很多地方存在谬误。以后我会不断的修正。**
**因为不清楚一些c++中的概念词的中文译名，所以暂时以英文单词取代。**

## 起因
最近一直在想如何更好的解决有关人物移动速度buff的恢复。
在StatckExchange找到了一篇相关的[帖子](http://gamedev.stackexchange.com/questions/46772/how-to-implement-buffs-debuffs-temporary-stat-changes-in-an-rpg)。虽然有关的问题我已经有了想法（这个稍后再写）,里面提到的有关在遍历技能列表的方法，移除超过技能使用时间的方法令我学习到了std里面的几个函数，同时复习了std::iterator的知识。

## skill buff相关代码

首先是buff的Struct

    struct Buff
    {
        enum Type
        {
            Type_BonusMalus,
            Type_Multiplier,
        };
    
        Type type;
        int value[STAT_COUNT];
        float timeLeft; // FLT_INFINITY = permanent
    
        void updateAndApply(float dt, int* stats)
        {
            for (int i=0; i<STAT_COUNT; ++i)
            {
                switch (type)
                {
                    case Type_BonusMalus: stats[i] += value[i]; break;
                    case Type_Multiplier: stats[i] *= value[i]; break;
                }
            }
            if (timeLeft != FLT_INFINITY)
            {
                timeLeft -= dt;
            }
        }
    
        bool hasTimedOut()
        {
            return (timeLeft <= 0.0f);
        }
    };

然后是更新和使用buff

    void updateAndApplyBuffs(float dt, int* stats)
    {
        // set base stats
        setToBase(stats);
    
        std::list<Buff>::iterator it;
    
        // apply multipliers first
        for (it = buffList.begin(); it != buffList.end(); ++it)
        {
            if (it->type == Buff::Type_Multiplier)
                it->updateAndApply(dt, stats);
        }
    
        // then apply bonuses/maluses
        for (it = buffList.begin(); it != buffList.end(); ++it)
        {
            if (it->type == Buff::Type_BonusMalus)
                it->updateAndApply(dt, stats);
        }
    
        // remove buffs that timed out
        // SuperCool® STL stolen from http://stackoverflow.com/a/596708/1005455
        buffList.remove_if(std::mem_fun(&Buff::hasTimedOut));
    }
最后一句我不明白

    buffList.remove_if(std::mem_fun(&Buff::hasTimedOut));

根据作者的介绍，他是援引于另外一个[提问的解答][].

## 另一个提问
这个提问问的问题很类似，既是在update列表项的同时，删除update过的item。

提问者给出的代码是：

    for (std::list<item*>::iterator i=items.begin();i!=items.end();i++)
    {
        bool isActive = (*i)->update();
        //if (!isActive) 
        //  items.remove(*i); 
        //else
           other_code_involving(*i);
    }
    items.remove_if(CheckItemNotActive);
而这会导致i++时出现报错：

> "List iterator not incrementable" 

#### 我理解的原因

首先，就像std的作用是使人们可以方便的处理各种类型的数据一样，itorator的作用是遍历container里的所有数据，而无需指定数据类型（type）。所以不同的container的itorator所指向的对象是不一样的。

>有的是object，有的是pointer。【C++ Primer Plus，P995】

*疑惑：对于itorator作为pointer我能理解，但是作为object的形式我还不明白。

通过debug的时候查看itor得知，在List中，itor是一个pointer, 指向一个包含prevLink, nextLink, 和value的Node。在list.erase(itor)之后，这个Node失去了作为itor的功能。（至于变成了什么，还不懂。但是通过在watch里的观察，它此时的值与list.end()一致。）而list.erase(itor)本身所return的，是下一个element的地址。

所以有：

    std::list<int> intList;
	for (int i = 0; i < 10; ++i)
	{
		intList.push_back(i);
	}

	for (std::list<int>::iterator i = intList.begin(); i != intList.end(); ++i)
	{
		if (*itor % 2 == 1)
		{
			intList.erase(itor++);
			// or itor = intList.erase(itor);
		}
		else
			++itor;
	}

而其中的 

    intList.erase(itor++);
正是利用了 int++ 首先copy一份（valuation），返回copy之后再对目标加1的特性(伪代码):

    copyObj = *this;
    *this += 1;
    return copyObj;
**（所以，普遍认为在c++中，一般情况下使用++int比较有效率。）**

但是需要注意的是，上面的代码还应该改为while loop, 不然每次会跳过一个item。 而且使用itor = intList.erase(itor) 更为安全。因为如果把List改为Vector的话，intList.erase(itor++) 会导致 "iterator is incompatible" 的错误:

但是在对vector的测试中，即使是intList.erase(itor++)也会出错("iterator is incompatible")：

	std::list<int> intList;
	std::vector<int> intVector;
	for (int i = 0; i < 10; ++i)
	{
		intVector.push_back(i);
	}

	std::Vector<int>::iterator itor = intVector.begin();
	while (itor != intVector.end())
	{
		if (*itor % 2 == 1)
		{
			itor = intVector.erase(itor);
			intVector.erase(itor++);
		}
		else
			++itor;
	}
至于出现"iterator is incompatible"错误的原因，下面有人提到是因为：
Vector使用array来组织信息，在其中一个item被移除的时候，它会将后续的items向左移动，填补空间。 
但是我不明白的是，根据debug观察，当itor=1时，

	if (*itor % 2 == 1)
		{
			intVector.erase(itor++);
		}
移除1之后，itor的值的确变成了3（因为2向左移，同时itor++，所以跳过了2）。那么情况因该与for loop中类似，每次循环跳过一个item才对，为什么会出错呢？

####debug中对于内存地址的观察
itor==1的时候，[ptr]:0x004aab74{1}。

![Alt text](/images/blog/cpp_iterator_001.jpg)

执行intVector.erase(itor++)之后，[ptr]0x004aab78{3}。此时，[ptr]:0x004aab74应该成为了2。

![Alt text](/images/blog/cpp_iterator_002.jpg)

**注意上图中，_Myproxy变为nullprt。**

此时，如果是使用whie loop。返回while loop 的比较阶段，进行比较时，出错
![Alt text](/images/blog/cpp_iterator_003.jpg)

如果是使用for loop，返回for loop的迭代阶段，进行++itor时，出错
![Alt text](/images/blog/cpp_iterator_004.jpg)

* 疑惑: Vector中使用 intList.erase(itor++) 出现"iterator is incompatible"错误的原因可能是因为itor指向的结构发生了变化？

##对于“如何删除升级后的技能”的解答
对于这个如何在update之后马上从技能列表中删除此技能的问题，[Mike][]给出的解答是：

    // Note: Using the pre-increment operator is preferred     for iterators because
    //       there can be a performance gain.
    //
    // Note: As long as you are iterating from beginning to     end, without inserting
    //       along the way you can safely save end once;     otherwise get it at the
    //       top of each loop.
    
    std::list< item * >::iterator iter = items.begin();
    std::list< item * >::iterator end  = items.end();
    
    while (iter != items.end())
    {
        item * pItem = *iter;
    
        if (pItem->update() == true)
        {
            other_code_involving(pItem);
            ++iter;
        }
        else
        {
            // BTW, who is deleting pItem, a.k.a. (*iter)?
            iter = items.erase(iter);
        }
    }
但是解答者还给出了更好的解决方案，既是本文要记录的有关std的知识点：

    // This implementation of update executes     other_code_involving(Item *) if
    // this instance needs updating.
    //
    // This method returns true if this still needs future     updates.
    //
    bool Item::update(void)
    {
        if (m_needsUpdates == true)
        {
            m_needsUpdates = other_code_involving(this);
        }
    
        return (m_needsUpdates);
    }
    
    // This call does everything the previous loop did!!!     (Including the fact
    // that it isn't deleting the items that are erased!)
    items.remove_if(std::not1(std::mem_fun(&Item::update)));
## 知识点
通过查资料，大概了解了这里用到的几个函数，试着写下来自己的简单理解：

#### std::mem_fun
返回一个函数体(function object)里面含有的成员函数(member function)，然后返回结果。在此，如果已经update过了，返回false。
#### std::transform
std::mem_fun的实现涉及std::transform。
它的作用是遍历参数中的container里的element，将此作为参数中的函数体的参数，依次执行，得到并返回结果。
[例子][3]：

    int op_increase (int i) { return ++i; }

    // example of std::transform
    void std_transform()
    {
    	std::vector<int> foo;
    	std::vector<int> bar;
    
    	// set some values:
    	for (int i=1; i<6; i++)
    		foo.push_back (i*10);                         // foo: 10 20 30 40 50
    
    	bar.resize(foo.size());                         // allocate space
    
    	std::transform (foo.begin(), foo.end(), bar.begin(), op_increase);
    	// bar: 11 21 31 41 51
    
    	// std::plus adds together its two arguments:
    	std::transform (foo.begin(), foo.end(), bar.begin(), foo.begin(), std::plus<int>());
    	// foo: 21 41 61 81 101
    
    	std::cout << "foo contains:";
    	for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it)
    		std::cout << ' ' << *it;
    	std::cout << '\n';
    }
#### std::not1
返回相反boolean值
#### std::remove_if
移除符合条件的elements。
*[std::remove_if在c++11里的实现][]：


    enter code hertemplate <class ForwardIterator, class UnaryPredicate>
    ForwardIterator remove_if (ForwardIterator first, ForwardIterator last, UnaryPredicate pred)
    {
      ForwardIterator result = first;
      while (first!=last) {
        if (!pred(*first)) {
          *result = std::move(*first);
          ++result;
        }
        ++first;
      }
      return result;
    }
#### std::move
获取参数的rvalue reference，然后将其转为xvalue。

    std::string str = "Hello";
    std::vector<std::string> v;
    v.push_back(std::move(str));
将string转移到Vector v里面而避免copy的过程。此时str=""。
但有趣的是，如果继续

    // string move assignment operator is often implemented as swap,
    // in this case, the moved-from object is NOT empty
    std::string str2 = "Good-bye";
    std::cout << "Before move from str2, str2 = '" << str2 << "'\n";
    v[0] = std::move(str2);
    std::cout << "After move from str2, str2 = '" << str2 << "'\n";
output:
Before move from str2, str2 = 'Good-bye'
After move from str2, str2 = 'Hello'

[std::move例子的完整代码][5]

[3]:	http://www.cplusplus.com/reference/algorithm/transform/ 
[提问的解答]:     http://stackoverflow.com/questions/596162/can-you-remove-elements-from-a-stdlist-while-iterating-through-it/596708#596708 "提问的解答"
[Mike]:     http://stackoverflow.com/users/65004/mike	"Mike"
[std::remove_if在c++11里的实现]:     http://www.cplusplus.com/reference/algorithm/remove_if/		"std::remove_if在c++11里的实现"
[5]:     http://en.cppreference.com/w/cpp/utility/move
