Download Link: https://assignmentchef.com/product/solved-cse-232-programming-project-11
<br>
<h1>Background</h1>

As this is the third time through on the BiMap, it is assumed that you understand the specifications of the individual methods. However, the update here is focused on using a linked list instead of a dynamic array. The overall idea remains the same:

<ul>

 <li>this is a templated class, two template types of key and value</li>

 <li>the ordered_by_keys_ list is always in sorted order according to the key of each Node</li>

 <li>the ordered_by_vals_ list is always in sorted order according to the value of each Node</li>

 <li>the keys and values are individually unique</li>

 <li>the list can grow dynamically in size over the course of the run</li>

</ul>




The spec below is different. You already know how the BiMap works. This is more about the pitfalls that are coming your way when using a linked list. You should really, really read this. It is advice that will help you!




<h1>Differences</h1>

<ol>

 <li>No iterators, no pointer arithmetic, no algorithms. In a singly linked list of our own construction, there are no iterators. C++ has the ability to add iterators to a class, but it is a bit beyond us. We got around iterators in arrays by using pointer arithmetic. Because an array is a single, contiguous, piece of memory, pointer arithmetic allows us to move to the next element of the array, just like an iterator. However, there are no iterators in our singly linked list. Likewise, pointer arithmetic doesn’t apply. Each Node is individually and separately created at a memory location chosen by the OS. There is no relationship between the memory addresses and the order of linked elements. We can only use the next pointers of each Node to put them in some order.</li>

</ol>




Without iterators/pointer_arithmetic, you cannot use any of the STL algorithms. None. You can’t sort, you can’t lower_bound, you can’t copy. Nothing.




<ol start="2">

 <li>find_key and find_value are sequential. The essential method used to find an element in our linked list can no longer use lower_bound. Further, even though the elements in the BiMap must be in sorted order, we cannot really do a binary search even if we construct that search ourselves. This is a fundamental limitation of a singly linked list: you can move only one direction, forward, through a list. A doubly linked list can move forward and backwards and that is a necessary requirement of a binary search</li>

</ol>




This means that find_key and find_value are going to be sequential search. That is, we must start at the beginning of the list and move forward until we find what we are looking for. Shame as it is sorted, but that is the penalty incurred with single linking.




<ol start="3">

 <li>add/remove_key/remove_value and find_key/find_value. The fact that this is a single linked list causes us some more headaches as well. Consider the diagram below.</li>

</ol>







<table width="361">

 <tbody>

  <tr>

   <td width="106">


    <table width="71">

     <tbody>

      <tr>

       <td width="44">“b”</td>

       <td width="27">1</td>

      </tr>

     </tbody>

    </table></td>

   <td width="145">


    <table width="71">

     <tbody>

      <tr>

       <td width="44">“d”</td>

       <td width="27">2</td>

      </tr>

     </tbody>

    </table></td>

   <td width="110">


    <table width="71">

     <tbody>

      <tr>

       <td width="44">“e”</td>

       <td width="27">3</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

nullptr

<table width="71">

 <tbody>

  <tr>

   <td width="43">“c”</td>

   <td width="28">7</td>

  </tr>

 </tbody>

</table>

headtail







Let’s try to “find” where to insert {“c”,7}. If find_key, using sequential search, uses the behavior we found in lower_bound, then the return location would point to {“d”, 2}, the first element greater than the element being inserted. That is a real problem! We can change the newly made node containing {“c”, 7} to point to {“d”, 2}, but we cannot go backwards to update that {“b”,1} points to the new node {“c”, 7}.




We could change find_key and find_value to imitate upper_bound (look it up), which finds the less than or equal value. In this case, it would return {“b”, 2}, and we could do the proper updates.




However, with either lower_bound or upper_bound behavior, what about remove? If we try to remove {“d”, 2}, then either algorithmic approach to find_key would point to the {“d”,2} Node. We could not update that the {“b”, 1} should point to the {“e”, 3} node (cutting out the {“d”,2} Node as required) because we have no way to go backwards.




You have two choices here:

<ol>

 <li>You can use either upper_bound or lower_bound behavior in find_key and find_value, but then update add/remove to search sequentially for the pointer “just behind”. That is, use find_key and find_value to “find” the Node, then walk forward (again) from the beginning up to the Node “just behind” the pointer that find_key/find_value returned.</li>

 <li>You could update find_key and find_value to return two pointers (as a pair for example). One of the pointers is the lower_bound/upper_bound pointer and the other is the “trailer” pointer, the one just behind.</li>

 <li>There is a third, bad choice. You could just skip find_key/find_value and put the behavior everywhere you need it, but you need it in a lot of places and you are going to screw up that way. Not modular at all!</li>

</ol>

I did <u>b </u>above, but I will leave the header ambiguous in the return type and you can do as you wish.




<ol start="4">

 <li>a weird error. A error that can be hard to dig out on its own is: “pointer being freed was not allocated” . This is again a RunTime (not Compiler) error. It basically means that you tried to delete a Node twice. This can happen if you delete something in a method (somewhere) and leave it hooked into in the list. When the destructor is called, it tries to delete it again. Can’t do that.</li>

</ol>




