---
layout:     post
title:      A Review of Iterator and Several std methods
category: blog
description: A review of the iterator and several std methods
---

**As a C++ learner, my points and code might be incorrect or incaccurate. I will update this article during future study.**
**This article could also be taken as a practic of english writing from a non native english spearker.LOL**

## Start
Recently I am confused about how to create a skill buff system, to apply several buff and, what is more important, to correctly unapply those
buff(Debuff). Then I did some search online, finally got [this][1] on StatckExchange. In this thread Laurent Couvidou gives out a very detailed 
answer. His answer includes a line using std methods, which let me confused since I only know std containers like List, Vector and some simple methods.
So that I decided to do a simple study of those std methods.

Following is the code which confused me:

    buffList.remove_if(std::mem_fun(&Buff::hasTimedOut));

Laurent Couvidou credits it to a answer from [StackOverflow][2].

## Move on
Then I walked into another class room. There, someone asked a similiar question: how to remove a skill from skill list immediately after a undate.

His code likes this:

    for (std::list<item*>::iterator i=items.begin();i!=items.end();i++)
    {
        bool isActive = (*i)->update();
        //if (!isActive) 
        //  items.remove(*i); 
        //else
           other_code_involving(*i);
    }
    items.remove_if(CheckItemNotActive);
But get "List iterator not incrementable" error while doing i++.
Then I recalled that I did the same thing several months ago, but can't remember how to fixed it.That is why I start to write blog. We Chinese says that "The palest ink is better than the best memory".

#### My understanding

Like the role of template, which is helping programmer to handle any type of data, the iterator is used to handle elements inside the different container. Internal structs of iterators from List and Vector might be different, but programmer can use them in the similiar way.

>...each container class(vector, list, deque, and so on) defines an iterator type appropriate to the class. For one class, the iterator might be apointer; for another, it might be an object【C++ Primer Plus，P995】

* question：I understand that a iterator is a pointer. But I don't know what kind of iterator could be a object.

#### My lab
I created a list and a vector, both contain 10 int(1 to 10). Then remove odd numbers from the container by a 'for' loop and a 'while' loop

In debug mode, I find that a iterator of std::List is pointing to a node, which contains nextLink, preLink, and value.( In class, I practiced to create a customized bio direction list. )

After list.erase(itor), that node losts its feature as a iterator.(But I noted that its value is the same to list.end() now.) List.erase(itor) returns the next iterator(++itor). So it can't be increased.

Follwing code gets "List iterator not incrementable" error:

    std::list<int> intList;
	for (int i = 0; i < 10; ++i)
	{
		intList.push_back(i);
	}

	for (std::list<int>::iterator i = intList.begin(); i != intList.end(); ++i)
	{
		if (*itor % 2 == 1)
		{
			intList.erase(itor);
		}
	}
We can avoid that error by changing it to:

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
That due to the difference between pre-increment and post-increment operators.

    intList.erase(itor++);
    intList.erase(++itor);
For ++itor(fake code)
	
	return itor += 1；

For itor ++(fake code)

    copyItor = *this;
    *this += 1;
    return copyItor;
**（So that, most people argues that using ++i has a performance gain. I guess it is real at least in c++.）**

Moreover，we should implement the 'while' loop to avoid skipping one element per iteration . And a good person in thread reminds that itor = intList.erase(itor) is safer. Yes, he is correct because the lazy coding( intList.erase(itor++) ) cause "iterator is incompatible" error.

Test code for vector:

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
			//itor = intVector.erase(itor);
			intVector.erase(itor++);
		}
		else
			++itor;
	}
About to the reason of this issue, Eric Seppanen says:

>The alternate usage i = items.erase(i) is safer, because it's equivalent for a list, but will still work if someone changes the container to a vector. With a vector, erase() moves everything to the left to fill the hole. If you try to remove the last item with code that increments the iterator after erasing, the end moves to the left, and the iterator moves to the right-- past the end. And then you crash.

However, in my test, before removing the last element, I got error message after the first erase operation. The process is as following:

When itor==1，genrate:

		if (*itor % 2 == 1)
		{
			//itor = intVector.erase(itor);
			intVector.erase(itor++);
		}

After it erase int 1, in watch window, I find that itor's value is 3(int 2 moves left by one step, iterator moves right by one step). So that, It looks that the problem should be skipping one element. It is strange to crash by a comparibility problem.

#### Memery address observation in debug
itor==1，pre-erase state. [ptr]:0x004aab74{1}。

![Alt text](/images/blog/cpp_iterator_001.jpg)

after intVector.erase(itor++). [ptr]0x004aab78{3}。I assume that [ptr]:0x004aab74 should be 2。

![Alt text](/images/blog/cpp_iterator_002.jpg)

**In above image, I noted that std::Iterator_base12 is nullprt。**

Then the code move to next step. For while loop, go to comparing( if(itor != intVector.end()) ), crash
![Alt text](/images/blog/cpp_iterator_003.jpg)

For for loop, go to iteration( ++itor ), crash
![Alt text](/images/blog/cpp_iterator_004.jpg)

* question: I think, in this case, the struct of itorator is changed. But I want to know how does it changed?

## The answer of 'How to remove the post-updated skill from list'
[Mike][] gives answer:

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
Then, he gives a specific answer for skill update system

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
This is a gem(for me at least):

    items.remove_if(std::not1(std::mem_fun(&Item::update)));

## My understanding for those STD methods
Before practical experience, my understanding is simple and could be wrong.

#### std::mem_fun
Generate a function object, which is Item::update here, then return its result. In this example, the result is a boolean value.
#### std::transform
std::transform is implemented in std::mem_fun.
This method could include a input parameter, a output parameter and a function object.
It calls the function by the input parameter( if the parameter is a container, it will use every item in the list as parameter of the funcion ), then put result into output parameter(if the output parameter is a containner, result will be put into container one by one)

[Test code][3]：

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
Return the oppsite of boolean value. In this case, if the skill has been updated, return true.
#### std::remove_if
Remove the elements if it match the condition。
*[Aprroach of std::remove_if in C++11][4]：

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
It gets the rvalue reference of parameter，convert it to xvalue。
According to the [example code][5] 

    std::string str = "Hello";
    std::vector<std::string> v;
    v.push_back(std::move(str));
Move string in str to Vector v, avoding copy process。Now str=""。
Following feature is very intersting:

    // string move assignment operator is often implemented as swap,
    // in this case, the moved-from object is NOT empty
    std::string str2 = "Good-bye";
    std::cout << "Before move from str2, str2 = '" << str2 << "'\n";
    v[0] = std::move(str2);
    std::cout << "After move from str2, str2 = '" << str2 << "'\n";

output:

Before move from str2, str2 = 'Good-bye'

After move from str2, str2 = 'Hello'

## TODO
+ Figure out what the Vector itertor is diffenrent from others
+ Try to use these std methords in my future practice
+ Learn more about data structure


[1]:	http://gamedev.stackexchange.com/questions/46772/how-to-implement-buffs-debuffs-temporary-stat-changes-in-an-rpg
[2]:	http://stackoverflow.com/a/596708/1005455
[3]:	http://www.cplusplus.com/reference/algorithm/transform/
[Mike]:	http://stackoverflow.com/users/65004/mike	"Mike"
[4]:    http://www.cplusplus.com/reference/algorithm/remove_if/
[5]:    http://en.cppreference.com/w/cpp/utility/move
