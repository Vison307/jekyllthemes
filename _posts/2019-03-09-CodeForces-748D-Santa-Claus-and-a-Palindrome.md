# CodeForces 748D Santa Claus and a Palindrome

* 题目描述

  >  Santa Claus likes palindromes very much. There was his birthday recently. *k* of his friends came to him to congratulate him, and each of them presented to him a string s~i~ having the same length *n*. We denote the beauty of the *i*-th string by a~i~. It can happen that a~i~ is negative — that means that Santa doesn't find this string beautiful at all.
  >
  > Santa Claus is crazy about palindromes. He is thinking about the following question: what is the maximum possible total beauty of a palindrome which can be obtained by concatenating some (possibly all) of the strings he has? Each present can be used at most once. Note that all strings have **the same length** *n*.
  >
  > Recall that a palindrome is a string that doesn't change after one reverses it.
  >
  > Since the empty string is a palindrome too, the answer can't be negative. Even if all a~i~'s are negative, Santa can obtain the empty string.

  > 有k个长度为n的字符串，每个字符串有一个beauty度a~i~（可以为负数）。现将这些字符串连接组成一个回文字符串，新构成的回文字符串beauty度为构成该字符串的字串的beauty度之和。求所有新构成的回文字符串中最大的beauty度。（注意，空串的beauty度为0，所以最大beauty度不可能为负数。）

* 输入

  > 样例输入#1：
  >
  > ````
  > 7 3
  > abb 2
  > aaa -3
  > bba -1
  > zyz -4
  > abb 5
  > aaa 7
  > xyx 4
  > ````
  >
  > 样例输出#1：
  >
  > ````
  > 12
  > ````

  > 样例输入#2：
  >
  > ````
  > 3 1
  > a 1
  > a 2
  > a 3
  > ````
  >
  > 样例输出#2：
  >
  > ````
  > 6
  > ````

  > 样例输入#3：
  >
  > ````
  > 2 5
  > abcde 10000
  > abcde 10000
  > ````
  >
  > 样例输出#3：
  >
  > ````
  > 0
  > ````

* 样例解释：

  > In the first example Santa can obtain abbaaaxyxaaabba by concatenating strings 5, 2, 7, 6 and 3 (in this order).

* 思路：

  > 1.若字符串是非回文字符串，那么它要找到它的反转字符串和它匹配，我们只要贪心地选取该字符串和它的反转字符串中beauty度最大的字符串，直到无法取得（自身或反转字符串已经被取完。
  >
  > 2.若字符串是回文字符串，那么有两种可能：
  >
  > 2-1.当该回文字符串只剩下一个，那么它只能插入字符串的中间。但是我们不知道是否取当前字符串是最优（可能还有其它只剩下一个的回文字符串，比当前回文字符串的beauty度大），所以我们需要保存当前字符串的beauty度rem。将rem和其他只剩下一个的字符串的beauty度比较，取有最大beauty度的字符串插入生成的字符串中间，即rem = max（rem，beauty[i]。
  >
  > 2-2.当该回文字符串剩下大于等于二个时：
  >
  > 取第一、第二大的两个串，他们的beauty度分别为a, b：
  >
  > 1^°^ 若a > 0 && b > 0：那么将这两个字符串插入两边是最优解。
  >
  > 2^°^ 若a + b < 0 && a > 0： 那么只能取a，是插入中间的情况，和2-1相同。
  >
  > 3^°^ 若a + b > 0 && b < 0：那么有两种情况：a，b都取并插入两边，或者只取a插入中间。（注意：最多只能有一个回文串插入生成的串的中间，所以如果取a插入中间那么rem就没法插入了。但是若取a，b插入两边，rem还是能插入中间的。所以这两种情况那种情况是更优情况我们现在还无法判断）。
  >
  > 我们可以先将a, b插入两边，并记录b的值（b的值相当于比只插入a少掉的beauty值）。最后只要比较所有存储的b的值中的最小值sub（b是负数，相当于b的绝对值最大）和rem的值，若rem > |sub|（即 -|sub| + rem > 0)说明保留这个sub，加上rem的值是合算的（相当于取和sub所在的那一对回文字符串放在两边，再取rem放在中间会使生成串的beauty值增加），我们就将最后的答案加上rem；若rem < |sub|说明保留这个sub再加上rem值不合算（相当不应该配对然后中间放rem，而应将和sub配对的那个字符串放在中间），我们就将答案减去sub（相当于加上|sub|)。
  >
  > 其余情况不管放在中间还是配对了放在两边都会使总beauty值减少，不符合题意。

  > 题目的关键在于2-2步，第一次做的时候错误的考虑了若a+b>0那么直接加入队伍，而没有考虑若b<0时，既可能a+b时最优解，也可能只取a放在中间是最优解。导致错误。

  > 本题的另一个难点在于构造数据结构。由于一个字符串和一个beauty值（int）对应，所以可以用map，并且我们每次要查看当前串的最大beauty值可以用优先队列。所以可以用map<string, priority_queue\<int> >。

* Answer：

  ````cpp
  #include<bits/stdc++.h>
  using namespace std;
  const int N = 1e5+7;
  int n;
  string revstr(string s){
      string ans = "";
      for(int i = 0; i < s.length(); i++){
          ans.insert(ans.begin(), s[i]);
      }
      return ans;
  }
  bool isrev(string s){
      for(int i = 0; i < n/2; i++){
          if(s[i] != s[n - i - 1])   return false;
      }
      return true;
  }
  struct s1{
      string str;
      int beauty;
      inline bool operator >(const s1& s2)const{
          return this->beauty > s2.beauty;
      }
  };
  map<string , priority_queue<int> > s;
  int main(void){
      int k, ans = 0;
      int rem = 0, sub = 0;
      cin >> k >> n;
  
      int cnt = 0;
      for(int i = 0; i < k; i++){
          string t;
          int b;
          cin >> t >> b;
          s[t].push(b);
      }
      for(map<string, priority_queue<int> >::iterator it = s.begin(); it != s.end(); ++it){
          string now = it -> first;
          string rev = revstr(now);
          while(!s[now].empty()){
  
              if(!isrev(now)){
                  if(s.find(rev) != s.end() && !s[rev].empty()){
                      int a = s[now].top(); s[now].pop();
                      int b = s[rev].top(); s[rev].pop();
                      if(a + b > 0){
                          // cout << "now: " << now << " " << a << endl;
                          // cout << "rev: " << rev << " " << b << endl;
                          ans += (a+b);
                          // cout << "ans: " << ans << endl;
                      }
                  }
                  else{
                      s[now].pop();
                  }
              }
              else if(isrev(now)){
                  int a = s[now].top();s[now].pop();
                  if(!s[rev].empty()){
                      int b = s[rev].top(); s[rev].pop();
                      if(a > 0 && b >= 0){
                          ans += (a+b);
                          cnt += 2;
                      }
                      else if(a + b > 0 && b < 0){
                          ans += (a+b);
                          sub = min(sub, b);
                      }
                      else if(a + b <= 0 && a > 0){
                          rem = max(rem, a);
                      }
                      else break;
                  }
                  else{
                      if(a > 0){
                          rem = max(rem, a);
                      }
                      else{
                          break;
                      }
                  }
              }
          }
      }
      if(rem > -sub){
          ans += rem;
      }
      else    ans -= sub;
      cout << ans << endl;
      return 0;
  }
  ````

  