<ol start="5">

 <li>nullptr and &amp;&amp;. You are going to segfault a lot. Why? Imagine, however you implement find_key, that what you are looking for is not found. At that point your return value will be nullptr (off the end of the list, so what you were seeking was not there). If you have something like the following, you are guaranteed to fault:</li>

</ol>




…

auto itr = find_key(“z”); if (itr-&gt;first == key){

…




What happens if find_key returns nullptr? When you try to dereference itr-&gt;first, it segfaults. You cannot dereference a nullptr. But you can easily protect yourself:




…

auto itr = find_key(“z”);

if (itr != nullptr &amp;&amp; itr-&gt;first == key){ …




This works because &amp;&amp; is sequential. It evaluates each clause in order, and the first clause that is false halts the evaluation of the remaining clauses (“short circuiting”, week 1 video!!!). If the variable is nullptr, then none of the remaining clauses execute. You should use that!!!




<ol start="6">

 <li>linked list iteration. This is the for loop to iterate through a linked list</li>

</ol>




…  for (auto itr = head_; itr != nullptr; itr = itr-&gt;next){

…




It isn’t ++itr (remember, no pointer arithmetic), it’s whatever the next pointer points to. Remember that. Also, ++itr will compile fine. It will segfault when you run!




<ol start="7">

 <li>all the cases. You have to enumerate all the possibilities and deal with them in your code. For example, the add function has at least 4 cases:

  <ol>

   <li>the Node is already there</li>

   <li>We’re adding a node and:

    <ol>

     <li>it goes at the front</li>

     <li>it goes at the back iii. it goes somewhere in the middle</li>

    </ol></li>

  </ol></li>

</ol>

Each might have its own needs and conditions. Look at all the cases!




<ol start="8">

 <li>Use the debugger. Look, segfaulting sucks but it happens a lot, so get used to it. However, the only truly effective way to figure out where a segfault occurred is by using the debugger. If you ask for help by saying “Why is my code segfaulting”, the only reasonable answer is “Where does the debugger tell you it is segfaulting”. No one knows until you answer that question. You only need a few things:

  <ol>

   <li>g++ … -g … use the -g to get debugging information in your executable b. gdb a.out</li>

   <li>run</li>

   <li>up/down to find code that looks familiar. You have to move up to get out of library code you didn’t write until you find code you wrote</li>

   <li>list shows code around where the fault occurred</li>

   <li>print variable allows you to print a variable (find that null pointer, address 0x0000…).</li>

  </ol></li>

</ol>




<ol start="9">

 <li>Use the example code. As before, feel free to use the example code from the course. You have to modify it of course, but it can be very helpful. I will not accuse anyone of cheating if you are using the course example code!</li>

</ol>




<h1>Class Node</h1>

Here is the header part of Node. Not much difference except for the next pointer .




template&lt;typename K, typename V&gt; struct Node {   K first;

V second;

Node *next = nullptr;




Node() = default;

Node(K,V);

bool operator==(const Node&amp;) const;

friend ostream&amp; operator&lt;&lt;(ostream &amp;out, const Node &amp;n){

// YOUR CODE HERE!

}

};




Much of this does not change I would think from your previous work.




<h1>Class BiMap</h1>

Here’s the updated BiMap




template&lt;typename K, typename V&gt; class BiMap{  private:

Node&lt;K,V&gt;* ordered_by_keys_head_ = nullptr;




Node&lt;K,V&gt;* ordered_by_vals_head_ = nullptr;




size_t sz_ = 0;




//Node&lt;K,V&gt;* find_key(K);

//Node&lt;K,V&gt;* find_value(V);




//pair&lt;Node&lt;K, V&gt;*, Node&lt;K, V&gt;*&gt; find_key(K)

//pair&lt;Node&lt;K, V&gt;*, Node&lt;K, V&gt;*&gt; find_value(V);

public:

BiMap()=default;

BiMap(initializer_list&lt; Node&lt;K,V&gt; &gt;);

BiMap (const BiMap&amp;);

BiMap operator=(BiMap);

~BiMap();   size_t size();   K remove_val(V value);    V remove_key(K key);    bool add(K,V);   V value_from_key(K);   K key_from_value(V);   bool update(K,V);     int compare(BiMap&amp;);

BiMap merge (BiMap&amp;);




friend ostream&amp; operator&lt;&lt;(ostream &amp;out, const BiMap &amp;bm){

//WRITE OPERATOR&lt;&lt; CODE HERE

}

};

<h1>Data Members</h1>

<ul>

 <li>ordered_by_keys_head_, ordered_by_vals_head_ are the pointers to the first Nodes in their respective lists, and are nullptr if the list is empty.</li>

 <li>sz_ is the number of Nodes in the list. You don’t have to update it, but it makes many things easier if it accurately reflects the size of the list. The size() method is tested.</li>

 <li>find_key/find_value the return type is your choice, as indicated above</li>

</ul>




<h1>Methods</h1>

There isn’t a lot different (in terms of input parameters and outputs) between 10 and 11. If you are careful, you can preserve a lot of what you did in 10




<h1>Requirements</h1>

We provide proj11_bimap.h as a skeleton that you must fill in. You submit to Mimir proj11_bimap.h




We will test your files using Mimir, as always.


